---
layout: "vault"
page_title: "Vault: vault_aws_auth_backend_client resource"
sidebar_current: "docs-vault-resource-aws-auth-backend-client"
description: |-
  Configures the client used by an AWS Auth Backend in Vault.
---

# vault\_aws\_auth\_backend\_client

Configures the client used by an AWS Auth Backend in Vault.

This resource sets the access key and secret key that Vault will use
when making API requests on behalf of an AWS Auth Backend. It can also
be used to override the URLs Vault uses when making those API requests.

For more information, see the
[Vault docs](https://www.vaultproject.io/api-docs/auth/aws#configure-client).

~> **Important** All data provided in the resource configuration will be
written in cleartext to state and plan files generated by Terraform, and
will appear in the console output when Terraform runs. Protect these
artifacts accordingly. See
[the main provider documentation](../index.html)
for more details.

## Example Usage

You can setup the AWS auth engine with Workload Identity Federation (WIF) for a secret-less configuration:
```hcl
resource "vault_auth_backend" "example" {
  type = "aws"
}

resource "vault_aws_auth_backend_client" "example" { 
  identity_token_audience = "<TOKEN_AUDIENCE>"
  identity_token_ttl      = "<TOKEN_TTL>"
  role_arn                = "<AWS_ROLE_ARN>"
  rotation_schedule       = "0 * * * SAT"
  rotation_window         = 3600
}
```

```hcl
resource "vault_auth_backend" "example" {
  type = "aws"
}

resource "vault_aws_auth_backend_client" "example" {
  backend           = vault_auth_backend.example.path
  access_key        = "INSERT_AWS_ACCESS_KEY"
  secret_key        = "INSERT_AWS_SECRET_KEY"
  rotation_schedule = "0 * * * SAT"
  rotation_window   = 3600
}
```

## Argument Reference

The following arguments are supported:

* `namespace` - (Optional) The namespace to provision the resource in.
  The value should not contain leading or trailing forward slashes.
  The `namespace` is always relative to the provider's configured [namespace](/docs/providers/vault/index.html#namespace).
   *Available only for Vault Enterprise*.

* `backend` - (Optional) The path the AWS auth backend being configured was
    mounted at.  Defaults to `aws`.

* `access_key` - (Optional) The AWS access key that Vault should use for the
    auth backend. Mutually exclusive with `identity_token_audience`.

* `secret_key` - (Optional) The AWS secret key that Vault should use for the
    auth backend.

* `identity_token_audience` - (Optional) The audience claim value. Mutually exclusive with `access_key`. 
    Requires Vault 1.17+. *Available only for Vault Enterprise*

* `identity_token_ttl` - (Optional) The TTL of generated identity tokens in seconds. Requires Vault 1.17+.
    *Available only for Vault Enterprise*

* `role_arn` - (Optional) Role ARN to assume for plugin identity token federation. Requires Vault 1.17+.
    *Available only for Vault Enterprise*

* `ec2_endpoint` - (Optional) Override the URL Vault uses when making EC2 API
    calls.

* `iam_endpoint` - (Optional) Override the URL Vault uses when making IAM API
    calls.

* `sts_endpoint` - (Optional) Override the URL Vault uses when making STS API
    calls.

* `sts_region` - (Optional) Override the default region when making STS API 
    calls. The `sts_endpoint` argument must be set when using `sts_region`.

* `use_sts_region_from_client` - (Optional) Available in Vault v1.15+. If set, 
    overrides both `sts_endpoint` and `sts_region` to instead use the region
    specified in the client request headers for IAM-based authentication.
    This can be useful when you have client requests coming from different 
    regions and want flexibility in which regional STS API is used.

* `iam_server_id_header_value` - (Optional) The value to require in the
    `X-Vault-AWS-IAM-Server-ID` header as part of `GetCallerIdentity` requests
    that are used in the IAM auth method.

* `max_retries` - (Optional) Number of max retries the client should use for recoverable errors. 
    The default `-1` falls back to the AWS SDK's default behavior.

* `rotation_period` - (Optional) The amount of time in seconds Vault should wait before rotating the root credential.
    A zero value tells Vault not to rotate the root credential. The minimum rotation period is 10 seconds. Requires Vault Enterprise 1.19+.

* `rotation_schedule` - (Optional) The schedule, in [cron-style time format](https://en.wikipedia.org/wiki/Cron),
    defining the schedule on which Vault should rotate the root token. Requires Vault Enterprise 1.19+.

* `rotation_window` - (Optional) The maximum amount of time in seconds allowed to complete
    a rotation when a scheduled token rotation occurs. The default rotation window is
    unbound and the minimum allowable window is `3600`. Requires Vault Enterprise 1.19+.

* `disable_automated_rotation` - (Optional) Cancels all upcoming rotations of the root credential until unset. Requires Vault Enterprise 1.19+.

## Attributes Reference

No additional attributes are exported by this resource.

## Import

AWS auth backend clients can be imported using `auth/`, the `backend` path, and `/config/client` e.g.

```
$ terraform import vault_aws_auth_backend_client.example auth/aws/config/client
```
