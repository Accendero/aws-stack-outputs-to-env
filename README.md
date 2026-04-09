# Use Amazon AWS Cloudformation Stack Outputs as environment variables

This action will fetch AWS CloudFormation Stack outputs from a given stack and
export the outputs as environment variables. 

**The keys of the Stack outputs will be used as the environment variable names. 
This could potentially overwrite existing environment variables.**

- [Usage](#usage)
    - [Fetching Outputs](#fetching-outputs)
    - [Inputs](#inputs)
- [Examples](#examples)
    - [Example Workflow](#example-workflow)

## Usage

Add the entry below to your workflow file. The `stack_name` should be the
case-sensitive name of the stack you are querying in AWS CloudFormation.

### Fetching Outputs

If you already have an active AWS session (e.g. via OIDC or a prior credential
step), `access_key_id` and `secret_access_key` can be omitted entirely:

```yaml
uses: accendero/aws-stack-outputs-to-env@v1
id: <step id 1>
with:
  region: "us-west-2"
  stack_name: "MyStack"
```

To authenticate with static credentials, provide them explicitly:

```yaml
uses: accendero/aws-stack-outputs-to-env@v1
id: <step id 1>
with:
  access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
  secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  region: ${{ secrets.AWS_DEFAULT_REGION }}
  stack_name: "MyStack"
```

### Inputs

* `access_key_id` - _(optional)_ AWS access key ID. Omit if an active AWS session is already configured (e.g. via OIDC).
* `secret_access_key` - _(optional)_ AWS secret access key. Omit if an active AWS session is already configured (e.g. via OIDC).
* `region` - AWS region
* `stack_name` - Name of the stack to fetch outputs from

### Outputs

There are no outputs for this action. All the stack outputs will be exported
as environment variables.

## Examples

### Example workflow (OIDC)

The workflow below authenticates via OIDC and fetches stack outputs from `MyStack`.

```yaml
name: Print Stack Outputs
on: push
jobs:
  setup:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/MyRole
          aws-region: us-west-2
      - uses: accendero/aws-stack-outputs-to-env@v1
        id: stack-outputs
        with:
          region: "us-west-2"
          stack_name: "MyStack"
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
      - uses: accendero/aws-stack-outputs-to-env@v1
        id: stack-outputs
        with:
          access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: ${{ secrets.AWS_DEFAULT_REGION }}
          stack_name: "MyStack"
      - run: echo "${{ env.StackOutputOne }}, ${{ env.StackOutputTwo }}"
```
