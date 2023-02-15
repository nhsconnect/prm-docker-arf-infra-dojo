# prm-docker-arf-infra-dojo

Dojo-compatible docker image that provides all tools required to build and deploy the Access Request Fulfilment infrastructure

[Dojo](https://github.com/kudulab/dojo) docker image with tools needed to deploy infrastructure in scope of access request fulfilment team.

Tested and released images are published to dockerhub as [nhsdev/arf-infra-dojo](https://hub.docker.com/r/nhsdev/arf-infra-dojo)

## Usage
1. Setup docker.
2. Install [Dojo](https://github.com/kudulab/dojo) binary.
3. Provide a Dojofile at the root of the project:
```
DOJO_DOCKER_IMAGE="nhsdev/arf-infra-dojo:<commit>"
```
4. Create and enter the container by running `dojo` at the root of project.

By default, current directory in docker container is `/dojo/work`.

### Access to AWS

In order to get sufficient access to work with terraform or AWS CLI:

Make sure to unset the AWS variables:
```
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
```

Set up a profile for each role you would like to assume in `~/.aws/config`.

As an example:

```
[profile default]
region = eu-west-2
output = json

[profile admin]
region = eu-west-2
role_arn = <role-arn>
mfa_serial = <mfa-arn>
source_profile = default
```

Your `source_profile` needs to match your profile in `~/.aws/credentials`.
```
[default]
aws_access_key_id = <your-aws-access-key-id>
aws_secret_access_key = <your-aws-secret-access-key>
```

You will need to assume the appropriate AWS role to use AWS CLI and terraform.

Run the following command to confirm you assumed the correct role:
`aws sts get-caller-identity`

Work with terraform as usual:
```
terraform init
terraform apply
```

If your session expires, exit the container to drop the temporary credentials and run `dojo` again.

# Specification

* base image is debian
* terraform binary on the PATH
* AWS CLI
* `jq` to parse JSON from bash scripts

