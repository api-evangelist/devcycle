---
name: Create and roll out a DevCycle feature flag
description: >-
  Create a feature flag with variations and a variable, then manage its status
  through the DevCycle Management API. Grounded in real Management API operationIds.
api: openapi/devcycle-management-openapi.json
operations:
- ProjectsController_findAll
- FeaturesController_create
- VariablesController_create
- VariationsController_create
- VariationsController_findAll
- FeaturesController_updateStatus
---

# Create and roll out a DevCycle feature flag

Use the DevCycle Management API to create a feature flag, attach a variable and
variations, and manage its lifecycle status.

## Auth
The Management API uses OAuth2 `client_credentials`. Exchange your dashboard
Client ID + Secret for a bearer token:

```
POST https://auth.devcycle.com/oauth/token
grant_type=client_credentials
audience=https://api.devcycle.com/
```

Send the token as `Authorization: Bearer <token>` on every call. Base URL is
`https://api.devcycle.com/v1`. See `authentication/devcycle-authentication.yml`.

## Steps
1. **Pick a project.** `ProjectsController_findAll` (`GET /v1/projects`) — note the
   project `key`; all subsequent resources are scoped under
   `/v1/projects/{project}`.
2. **Create the feature.** `FeaturesController_create`
   (`POST /v1/projects/{project}/features`) with a unique `key`, `name`, and type.
   A duplicate `key` returns `409 Conflict`.
3. **Create a variable.** `VariablesController_create`
   (`POST /v1/projects/{project}/variables`) — choose the type
   (Boolean/String/Number/JSON) and associate it with the feature.
4. **Define variations.** `VariationsController_create`
   (`POST /v1/projects/{project}/features/{feature}/variations`) for each variation
   (e.g. control/treatment) with the variable values it serves. List them with
   `VariationsController_findAll`.
5. **Manage status.** `FeaturesController_updateStatus` to move the feature
   through its lifecycle (e.g. active → complete/archived).

## Conventions & errors
- Pagination on list endpoints uses `page`/`perPage` with `sortBy`/`sortOrder`.
- There is **no idempotency-key** contract; rely on unique `key`s (duplicate → 409).
- Errors use `{ statusCode, message, error }` (not RFC 9457). See
  `errors/devcycle-problem-types.yml`: 400 validation, 401/403 auth, 404 missing,
  409 duplicate key, 412 precondition (e.g. "Variable does not belong to Feature").
