# Learn Vault

https://learn.hashicorp.com/vault

## Install Vault

[Vault must first be installed on your machine](https://www.vaultproject.io/downloads.html) 

:ship: To install completions, run:
```bash
vault -autocomplete-install
```

## Starting the Server

Vault operates as a client/server application. 

First, we're going to start a Vault _dev server_. The dev server is a built-in, pre-configured server that is not very secure but useful for playing with Vault locally. Later in this guide we'll configure and start a real server.

:warning: `dev server` is not very secure but can be used for local testing

:warning: Do not run a dev server in production!

:ship: Start the Vault `dev server` for local testing.
```bash
vault server -dev
```

:warning: The dev server stores all its data in-memory (but still encrypted),
listens on `localhost` without TLS, and automatically unseals and shows you the
unseal key and root access key.

:ship: Set necessary environment tokens
```bash

 # see printed VAULT_ADDR
export VAULT_ADDR='url'

 # See printed Root Token
export VAULT_DEV_ROOT_TOKEN_ID="token"

 # See printed Unseal Key
echo "key" > unseal.key

 # Verify server is running
vault status
```

## Your First Secret

[HTTP API](https://www.vaultproject.io/api/index.html) can be used to
programmatically do anything with Vault.

Secrets written to Vault are encrypted and then written to backend storage. For
our dev server, backend storage is in-memory, but in production this would more
likely be on disk or in [Consul](https://www.consul.io/). 

Vault encrypts the value before it is ever handed to the storage driver. The
backend storage mechanism _never_ sees the unencrypted value and doesn't have
the means necessary to decrypt it without Vault.

:ship: Let's start by writing a secret. 
```bash
vault kv put secret/hello foo=world
```

:ship: writing multiple secrets
```bash
vault kv put secret/hello foo=world excited=yes
```

[command documentation](https://www.vaultproject.io/docs/commands/index.html)
for example of reading from `STDIN`.

:ship: Reading a secret
```bash
vault kv get secret/hello
```

The output format is purposefully whitespace separated to make it easy to pipe
into a tool like `awk`.

:ship: To print only the value of a given field:
```bash
vault kv get -field=excited secret/hello
```

:ship: Optional JSON output with `-format=json`. For example below we use the
`jq` tool to extract the value of the `excited` secret:
```bash
vault kv get -format=json secret/hello | jq -r .data.data.excited
```

:ship: Deleting a Secret
```bash
vault kv delete secret/hello
```

## Secrets Engines

Previously, we saw how to read and write arbitrary secrets to Vault. You may
have noticed all requests started with `secret/`. Try the following command
which will result an error:


:ship: All requests must start with `secret/`
```bash
vault kv put foo/bar a=b

    Error making API request.

    URL: GET http://localhost:8200/v1/sys/internal/ui/mounts/foo/bar
    Code: 403. Errors:

    * preflight capability check returned 403, ... grant access to path "foo/bar/"
```

By default, Vault enables [Key/Value version2 secrets
engine](https://www.vaultproject.io/docs/secrets/kv/kv-v2/) (`kv-v2`) at the
path `secret/` when running in `dev` mode. 

**NOTE:** The key/value secrets engine has two versions: `kv` (version 1) and
`kv-v2` (version 2). The `kv-v2` is versioned `kv` secrets engine which can
retain a number of secrets versions.

:ship: Enable a `kv` secrets engine
```bash
vault secrets enable -path=kv kv
```

:ship: you can leave off `-path` as it defaults to the name
```bash
vault secrets enable kv
```

:ship: To verify our success and get more information about the secrets engine 
```bash
vault secrets list
```

:ship: testing 
```bash
vault kv put kv/hello target=world
vault kv get kv/hello

vault kv put kv/my-secret value="s3c(eT"
vault kv get kv/my-secret
```

:ship: Delete a secret
```bash
vault kv delete kv/my-secret
```

:ship: List all secrets in a store
```bash
vault kv list kv/   # vault TYPE list PATH/
```

:ship: disable a secrets engine. :warning: this takes `PATH/` as the argument,
not the `TYPE` 
```bash
vault secrets disable kv/   # vault secrets disable PATH/
```

As mentioned above, Vault behaves similarly to a [virtual
filesystem](https://en.wikipedia.org/wiki/Virtual_file_system). The
read/write/delete/list operations are forwarded to the corresponding secrets
engine, and the secrets engine decides how to react to those operations.

This abstraction is incredibly powerful. It enables Vault to interface directly
with physical systems, databases, HSMs, etc. But in addition to these physical
systems, Vault can interact with more unique environments like AWS IAM, dynamic
SQL user creation, etc. all while using the same read/write interface.

## Dynamic Secrets

:fire: **dynamic secrets** are generated when they are accessed. Dynamic
secrets do not exist until they are read, so there is no risk of someone
stealing them or another client using the same secrets. 

:ship: Enable AWS secrets engine. :warning: AWS secrets engine must be
explicitly enabled
```bash
vault secrets enable -path=aws aws

vault secrets list
```

The AWS secrets engine is now enabled at `aws/`. As we covered in the previous
sections, different secrets engines allow for different behavior. In this case,
the AWS secrets engine generates dynamic, on-demand AWS access credentials.

:warning: Do not use your root account keys in production. This is a getting
started guide and is not a best practices guide for production installations.

:flashlight: using creds stored in `~/.aws/credentials`

:ship: configure AWS secrets engine with [administrator
credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)
```bash
vault write aws/config/root \
  access_key=$(aws configure get aws_access_key_id) \
  secret_key=$(aws configure get aws_secret_access_key) \
  region=us-east-1
```

:flashlight: Example IAM policy to enable all actions on EC2
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1426528957000",
      "Effect": "Allow",
      "Action": ["ec2:*"],
      "Resource": ["*"]
    }
  ]
}
```

:ship: Create a role. We need to map this policy document to a named role. To
do that, write to `aws/roles/:name` where `:name` is your unique name that
describes the role (such as `aws/roles/my-role`):
```bash
vault write aws/roles/vault-user \
credential_type=iam_user \
policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1426528957000",
      "Effect": "Allow",
      "Action": [
        "ec2:*"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}
EOF
```

We just told Vault when I ask for a credential for "my-role", create it and
attach the IAM policy `{ "Version": "2012..." }`.

:ship: Generate an access key pair for the role by reading `aws/creds/:name`
```bash
vault read aws/creds/my-role
```

:fire: The access and secret key can now be used to perform any EC2 operations
within AWS.

:warning: Notice that these keys are new, they are not the keys you entered
earlier. If you were to run the command a second time, you would get a new
access key pair. 

:exclamation: Each time you read from `aws/creds/:name`, Vault will connect to
AWS and generate a new IAM user and key pair.

Copy the full path of this `lease_id` value found in the output. This value is
used for renewal, revocation, and inspection.

:warning: Vault will automatically revoke this credential after 768 hours (see
`lease_duration` in the output), but perhaps we want to revoke it early. Once
the secret is revoked, the access keys are no longer valid.

:ship: Revoke the secret with `vault revoke`
```bash
vault lease revoke aws/creds/my-role/0bce0782-32aa-25ec-f61d-c026ff22106
```

## Built-in Help

:ship: Secrets Engines Overview
```bash
vault path-help aws   # overview of aws secrets engine
```

:ship: Path Help

```bash
vault path-help aws/creds/my-non-existent-role
```

## Authentication

When starting the Vault server in `dev` mode, it automatically logs you in as
the root user with admin permissions. In a non-dev setup, you would have had to
authenticate first.

We will use the token auth method and the GitHub auth method.

:ship: You can create more tokens using the `vault token create` command.
```bash
vault token create
```

:flashlight: By default, this will create a child token of your current token
that inherits all the same policies. 

:flashlight: tokens always have a parent, and when that parent token is
revoked, children can also be revoked all in one operation. 

:ship: To authenticate with a token, execute the `vault login` command.
```bash
vault login TOKEN
```

:ship: After a token is created, you can revoke it.
```bash
vault token revoke TOKEN
```
 
:ship: Log back in with root token.
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

### Recommended Patterns

:ship: GitHub Auth 
```bash
vault auth enable -path=github github

vault auth enable -path=github github
    
vault auth enable github

vault auth enable -path=my-github github

vault write auth/github/config organization=hashicorp

vault write auth/github/map/teams/my-team value=default,my-policy
    # Map policies to a team within the organization.

```

:ship: Find which auth methods are enabled and available.
```bash
vault auth list

vault auth help github

vault auth help aws

vault auth help userpass

vault auth help token
```
    
:ship: Authenticate to Github
```bash
vault login -method=github
```

:ship: You can revoke any logins from an auth method using `vault token revoke`
with the `-mode` argument. For example:
```bash
vault token revoke -mode path auth/github
```

:ship: Alternatively, if you want to completely disable the GitHub auth method,
execute the following command.
```bash
vault auth disable github
```
    
## Policies

Policies in Vault control what a user can access. 

There are some built-in policies that cannot be removed. For example, the
`root` and `default` policies are required policies and cannot be deleted. 
- The `default` policy provides a common set of permissions and is included on
  all tokens by default. 
- The `root` policy gives a token super admin permissions, similar to a root
  user on a linux machine.

:ship: Policies are authored in [HCL](https://github.com/hashicorp/hcl), but
are JSON compatible. Here is an example policy:
```json
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
```
    
With this policy, a user could write any secret to `secret/data/`, except to
`secret/data/foo`, where only read access is allowed. 

Policies default to deny, so any access to an unspecified path is not allowed.

:ship: Vault includes a command that will format the policy automatically
according to specification. It also reports on any syntax errors.
```bash
vault policy fmt my-policy.hcl
```
    
:ship: To write a policy using the command line, specify the path to a policy
file to upload.
```bash
vault policy write my-policy my-policy.hcl
```

:ship: Here is an example you can copy-paste in the terminal.
```bash
vault policy write my-policy -<<EOF
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
EOF
```

:ship: To see the list of policies
```bash
vault policy list
```

:ship: To view the contents of a policy
```bash
vault policy read my-policy
```
    
#### Testing the Policy

:ship: First, check to verify that KV v2 secrets engine has not been enabled at `secret/`.
```bash
vault secrets list

```

:ship: To use the policy, create a token and assign it to that policy.
```bash
vault token create -policy=my-policy
```

:ship: Copy the generated token value and authenticate with Vault.
```bash
vault login s.X6gvFko7chPilgV0lpWXsdeu
```
    
:ship: Verify that you can write any data to `secret/data/`.
```bash
vault kv put secret/creds password="my-long-password"
```

:ship: Since `my-policy` only permits read from the `secret/data/foo` path, any
attempt to write fails with "permission denied" error.
```bash
vault kv put secret/foo robot=beepboop
```
    
:ship: Re-authenticate with the initial root token to continue.
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```
    
#### Mapping Policies to Auth Methods

Vault itself is the single policy authority, unlike authentication where you
can enable multiple auth methods. Any enabled auth method must map identities
to these core policies.

:ship: Use the `vault path-help` system with your auth method to determine how the
mapping is done since it is specific to each auth method. For example, with
GitHub, it is done per team using the `map/teams/<team>` path.
```bash
vault write auth/github/map/teams/default value=my-policy
```

## Deploy Vault

On this page, we'll cover how to configure Vault, start Vault, the seal/unseal
process, and scaling Vault.

:ship: Copy to `config.hcl`
```bash
    storage "consul" {            # Uses Consul backend
      address = "127.0.0.1:8500"
      path    = "vault/"
    }
    
    listener "tcp" {
     address     = "127.0.0.1:8200"
     tls_disable = 1
    }
```


:ship:  [Consul Getting Started
Guide](https://www.consul.io/intro/getting-started/install.html) to install and
start Consul 
```bash
consul agent -dev
```

Within the configuration file, there are two primary configurations:

* [`storage`](#storage) - This is the physical backend that Vault uses for
  storage. Up to this point the dev server has used "inmem" (in memory), but
  the example above uses [Consul](https://www.consul.io/), a much more
  production-ready backend.
    
* [`listener`](#listener) - One or more listeners determine how Vault listens
  for API requests. The example above listens on localhost port 8200 without
  TLS. In your environment set `VAULT_ADDR=http://127.0.0.1:8200` so the Vault
  client will connect without TLS.

:ship: Starting the Server
```bash
vault server -config=config.hcl
```

:warning: If you get a warning message about mlock not being supported, that is
okay. However, for maximum security you should run Vault on a system that
supports mlock.

Initialization is the process configuring the Vault. This only happens once
when the server is started against a new backend that has never been used with
Vault before. When running in HA mode, this happens once _per cluster_, not
_per server_.


:ship: During initialization, the encryption keys are generated, unseal keys
are created, and the initial root token is setup.  This is an _unauthenticated_
request, but it only works on brand new Vaults with no data:
```bash
vault operator init
```

Initialization outputs two important pieces of information:
- _unseal keys_ 
- _initial root token_. 

:warning: This is the **only time ever** that all of this data is known by
Vault, and also the only time that the unseal keys should ever be so close
together.

:exclamation: For the purpose of this getting started guide, save all of these
keys somewhere, and continue. In a real deployment scenario, you would never
save these keys together. Instead, you would likely use Vault's PGP and
Keybase.io support to encrypt each of these keys with the users' PGP keys. This
prevents one single person from having all the unseal keys. Please see the
documentation on [using PGP, GPG, and
Keybase](https://www.vaultproject.io/docs/concepts/pgp-gpg-keybase.html) for
more information.

### Seal/Unseal

Every initialized Vault server starts in the _sealed_ state. From the
configuration, Vault can access the physical storage, but it can't read any of
it because it doesn't know how to decrypt it. 

The process of teaching Vault how to decrypt the data is known as **unsealing**
the Vault.

:flashlight: Vault does not store any of the unseal key shards. Vault uses an
algorithm known as [Shamir's Secret
Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) to split the
master key into shards. Only with the threshold number of keys can it be
reconstructed and your data finally accessed.

Begin unsealing the Vault:

:ship: Unseal the vault
```bash
vault operator unseal
```

:flashlight: Also notice that the unseal process is stateful. You can go to
another computer, use `vault operator unseal`, and as long as it's pointing to
the same server, that other computer can continue the unseal process. This is
incredibly important to the design of the unseal process: multiple people with
multiple keys are required to unseal the Vault. The Vault can be unsealed from
multiple computers and the keys should never be together. A single malicious
operator does not have enough keys to be malicious.

:ship: 
Continue with `vault operator unseal` to complete unsealing the Vault. To unseal the vault you must use three _different_ unseal keys, the same key repeated will not work.
```bash
vault operator unseal
```

As you use keys, as long as they are correct, you should soon see output like this:

    Key                    Value
    ---                    -----
    Seal Type              shamir
    Initialized            true
    Sealed                 false
    Total Shares           5
    Threshold              3
    Version                1.3.4
    Cluster Name           vault-cluster-df0d8ee9
    Cluster ID             c91bc0f4-3500-8293-23f4-ab6e1ec317a3
    HA Enabled             true
    HA Cluster             n/a
    HA Mode                standby
    Active Node Address    <none>
    

When the value for `Sealed` changes to `false`, the Vault is unsealed.

:ship: Finally, authenticate as the initial root token (it was included in the
output with the unseal keys):
```bash
vault login s.KkNJYWF5g0pomcCLEmDdOVCW
```

As a root user, you can reseal the Vault with `vault operator seal`. A single
operator is allowed to do this. 

When the Vault is sealed again, it clears all of its state (including the
encryption key) from memory. The Vault is secure and locked down from access.

## Using the HTTP APIs with Authentication | Vault

All of Vault's capabilities are accessible via the HTTP API in addition to the CLI. In fact, most calls from the CLI actually invoke the HTTP API. In some cases, Vault features are not available via the CLI and can only be accessed via the HTTP API.

Once you have started the Vault server, you can use Client URL (cURL) or any other http client to make API calls. For example, if you started the Vault server in [dev mode](https://www.vaultproject.io/docs/concepts/dev-server.html), you could validate the initialization status like this:

    $ curl http://127.0.0.1:8200/v1/sys/init
    

This will return a JSON response:

    {
      "initialized": true
    }
    

:ship: Accessing Secrets via the REST APIs
----------------------------------------------------------------------------

Machines that need access to information stored in Vault will most likely access Vault via its REST API. For example, if a machine were using [AppRole](https://www.vaultproject.io/docs/auth/approle.html) for authentication, the application would first authenticate to Vault which would return a Vault API token. The application would use that token for future communication with Vault.

For the purpose of this guide, we will use the following configuration which disables TLS and uses a file-based backend. TLS is disabled here only for example purposes; it should never be disabled in production.

    backend "file" {
      path = "vault"
    }
    
    listener "tcp" {
      tls_disable = 1
    }
    

Save this file on disk as `config.hcl`. Stop the Vault server instance that you previously started and then start a new instance using the newly created configuration.

    $ vault server -config=config.hcl
    

At this point, we can use Vault's API for all our interactions. For example, we can initialize Vault like this.

**NOTE:** This example uses [jq](https://stedolan.github.io/jq/download/) to process the JSON output for readability.

    $ curl \
        --request POST \
        --data '{"secret_shares": 1, "secret_threshold": 1}' \
        http://127.0.0.1:8200/v1/sys/init | jq
    

The response should be JSON and looks something like this:

    {
      "keys": [
        "ff27b63de46b77faabba1f4fa6ef44c948e4d6f2ea21f960d6aab0eb0f4e1391"
      ],
      "keys_base64": [
        "/ye2PeRrd/qruh9Ppu9EyUjk1vLqIflg1qqw6w9OE5E="
      ],
      "root_token": "s.Ga5jyNq6kNfRMVQk2LY1j9iu"
    }
    

This response contains our initial root token. It also includes the unseal key. You can use the unseal key to unseal the Vault and use the root token perform other requests in Vault that require authentication.

To make this guide easy to copy-and-paste, we will be using the environment variable `$VAULT_TOKEN` to store the root token. You can export this Vault token in your current shell like this:

    $ export VAULT_TOKEN="s.Ga5jyNq6kNfRMVQk2LY1j9iu"
    

Using the unseal key (not the root token) from above, you can unseal the Vault via the HTTP API:

    $ curl \
        --request POST \
        --data '{"key": "/ye2PeRrd/qruh9Ppu9EyUjk1vLqIflg1qqw6w9OE5E="}' \
        http://127.0.0.1:8200/v1/sys/unseal | jq
    

Note that you should replace `/ye2PeRrd/qru...` with the generated key from your output. This will return a JSON response:

    {
      "type": "shamir",
      "initialized": true,
      "sealed": false,
      "t": 1,
      "n": 1,
      "progress": 0,
      "nonce": "",
      "version": "1.3.2",
      "migration": false,
      "cluster_name": "vault-cluster-a90e2cd2",
      "cluster_id": "0bc4b0fa-f876-8069-8d45-e0daae74e90e",
      "recovery_seal": false,
      "storage_type": "file"
    }
    

Now any of the available auth methods can be enabled and configured. For the purposes of this guide lets enable [AppRole](https://www.vaultproject.io/docs/auth/approle.html) authentication.

The [Authentication](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/authentication#auth-methods) guide showed how to enable the GitHub auth method using Vault CLI.

    $ vault auth enable <auth_method_type>
    

To see the cURL equivalent of the CLI command to enable AppRole auth method, use the `-output-curl-string` flag.

    $ vault auth enable -output-curl-string approle
    

Enable the AppRole auth method by invoking the Vault API.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{"type": "approle"}' \
        http://127.0.0.1:8200/v1/sys/auth/approle
    

Notice that the request to enable the AppRole endpoint needed an authentication token. In this case we are passing the root token generated when we started the Vault server. We could also generate tokens using any other authentication mechanisms, but we will use the root token for simplicity.

Now create an AppRole with desired set of [ACL policies](https://www.vaultproject.io/docs/concepts/policies.html).

The [Policies](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies) guide used CLI to create `my-policy`. In this guide, use the `/sys/policies/acl` endpoint to create the same policy via Vault API.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request PUT \
        --data '{"policy":"# Dev servers have version 2 of KV secrets engine mounted by default, so will\n# need these paths to grant permissions:\npath \"secret/data/*\" {\n  capabilities = [\"create\", \"update\"]\n}\n\npath \"secret/data/foo\" {\n  capabilities = [\"read\"]\n}\n"}' \
        http://127.0.0.1:8200/v1/sys/policies/acl/my-policy
    

Since `my-policy` expects `secret/data` path to exist, enable KV v2 secrets engine at `secret/` using API.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{ "type":"kv-v2" }' \
        http://127.0.0.1:8200/v1/sys/mounts/secret
    

The following command specifies that the tokens issued under the AppRole `my-role` should be associated with `my-policy`.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{"policies": ["my-policy"]}' \
        http://127.0.0.1:8200/v1/auth/approle/role/my-role
    

The AppRole auth method expects a RoleID and a SecretID as its input. The RoleID is similar to a username and the SecretID can be thought as the RoleID's password.

The following command fetches the RoleID of the role named `my-role`.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
         http://127.0.0.1:8200/v1/auth/approle/role/my-role/role-id | jq -r ".data"
    

The response will include the `role_id`:

    {
      "role_id": "3c301960-8a02-d776-f025-c3443d513a18"
    }
    

This command creates a new SecretID under the `my-role`.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        http://127.0.0.1:8200/v1/auth/approle/role/my-role/secret-id | jq -r ".data"
    

The response will include the `secret_id`:

    {
      "secret_id": "22d1e0d6-a70b-f91f-f918-a0ee8902666b",
      "secret_id_accessor": "726ab786-70d0-8cc4-e775-c0a75070e5e5"
    }
    

These two credentials can be supplied to the login endpoint to fetch a new Vault token.

    $ curl --request POST \
           --data '{"role_id": "c3ec4eab-5477-c669-fca8-6a71fdf38c23", "secret_id": "fc2710e5-9536-3f4f-666d-fd5d8379b2b9"}' \
           http://127.0.0.1:8200/v1/auth/approle/login | jq -r ".auth"
    

The response will be JSON, under the key `auth`:

    {
      "client_token": "s.p5NB4dTlsPiUU94RA5IfbzXv",
      "accessor": "EQTlZwOD4yIFYWIg5YY6Xr29",
      "policies": [
        "default",
        "my-policy"
      ],
      "token_policies": [
        "default",
        "my-policy"
      ],
      "metadata": {
        "role_name": "my-role"
      },
      "lease_duration": 2764800,
      "renewable": true,
      "entity_id": "4526701d-b8fd-3c39-da93-9e17506ec894",
      "token_type": "service",
      "orphan": true
    }
    

The returned client token (`s.p5NB4dTlsPiUU94RA5IfbzXv`) can be used to authenticate with Vault. This token will be authorized with specific capabilities on all the resources encompassed by the `default` and `my-policy` policies. (As it was mentioned in the [Policies](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies) guide, the `default` policy is attached to all tokens by default. )

The newly acquired token can be exported as the `VAULT_TOKEN` environment variable value and used to authenticate subsequent Vault requests.

    $ export VAULT_TOKEN="s.p5NB4dTlsPiUU94RA5IfbzXv"
    

Create a version 1 of secret named `creds` with a key `password` and its value set to `my-long-password`.

    $ curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{ "data": {"password": "my-long-password"} }' \
        http://127.0.0.1:8200/v1/secret/data/creds | jq -r ".data"
    

    {
      "created_time": "2020-02-05T16:51:34.0887877Z",
      "deletion_time": "",
      "destroyed": false,
      "version": 1
    }
    

You can stop the server and unset the `VAULT_TOKEN` environment variable.

    $ unset VAULT_TOKEN
    

You can see the documentation on the [HTTP APIs](https://www.vaultproject.io/api/index.html) for more details on other available endpoints.

Congratulations! You now know all the basics needed to get started with Vault.> Vault comes with support for a user-friendly and functional web UI out of the box. In this guide we will explore the Vault UI.


## Web UI | Vault - HashiCorp Learn
Vault features a user interface (web interface) for interacting with Vault. Easily create, read, update, and delete secrets, authenticate, unseal, and more with the Vault UI.

:ship: Dev servers
----------------------------

When you start the Vault server in dev mode, Vault UI is automatically enabled and ready to use.

    $ vault server -dev
    ...
    

Open a web browser and enter `http://127.0.0.1:8200/ui` to launch the UI.

Enter the initial root token to sign in.

![Sign in](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/img/vault-autounseal-3.png)

:ship: Non-Dev servers
------------------------------------

The Vault UI is not activated by default. To activate the UI, set the `ui` configuration option in the Vault server configuration.

    ui = true
    
    listener "tcp" {
      
    }
    
    storage "consul" {
      
    }
    

The UI runs on the same port as the Vault listener. As such, you must configure at least one `listener` stanza in order to access the UI.

**Example:**

    ui = true
    
    listener "tcp" {
      address = "10.0.1.35:8200"
    
      
      
      
    }
    
    

In this case, the UI is accessible the following URL from any machine on the subnet (provided no network firewalls are in place): `https://10.0.1.35:8200/ui`

It is also accessible at any DNS entry that resolves to that IP address, such as the Consul service address (if using Consul): `https://vault.service.consul:8200/ui`

:ship: Web UI Wizard
--------------------------------

Vault UI has a built-in guide to navigate you through the common steps to operate various Vault features.

![Sign in](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/img/vault-ui-wizard.png)> Resources and further tracks now that you're confident using Vault.


## Next Steps | Vault - HashiCorp Learn
[](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault)

[Learn Vault](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault)

/Getting Started

*   [Getting Started](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=getting-started#getting-started)
*   [Kubernetes](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=getting-started-k8s#getting-started-k8s)
*   [Product Certification Exam Prep](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=certification#certification)
*   [Vault 1.4 Release Highlights](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=new-release#new-release)
*   [Day 1: Deploying Your First Vault Cluster](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=day-one#day-one)
*   [Secrets Management](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=secrets-management#secrets-management)
*   [Advanced Data Protection](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=ADP#ADP)
*   [Data Encryption](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=encryption-as-a-service#encryption-as-a-service)
*   [Access Management](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=identity-access-management#identity-access-management)
*   [Monitoring & Troubleshooting](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=monitoring#monitoring)
*   [Security](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=security#security)
*   [Operations](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=operations#operations)
*   [Developer](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=developer#developer)

/Next Steps

*   [
    
    ###### Install Vault
    
    2 minThe first step to using Vault is to get it installed.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/install)
*   [
    
    ###### Starting the Server
    
    5 minAfter installing Vault, the next step is to start the server.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/dev-server)
*   [
    
    ###### Your First Secret
    
    5 minWith the Vault server running, let's read and write our first secret.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/first-secret)
*   [
    
    ###### Secrets Engines
    
    5 minSecrets engines create, read, update, and delete secrets.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/secrets-engines)
*   [
    
    ###### Dynamic Secrets
    
    10 minOn this page we introduce dynamic secrets by showing you how to create AWS access keys with Vault.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/dynamic-secrets)
*   [
    
    ###### Built-in Help
    
    5 minVault has a built-in help system to learn about the available paths in Vault and how to use them.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/help)
*   [
    
    ###### Authentication
    
    5 minUsers can authenticate to Vault using multiple methods.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/authentication)
*   [
    
    ###### Policies
    
    10 minPolicies in Vault control what a user can access.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies)
*   [
    
    ###### Deploy Vault
    
    5 minLearn how to deploy Vault, including configuring, starting, initializing, and unsealing it.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/deploy)
*   [
    
    ###### Using the HTTP APIs with Authentication
    
    5 minHTTP APIs can control authentication and access to secrets.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/apis)
*   [
    
    ###### Web UI
    
    5 minVault comes with support for a user-friendly and functional web UI out of the box. In this guide we will explore the Vault UI.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/ui)
*   [
    
    ###### Next Steps
    
    2 minResources and further tracks now that you're confident using Vault.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/next-steps)

*   [
    
    •
    
    Overview](#overview)

Getting Started

*   [
    
    ###### Install Vault
    
    2 minThe first step to using Vault is to get it installed.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/install)
*   [
    
    ###### Starting the Server
    
    5 minAfter installing Vault, the next step is to start the server.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/dev-server)
*   [
    
    ###### Your First Secret
    
    5 minWith the Vault server running, let's read and write our first secret.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/first-secret)
*   [
    
    ###### Secrets Engines
    
    5 minSecrets engines create, read, update, and delete secrets.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/secrets-engines)
*   [
    
    ###### Dynamic Secrets
    
    10 minOn this page we introduce dynamic secrets by showing you how to create AWS access keys with Vault.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/dynamic-secrets)
*   [
    
    ###### Built-in Help
    
    5 minVault has a built-in help system to learn about the available paths in Vault and how to use them.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/help)
*   [
    
    ###### Authentication
    
    5 minUsers can authenticate to Vault using multiple methods.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/authentication)
*   [
    
    ###### Policies
    
    10 minPolicies in Vault control what a user can access.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies)
*   [
    
    ###### Deploy Vault
    
    5 minLearn how to deploy Vault, including configuring, starting, initializing, and unsealing it.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/deploy)
*   [
    
    ###### Using the HTTP APIs with Authentication
    
    5 minHTTP APIs can control authentication and access to secrets.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/apis)
*   [
    
    ###### Web UI
    
    5 minVault comes with support for a user-friendly and functional web UI out of the box. In this guide we will explore the Vault UI.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/ui)
*   [
    
    ###### Next Steps
    
    2 minResources and further tracks now that you're confident using Vault.
    
    
    
    ](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/next-steps)

Getting Started

Time to Complete

2 minutes

Level

Getting Started

Products Used

Vault

That concludes the getting started guide for Vault. Hopefully you're excited about the possibilities of Vault and ready to put this knowledge to use to improve your environment.

We've covered the basics of all the core features of Vault in this guide. Due to the importance of securing secrets, we recommend reading the following as next steps.

*   [Documentation](https://www.vaultproject.io/docs/index.html) - The documentation is an in-depth reference guide to all the features of Vault.

Was this guide helpful?

[

Previous

Web UI



](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/ui)[

Keep Learning

Explore Operations & Development Tracks



](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault#operations-and-development)

On This Page

*   [
    
    •
    
    Overview](#overview)