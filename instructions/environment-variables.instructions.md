---
description: "Use when creating or changing Python packages, libraries, or applications involving environment variables, settings, SDK client setup, credentials, or dependency ownership."
---

## Environment Independence

### Python packages and reusable libraries

- Accept configuration through explicit function or constructor parameters instead of reading environment variables directly. The caller is responsible for retrieving environment values and passing them into the package.
- Read an environment variable inside a package only when the package itself introduces and documents that variable as part of its own configuration contract. Do not fetch environment values on behalf of another library or service.
- Prefer accepting caller-created service or SDK clients through dependency injection. Delegate client creation, credential selection, configuration retrieval, and client lifecycle ownership to the caller as much as possible.

### Python applications

- Centralize all environment-variable retrieval in one shared configuration class or module. Do not scatter environment reads throughout application code.
- Group related settings in child configuration classes when useful, but expose those groups through one root configuration class that holds the child configurations.
- Expose environment-backed settings as UPPERCASE properties or attributes matching their environment-variable names, such as `MICROSOFT_FOUNDRY_PROJECT_ENDPOINT`.
- Retrieve configuration through this shared entry point, then pass values or configured clients explicitly to application components and reusable packages.
