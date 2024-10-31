# dynamic-config-custom-exec-tests

## Overview

A sample config.yml that uses a modified versions of the `circleci/path-filtering` and `circleci/continuation` orbs to allow the orbs to be used with private Docker images as executors.

## Configuration Parameters

The file includes customizable parameters that control various aspects of the workflows:

- **registry**: (Type: `string`)  
  Default: `vjpandian1818`  
  Specifies the Docker registry to be used for pulling images.

- **image**: (Type: `string`)  
  Default: `vjpandian`  
  Defines the base Docker image for the executor.

- **image-tag**: (Type: `string`)  
  Default: `cimg-python`  
  Sets the tag for the Docker image.

- **run-continue**: (Type: `boolean`)  
  Default: `true`  
  Determines whether the continuation workflow should be executed.  
  **Description**: If `true`, the continuation workflow runs; if `false`, the path filtering workflow executes.

## Orbs Used

1. **path-filtering Orb**: `vjpandian/path-filtering-orb@1.1.4`  
   A [fork](https://circleci.com/developer/orbs/orb/vjpandian/path-filtering-orb) of the official `circleci/path-filtering` orb.  
   **Purpose**: Adds support for private Docker images while maintaining the path filtering functionality.

2. **continuation Orb**: `vjpandian/continuation@1.1.0`  
   A [fork](https://circleci.com/developer/orbs/orb/vjpandian/continuation) of the official `circleci/continuation` orb.  
   **Purpose**: Extends the continuation functionality to support private Docker images.

## Workflows

### 1. `path-workflow`

**Execution Condition**:  
Runs when `run-continue` is set to `false`.

**Job Details**:
- **Job**: `path-filtering/filter-private-docker-executor`
- **Pre-steps**:
  - **Checkout**: Retrieves the source code.
  - **Print Message**: Outputs "hello" for debugging purposes.

**Parameters**:
- **base-revision**: `main`
- **circleci_domain**: `circleci.com`
- **dockerhub-username**: Environment variable `$ARTIFACTORY_USER`
- **dockerhub-password**: Environment variable `$ARTIFACTORY_PASSWORD`
- **registry**: Uses `<< pipeline.parameters.registry >>`
- **image**: Uses `<< pipeline.parameters.image >>`
- **tag**: Uses `<< pipeline.parameters.image-tag >>`
- **config-path**: `.circleci/continue-config.yml`
- **context**: `artifactory-creds`
- **mapping**: Specifies path mappings for conditional deployments:
  - `terraform/.*`: Maps to `tf-deploy` with `true`
  - `helm/.*`: Maps to `helm-deploy` with `true`

### 2. `force-workflow`

**Execution Condition**:  
Runs when `run-continue` is set to `true`.

**Job Details**:
- **Job**: `continuation/continue-private-docker-executor`

**Parameters**:
- **dockerhub-username**: Environment variable `$ARTIFACTORY_USER`
- **dockerhub-password**: Environment variable `$ARTIFACTORY_PASSWORD`
- **registry**: Uses `<< pipeline.parameters.registry >>`
- **image**: Uses `<< pipeline.parameters.image >>`
- **tag**: Uses `<< pipeline.parameters.image-tag >>`
- **circleci_domain**: `circleci.com`
- **configuration_path**: `.circleci/workflows.yml`
- **parameters**: `{ "force": true }`
- **context**: `artifactory-creds`

## Environment Variables

- **$ARTIFACTORY_USER**: DockerHub username for authentication.
- **$ARTIFACTORY_PASSWORD**: DockerHub password for authentication.

## Usage

1. Modify the parameters as needed to customize the registry, image, or tag.
2. Set the `run-continue` parameter to control which workflow should execute:
   - **`true`**: Runs the `force-workflow` using continuation.
   - **`false`**: Runs the `path-workflow` for path-based filtering.

3. Use the appropriate environment variables for authentication to DockerHub.

## Notes

- Ensure that the `artifactory-creds` context is properly configured with the required credentials.
- Adjust the `mapping` rules as necessary to fit the needs of your path-based deployments.
- Both `vjpandian/path-filtering-orb@1.1.4` and `vjpandian/continuation@1.1.0` are specifically forked to handle private Docker images, adding extended functionality over the official CircleCI orbs.
