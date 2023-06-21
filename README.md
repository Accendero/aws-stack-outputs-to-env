# Use Amazon AWS Cloudformation Stack Outputs as environment variables

This action will fetch AWS CloudFormation Stack outputs from a given stack and
export the outputs as environment variables. 

**The keys of the Stack outputs will be used as the environment variable names. 
This could potentially overwrite existing environment variables.**

- [Usage](#usage)
    - [Fetching Outputs](#fetching-outputs)
    - [Inputs](#inputs)
- [Examples](#examples)
    - [Storing the SSM parameters as JSON](#storing-the-ssm-parameters-as-json)
    - [Example Workflow](#example-workflow)

## Usage

Add the entry below to your workflow file. The `stack_name` should be the
case-sensitive name of the stack you are querying in AWS CloudFormation.

### Fetching Outputs

The example below will fetch the stack outputs from a stack named `MyStack` on
the `us-west-2` region and export the outputs as environment variables.

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

* `access_key_id` - AWS access key ID
* `secret_access_key` - AWS secret access key
* `region` - AWS region
* `stack_name` - Name of the stack to fetch outputs from

### Outputs

There are no outputs for this action. All the stack outputs will be exported
as environment variables.

## Examples

### Example workflow

The workflow below will fetch the stack outputs from `MyStack` and
store the outputs as environment variables. 

```yaml
name: Print Stack Outputs
on: push
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: accendero/aws-stack-outputs-to-env@v1
        id: stack-outputs
        with:
          access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: ${{ secrets.AWS_DEFAULT_REGION }}
          stack_name: "MyStack"
      - run: echo "${{ env.StackOutputOne }}, ${{ env.StackOutputTwo }}"
```      
