# Use Amazon AWS Cloudformation Stack Outputs as environment variables

This action will fetch AWS CloudFormation Stack outputs from a given stack and
export the outputs as environment variables. 

**The keys of the Stack outputs will be used as the environment variable names. 
This could potentially overwrite existing environment variables.**

- [Usage](#usage)
    - [Fetching Outputs](#fetching-outputs)
    - [Fetching Outputs with OIDC](#fetching-outputs-with-oidc)
    - [Inputs](#inputs)
- [Examples](#examples)
    - [Example Workflow (OIDC)](#example-workflow-oidc)
    - [Example Workflow (static credentials)](#example-workflow-static-credentials)
- [Development](#development)
    - [Running Tests Locally](#running-tests-locally)

## Usage

Add the entry below to your workflow file. The `stack_name` should be the
case-sensitive name of the stack you are querying in AWS CloudFormation.

### Fetching Outputs

To authenticate with static credentials, provide them explicitly:

```yaml
uses: accendero/aws-stack-outputs-to-env@v2.1
id: <step id 1>
with:
  access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
  secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  region: ${{ secrets.AWS_DEFAULT_REGION }}
  stack_name: "MyStack"
```

### Fetching Outputs with OIDC

If you're using OIDC authentication (via `aws-actions/configure-aws-credentials`),
you can omit the `access_key_id` and `secret_access_key` inputs. The action will
use the existing AWS session credentials.

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/MyRole
    aws-region: us-west-2

- uses: accendero/aws-stack-outputs-to-env@v2.1
  id: <step id 1>
  with:
    region: "us-west-2"
    stack_name: "MyStack"
```

### Inputs

* `access_key_id` - _(optional)_ AWS access key ID. Must be set together with `secret_access_key`. Omit both if an active AWS session is already configured (e.g. via OIDC).
* `secret_access_key` - _(optional)_ AWS secret access key. Must be set together with `access_key_id`. Omit both if an active AWS session is already configured (e.g. via OIDC).
* `session_token` - _(optional)_ AWS session token. Required when using temporary credentials from STS or OIDC.
* `region` - AWS region
* `stack_name` - Name of the stack to fetch outputs from
* `prefix` - _(optional)_ When set, prefixes each exported variable name with the given string (e.g. `prefix: MYPREFIX` exports `MYPREFIX_my_var` instead of `my_var`). Must contain uppercase letters only (A-Z). Useful when fetching outputs from multiple stacks in the same workflow.
* `scope` - _(optional)_ What to collect from the stack. Accepts `Outputs` (default) or `Parameters`.

### Outputs

Each stack output key is set as both an environment variable (via `$GITHUB_ENV`) and a step output (via `$GITHUB_OUTPUT`). Since the keys are determined at runtime by the stack, they cannot be declared statically — but they are accessible in subsequent steps as `${{ steps.<id>.outputs.MyOutputKey }}` or via the environment as `${{ env.MyOutputKey }}`.

## Examples

### Example workflow (OIDC)

The workflow below authenticates via OIDC and fetches stack outputs from `MyStack`.

```yaml
name: Print Stack Outputs
on: push
permissions:
  id-token: write
  contents: read
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/MyRole
          aws-region: us-west-2
      - uses: accendero/aws-stack-outputs-to-env@v2.1
        id: stack-outputs
        with:
          region: "us-west-2"
          stack_name: "MyStack"
      # Access via step outputs
      - run: echo "${{ steps.stack-outputs.outputs.StackOutputOne }}, ${{ steps.stack-outputs.outputs.StackOutputTwo }}"
      # Or via environment variables
      - run: echo "${{ env.StackOutputOne }}, ${{ env.StackOutputTwo }}"
```

### Example workflow (static credentials)

```yaml
name: Print Stack Outputs
on: push
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: accendero/aws-stack-outputs-to-env@v2.1
        id: stack-outputs
        with:
          access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: ${{ secrets.AWS_DEFAULT_REGION }}
          stack_name: "MyStack"
      # Access via step outputs
      - run: echo "${{ steps.stack-outputs.outputs.StackOutputOne }}, ${{ steps.stack-outputs.outputs.StackOutputTwo }}"
      # Or via environment variables
      - run: echo "${{ env.StackOutputOne }}, ${{ env.StackOutputTwo }}"
```

## Development

### Running tests locally

This project uses [act](https://github.com/nektos/act) to run GitHub Actions locally.

#### Install act

If you follow the linux instructions below, it may install to the local directory you are running the installation
command from, and the resulting binary will be in `./bin/act`. You can move this to a directory in your PATH or add the
local directory to your PATH to run `act` from anywhere.

Additionally, act requires docker to run the GitHub Actions locally, so make sure you have Docker installed and
running on your machine.

```bash
# macOS
brew install act

# Linux
curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Other methods: https://github.com/nektos/act#installation
```

#### Configure act

Run the setup script to generate `.actrc` with the correct Docker socket path for your system:

```bash
./setup-act.sh
```

This will auto-detect your Docker socket location (works with Docker Desktop and standard installations).

#### Run tests

The test suite includes validation tests that don't require AWS credentials, plus an integration
test that requires real AWS access.

```bash
# Run validation tests (no AWS credentials needed)
./bin/act -j test-validation
```

#### Run integration tests with AWS

To run the CloudFormation integration tests, you need AWS credentials and an existing stack.

1. Copy the secrets example file:
   ```bash
   cp .secrets.example .secrets
   ```

2. Edit `.secrets` with your AWS credentials and stack name:
   ```
   AWS_ACCESS_KEY_ID=your-access-key-id
   AWS_SECRET_ACCESS_KEY=your-secret-access-key
   AWS_REGION=us-east-1
   STACK_NAME=your-stack-name
   ```

3. Run the integration test:
   ```bash
   ./bin/act workflow_dispatch -j test-cfn-fetch --secret-file .secrets
   ```

Alternatively, pass secrets directly:
```bash
./bin/act workflow_dispatch -j test-cfn-fetch \
  -s AWS_ACCESS_KEY_ID=xxx \
  -s AWS_SECRET_ACCESS_KEY=xxx \
  -s AWS_REGION=us-east-1 \
  -s STACK_NAME=your-stack-name
```