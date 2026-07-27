# Reference — GitHub → ADO issue migration (script template + snippets)

A ready-to-adapt PowerShell template plus modular snippets for the **GitHub → Azure DevOps** direction, which is where the CLI mechanics are hardest. Other directions reuse the same shape (discover → map → create parents first → link → set state → verify by id), swapping `gh`/`az`/`glab` commands.

## How to use this (do NOT bypass the skill's rules)

This template is a helper the agent **generates and adapts**, then runs **inside** the normal workflow — never a "just run it" replacement for the safety gates:

1. **Preflight & discover (read-only)** — snippets 0–1. Confirm auth and enumerate source/target.
2. **Present the dry-run** — snippet 2 builds the plan; show it and get explicit approval. Nothing is written yet.
3. **Execute the approved batch** — snippets 3–6. Run only after approval; the script is idempotent (skip-if-exists) and persists a `GH#→ADO#` map so a failed run is resumable.
4. **Verify** — snippet 7, by direct work-item id (WIQL lags after writes).

**Auth is a prerequisite the agent confirms, not automates** — it cannot sign the user in (EMU `gh auth login`, MSA-backed ADO, etc.). Fail fast with a clear message if either side is not signed in.

## Parameters (top of the script)

```powershell
$GhRepo   = 'owner/repo'                          # GitHub source (owner may be an EMU handle, e.g. name_enterprise)
$Org      = 'https://dev.azure.com/your-org/'     # ADO org
$Project  = 'Your-Project'                         # ADO project (work items are project-scoped)
$DescFormat = 'Html'                               # 'Html' (safe default) or 'Markdown' (needs org feature)
$DefaultType = 'User Story'                         # type when no type-signal label is present
$TypeByLabel = @{ 'Epic' = 'Epic'; 'User Story' = 'User Story' }   # GitHub label -> ADO work-item type
$StateByGh   = @{ 'OPEN' = 'New'; 'CLOSED' = 'Closed' }            # PROCESS-SPECIFIC (Agile shown; Basic = To Do/Done)
$MapFile  = Join-Path $env:TEMP 'gh_ado_map.json'  # persisted GH#->ADO# map (resumable)
```

---

## 0. Preflight & auth (read-only — fail fast)

```powershell
# GitHub: is the signed-in account able to READ the source? (EMU repos need the enterprise account)
$repo = gh repo view $GhRepo --json name,visibility 2>$null | ConvertFrom-Json
if (-not $repo) { throw "Cannot read $GhRepo. If the owner is an EMU handle (name_enterprise), run: gh auth login --hostname github.com --web  (then gh auth switch)." }

# ADO: az signed in + azure-devops extension + discover the PROCESS (dictates valid types/states)
az account show -o none 2>$null; if ($LASTEXITCODE) { throw 'Run: az login' }
if (-not (az extension show -n azure-devops -o none 2>$null; $LASTEXITCODE -eq 0)) { az extension add -n azure-devops --only-show-errors }
$process = (az devops project show --project $Project --org $Org -o json | ConvertFrom-Json).capabilities.processTemplate.templateName
Write-Host "ADO process: $process   (map $StateByGh states to THIS process's valid states)"
```

## 1. Discover source + target (read-only)

```powershell
# Source labels + issues (full data incl. body/author/date/url/labels)
$labels = gh label list -R $GhRepo --limit 200 --json name,color,description | ConvertFrom-Json
$issues = gh issue list -R $GhRepo --state all -L 500 --json number,title,state,body,author,createdAt,url,labels | ConvertFrom-Json

# Authoritative hierarchy: GitHub NATIVE sub-issues (parentGh -> childGh[])
$children = @{}
foreach ($iss in $issues) {
    $subs = gh api "repos/$GhRepo/issues/$($iss.number)/sub_issues" 2>$null | ConvertFrom-Json
    if ($subs) { $children["$($iss.number)"] = @($subs.number) }
}

# Target: existing work items (match by title to avoid duplicates). NOTE: WIQL lags right after writes.
$wiql = "SELECT [System.Id],[System.Title] FROM WorkItems WHERE [System.TeamProject]=@project"
$existing = az boards query --org $Org --project $Project --wiql $wiql -o json | ConvertFrom-Json
$adoIdByTitle = @{}; foreach ($w in $existing) { $adoIdByTitle[$w.fields.'System.Title'] = $w.fields.'System.Id' }
```

## 2. Map labels → type / tags, and build the dry-run

```powershell
function Get-AdoType($iss) {
    foreach ($lbl in $iss.labels.name) { if ($TypeByLabel.ContainsKey($lbl)) { return $TypeByLabel[$lbl] } }
    return $DefaultType
}
function Get-AdoTags($iss) {
    # Every label EXCEPT the type-signal ones becomes an ADO tag
    return (($iss.labels.name | Where-Object { -not $TypeByLabel.ContainsKey($_) }) -join '; ')
}

# DRY-RUN: print what would happen, write nothing. Present this and get approval.
$issues | Sort-Object number | ForEach-Object {
    '{0,-4} {1,-11} [{2}] tags="{3}"  {4}' -f $_.number, (Get-AdoType $_), $StateByGh[$_.state], (Get-AdoTags $_), $_.title
}
```

## 3. Description (body only) + attribution as a COMMENT

Keep the description as the **clean original body** and record migration attribution as a **comment**, not a description prefix (the comment still posts as the authenticated user, but the description stays faithful and renders cleanly). `System.Description` is **HTML by default** over the API; do not pass multi-line values inline (Windows `az.cmd %*` mangles quotes/newlines).

```powershell
function New-Note($iss) {   # posted as a COMMENT (snippet 4), not embedded in the description
    "Originally created by @$($iss.author.login) on $((Get-Date $iss.createdAt).ToString('yyyy-MM-dd')) - migrated from GitHub $GhRepo#$($iss.number) ($($iss.url))"
}

# (a) HTML mode (VERIFIED, no org feature needed): encode + <br>, collapse to ONE quote-free line -> CLI-safe.
function New-DescHtml($iss) {
    if ([string]::IsNullOrWhiteSpace($iss.body)) { return '' }
    return (([System.Net.WebUtility]::HtmlEncode($iss.body) -replace "`r`n","`n" -replace "`n",'<br>') -replace "`r",'' -replace "`n",'')
}

# (b) Markdown mode (RENDERS ##/**bold**; requires the org's work-item Markdown feature).
#     Store RAW markdown AND set the field format via a JSON-Patch through `az devops invoke`
#     (System.Description is HTML unless multilineFieldsFormat says Markdown). az boards has NO flag for this.
function Set-DescMarkdown($adoId, $iss) {
    $patch = @(
        @{ op = 'add'; path = '/fields/System.Description';              value = "$($iss.body)" },
        @{ op = 'add'; path = '/multilineFieldsFormat/System.Description'; value = 'Markdown' }
    )
    $f = Join-Path $env:TEMP "wi_$adoId.json"; $patch | ConvertTo-Json -Depth 5 | Set-Content $f -Encoding utf8
    az devops invoke --org $Org --area wit --resource workitems --route-parameters id=$adoId `
        --http-method PATCH --media-type 'application/json-patch+json' --in-file $f --api-version 7.1 -o none
    Remove-Item $f -ErrorAction SilentlyContinue
}

# Attribution as a work-item COMMENT (ADO discussion). Other directions: `gh issue comment`, `glab issue note`.
function Add-MigrationComment($adoId, $iss) {
    az boards work-item update --id $adoId --discussion (New-Note $iss) --org $Org -o none
}
```

## 4. Create (parents first) + persist GH#→ADO# map

```powershell
$map = if (Test-Path $MapFile) { (Get-Content $MapFile -Raw | ConvertFrom-Json).PSObject.Properties | ForEach-Object -Begin { $h=@{} } -Process { $h[$_.Name]=$_.Value } -End { $h } } else { @{} }

function New-WorkItem($iss) {
    if ($map.ContainsKey("$($iss.number)")) { return $map["$($iss.number)"] }         # resume
    if ($adoIdByTitle.ContainsKey($iss.title)) { $map["$($iss.number)"] = $adoIdByTitle[$iss.title]; return $map["$($iss.number)"] }  # skip-if-exists
    $type = Get-AdoType $iss; $tags = Get-AdoTags $iss
    if ($DescFormat -eq 'Html') {
        $res = az boards work-item create --org $Org --project $Project --type $type --title $iss.title --description (New-DescHtml $iss) --fields "System.Tags=$tags" -o json | ConvertFrom-Json
    } else {
        $res = az boards work-item create --org $Org --project $Project --type $type --title $iss.title --fields "System.Tags=$tags" -o json | ConvertFrom-Json
        Set-DescMarkdown $res.id $iss
    }
    if (-not $res.id) { throw "Create failed for GH #$($iss.number)" }
    Add-MigrationComment $res.id $iss                       # attribution as a COMMENT, not a description prefix
    $map["$($iss.number)"] = $res.id
    ($map | ConvertTo-Json) | Set-Content $MapFile          # persist after EVERY create -> resumable
    return $res.id
}

# Parents (issues that have children) first, then the rest — so parent ids exist for linking.
$parentNums = $children.Keys
($issues | Where-Object { $parentNums -contains "$($_.number)" } | Sort-Object number) | ForEach-Object { [void](New-WorkItem $_) }
($issues | Where-Object { $parentNums -notcontains "$($_.number)" } | Sort-Object number) | ForEach-Object { [void](New-WorkItem $_) }
```

## 5. Parent links from sub-issues

```powershell
foreach ($parentGh in $children.Keys) {
    foreach ($childGh in $children[$parentGh]) {
        az boards work-item relation add --id $map["$childGh"] --relation-type parent --target-id $map[$parentGh] --org $Org -o none
    }
}
```

## 6. Set closed state

```powershell
foreach ($iss in $issues) {
    if ($iss.state -eq 'CLOSED') { az boards work-item update --id $map["$($iss.number)"] --state $StateByGh['CLOSED'] --org $Org -o none }
}
# New items default to 'New' in Agile, so only closed items need a state update.
```

## 7. Verify (direct id — NOT WIQL)

```powershell
# WIQL can return 0 for several seconds after writes. Verify over the created id range instead.
$ids = $map.Values | Sort-Object
$types=@{}; $states=@{}; $ok=0
foreach ($id in $ids) {
    $wi = az boards work-item show --id $id --org $Org -o json 2>$null | ConvertFrom-Json
    if ($wi) { $ok++; $t=$wi.fields.'System.WorkItemType'; $s=$wi.fields.'System.State'; $types[$t]++; $states[$s]++ }
}
Write-Host "Confirmed $ok / $($ids.Count) work items"
$types.GetEnumerator()  | ForEach-Object { "  type  {0,-12} {1}" -f $_.Key,$_.Value }
$states.GetEnumerator() | ForEach-Object { "  state {0,-10} {1}" -f $_.Key,$_.Value }
```

## Gotchas this template encodes

- **`az.cmd %*` escaping** — never pass multi-line/quoted bodies inline; HTML mode collapses to one line, Markdown mode uses `--in-file`.
- **`System.Description` = HTML by default** — rendered markdown needs `multilineFieldsFormat = Markdown` (no `az boards` flag; use `az devops invoke`) and the org's work-item Markdown feature.
- **WIQL indexing lag** — verify by direct `work-item show --id`, not `az boards query`.
- **Process-specific states/types** — query the process first; `Closed`/`New` are Agile; Basic uses `Done`/`To Do`.
- **Attribution as a comment** — the original author/date/link goes in a work-item comment (`--discussion`), not a description prefix, so the description stays the clean original body; the comment still posts as the authenticated user.
- **EMU source access** — a personal `gh` account can't read `name_enterprise` repos; sign in as the EMU account.
- **Resumability** — the `GH#→ADO#` map is persisted after every create, and creates are skip-if-exists, so a failed run re-runs safely.
