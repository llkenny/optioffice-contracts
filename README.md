# OptiOffice Contracts

This repository contains the OpenAPI contract for the OptiOffice API.

## Linting

We use [Redocly CLI](https://redocly.com/docs/cli/) to lint the OpenAPI specification.

### Run locally

```sh
npx @redocly/cli lint openapi.yaml
```

### Run in CI

The GitHub Actions workflow `.github/workflows/lint.yml` runs the lint check on every push and pull request.
