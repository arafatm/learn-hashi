# Learn Vault

Source https://learn.hashicorp.com/vault#getting-started

Docs https://www.vaultproject.io/docs

## cheatsheet

```bash
vault secrets list
vault kv list secret/
vault kv get secret/hello  
```

## Install Vault | https://learn.hashicorp.com/vault/getting-started/install

### Installing Vault

https://www.vaultproject.io/downloads.html 

### Verifying the Installation

:ship: Verify
```bash
vault
```

### Command Completion

:ship: To install completions, run:
```bash
vault -autocomplete-install

exec $SHELL
```

## Starting the Server | https://learn.hashicorp.com/vault/getting-started/dev-server

Vault operates as a client/server application. 

### Starting the Dev Server

:ship: To start the Vault dev server, run:
```bash
vault server -dev
```

Look for the following lines:

    $ export VAULT_ADDR='http://127.0.0.1:8200'

    Unseal Key: 1+yv+v5mz+aSCK67X6slL3ECxb4UDL8ujWZU/ONBpn0=
    Root Token: s.XmpNPoi9sRhYtdKHaQhkHP6x
:warning: Do not run a dev server in production!

:ship: With the dev server running, do the following four things before
anything else: 
1. `export VAULT_ADDR='VAULT_ADDR'`
2. Save the unseal key somewhere. 
3. `$ export VAULT_DEV_ROOT_TOKEN_ID="ROOT_TOKEN"`

### Verify the Server is Running

:ship:
```bash
vault status
```

## Your First Secret | https://learn.hashicorp.com/vault/getting-started/first-secret

Using [CLI](https://www.vaultproject.io/docs/commands) here, but can also 
[HTTP API](https://www.vaultproject.io/api/index.html)

Using **inmem** backend in `-dev` but can use [Consul](https://www.consul.io/)

### Writing a Secret

:ship: write a secret
```bash
vault kv put secret/hello foo=world
```

:ship: You can even write multiple pieces of data, if you want:
```bash
vault kv put secret/hello foo=world excited=yes
```

[command documentation](https://www.vaultproject.io/docs/commands/index.html)

:warning: The documentation uses the `key=value` based entry throughout, but it
is more secure to use files if possible. Sending data via the CLI is often
logged in shell history. For real secrets, please use files. See the link above
about reading in from `STDIN`.

### Getting a Secret

:ship: read secret
```bash
vault kv get secret/hello
```

:ship: To print only the value of a given field: 
```bash
vault kv get -field=excited secret/hello
```

:ship: Optional json output
```bash
vault kv get -format=json secret/hello | jq -r .data.data.excited
```

### Deleting a Secret

:ship: delete a secret
```bash
vault kv delete secret/hello
```

## Secrets Engines | https://learn.hashicorp.com/vault/getting-started/secrets-engines

:flashlight: By default, Vault enables [Key/Value version2 secrets
engine](https://www.vaultproject.io/docs/secrets/kv/kv-v2/) at the path
`secret/` when running in `dev` mode. 

### Enable a Secrets Engine

:ship: enable a new `kv` [Secrets Engine](https://www.vaultproject.io/docs/secrets)
```bash
vault secrets enable -path=kv kv
```

:flashlight:  Each path is completely isolated and cannot talk to other paths.
For example, a `kv` secrets engine enabled at `foo` has no ability to
communicate with a `kv` secrets engine enabled at `bar`.
:
:ship: If `-path` is not specified it defaults. This is same as above.
```bash
vault secrets enable kv
```

:ship: To verify our success and get more information about the secrets engine 
```bash
vault secrets list
```

    Path          Type         Accessor              Description
    ----          ----         --------              -----------
    cubbyhole/    cubbyhole    cubbyhole_78189996    per-token private secret storage
    identity/     identity     identity_ac07951e     identity store
    kv/           kv           kv_15087625           n/a
    secret/       kv           kv_4b990c45           key/value secret storage
    sys/          system       system_adff0898       system endpoints used for control, policy and debugging

:flashlight: The `sys/` path corresponds to the system backend. These paths
interact with Vault's core system and are not required for beginners.

:ship: To create secrets, use the `kv put` command.
```bash
vault kv put kv/hello target=world
```

    Success! Data written to: kv/hello
:ship: To read the secrets stored in the `kv/hello` path
```bash
vault kv get kv/hello
```

    ===== Data =====
    Key       Value
    ---       -----
    target    world

:ship: Create secrets at the `kv/my-secret` path.
```bash
vault kv put kv/my-secret value="s3c(eT"
```

:ship: Read the secrets at `kv/my-secret`.
```bash
vault kv get kv/my-secret
```

:ship: Delete the secrets at `kv/my-secret`.
```bash
vault kv delete kv/my-secret
```
:ship: List existing keys at the `kv` path.
```bash
vault kv list kv/
```

    Keys
    ----
    hello

### Disable a Secrets Engine

:flashlight: When a secrets engine is disabled, all secrets are revoked and the
corresponding Vault data and configuration is removed.

:ship: Disable engine
```bash
vault secrets disable kv/
```

:exclamation: this command takes a `PATH` to the secrets engine as an argument,
not the `TYPE` of the secrets engine.

### What is a Secrets Engine?

Vault behaves similarly to a 
[virtual filesystem](https://en.wikipedia.org/wiki/Virtual_file_system)

This abstraction is incredibly powerful. It enables Vault to interface directly
with physical systems, databases, HSMs, etc. But in addition to these physical
systems, Vault can interact with more unique environments like AWS IAM, dynamic
SQL user creation, etc. all while using the same read/write interface.

## Dynamic Secrets | https://learn.hashicorp.com/vault/getting-started/dynamic-secrets 

Unlike the `kv` secrets where you had to put data into the store yourself,
dynamic secrets are generated when they are accessed. 

Dynamic secrets do not exist until they are read, so there is no risk of
someone stealing them or another client using the same secrets. 

Because Vault has built-in revocation mechanisms, dynamic secrets can be
revoked immediately after use, minimizing the amount of time the secret
existed.

:warning: Before starting this page, please register for an 
[AWS account](https://aws.amazon.com/)

### Enable the AWS secrets engine

:ship: Enable aws secrets engine at `aws/`
```bash
vault secrets enable -path=aws aws
```

:flashlight: In this case, the AWS secrets engine generates dynamic, on-demand
AWS access credentials.

### Configure the AWS secrets engine

After enabling the AWS secrets engine, you must configure it to authenticate and communicate with AWS. This requires privileged account credentials. If you are unfamiliar with AWS, use your root account keys.

:warning: Do not use your root account keys in production. This is a getting started guide and is not a best practices guide for production installations.

:exclamation: I created an 
[administrator credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html).
and copied keys into `~/.aws/credentials`

:ship: configure AWS secrets engine with 
```bash
vault write aws/config/root \
access_key=aws_access_key_id \
secret_key=aws_secret_access_key \
region=us-east-1
```

### Create a role

The next step is to configure a _role_. Vault knows how to create an IAM user
via the AWS API, but it does not know what permissions, groups, and policies
you want to attach to that user. This is where roles come in - a role in Vault
is a human-friendly identifier to an action.

For example, here is an 
[IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_access-management.html)
that enables all actions on EC2, but not IAM or other AWS services.

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

We need to map this policy document to a 
[named role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)

:ship: Map the above IAM policy to a new role `my-role` for user `iam_user`
```bash
vault write aws/roles/my-role \
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

### Generate the secret

:ship: ask Vault to generate an access key pair for the new role
```bash
vault read aws/creds/my-role
```

Success! The access and secret key can now be used to perform any EC2
operations within AWS. 

:exclamation: If you were to run the command a second time, you would get a new
access key pair. Each time you read from `aws/creds/:name`, Vault will connect
to AWS and generate a new IAM user and key pair.

:warning: Copy the full path of this `lease_id` value found in the output. This
value is used for renewal, revocation, and inspection.

### Revoke the secret

:exclamation: Vault will automatically revoke this credential after 768 hours
(see `lease_duration` in the output)
:ship: To revoke the secret, use `vault revoke` with the lease ID that was
outputted from `vault read` when you ran it
```bash
vault lease revoke LEASE_ID
```

## Built-in Help | Vault - HashiCorp Learn

You've now worked with `vault write` and `vault read` for multiple paths: the `kv` secrets engine with `kv/` and dynamic AWS credentials with the AWS secrets engine provider at `aws/`. In both cases, the structure and usage of each secrets engines differed, for example the AWS backend has special paths like `aws/config`.

Instead of having to memorize or reference documentation constantly to determine what paths to use, Vault has a built-in help system. This help system can be accessed via the API or the command-line and generates human-readable help for any path.

## Secrets Engines Overview | https://learn.hashicorp.com/vault/getting-started/help

This section assumes you have the AWS secrets engine enabled at `aws/`. If you
do not, enable it before continuing:

:ship:
```bash
vault secrets enable -path=aws aws
```

With the secrets engine enabled, learn about it with the `vault path-help` command:

:ship:
```bash
vault path-help aws
```

    ### DESCRIPTION

    The AWS backend dynamically generates AWS access keys for a set of
    IAM policies. The AWS access keys have a configurable lease set and
    are automatically revoked at the end of the lease.

    After mounting this backend, credentials to generate IAM keys must
    be configured with the "root" path and policies must be written using
    the "roles/" endpoints before any access keys can be generated.

    ### PATHS

    The following paths are supported by this backend. To view help for
    any of the paths below, use the help command with any route matching
    the path pattern. Note that depending on the policy of your auth token,
    you may or may not be able to access certain paths.

        ^config/lease$
            Configure the default lease information for generated credentials.

        ^config/root$
            Configure the root credentials that are used to manage IAM.

        ^creds/(?P<name>\w+)$
            Generate an access key pair for a specific role.

        ^roles/(?P<name>\w+)$
            Read and write IAM policies that access keys can be made for.

The `vault path-help` command takes a path. By specifying a root path, it will give us the overview of that secrets engine. Notice how the help not only contains a description, but also the exact regular expressions used to match routes for this backend along with a brief description of what the route is for.

### Path Help

After seeing the overview, we can continue to dive deeper by getting help for an individual path. For this, just use `vault path-help` with a path that would match the regular expression for that path. Note that the path doesn't need to actually _work_. For example, we'll get the help below for accessing `aws/creds/my-non-existent-role`, even though we never created the role:

:ship:
```bash
vault path-help aws/creds/my-non-existent-role
```

    Request:        creds/my-non-existent-role
    Matching Route: ^creds/(?P<name>\w(([\w-.]+)?\w)?)$

    Generate an access key pair for a specific role.

    ### PARAMETERS

        name (string)
            Name of the role

    ### DESCRIPTION

    This path will generate a new, never before used key pair for
    accessing AWS. The IAM policy used to back this key pair will be
    the "name" parameter. For example, if this backend is mounted at "aws",
    then "aws/creds/deploy" would generate access keys for the "deploy" role.

    The access keys will have a lease associated with them. The access keys
    can be revoked by using the lease ID.

Within a path, we are given the parameters that this path requires. Some parameters come from the route itself. In this case, the `name` parameter is a named capture from the route regular expression. There is also a description of what that path does.

Go ahead and explore more paths! Enable other secrets engines, traverse their help systems, and learn about what they do.

### Next

The help system may not be the most exciting feature of Vault, but it is indispensable in day-to-day usage. The help system lets you learn about how to use any backend within Vault without leaving the command line.> Users can authenticate to Vault using multiple methods.

## Authentication | https://learn.hashicorp.com/vault/getting-started/authentication

:warning: When starting the Vault server in `dev` mode, it automatically logs
you in as the root user with admin permissions. In a non-dev setup, you would
have had to authenticate first.

Authentication is the mechanism of assigning an identity to a Vault user. 

The access control and permissions associated with an identity are
authorization https://learn.hashicorp.com/vault/getting-started/policies)

Vault has pluggable auth methods https://www.vaultproject.io/docs/auth 

### Tokens

Token authentication is enabled by default in Vault and cannot be disabled.

When you start a dev server with `vault server -dev`, it prints your _root
token_. The root token is the initial access token to configure Vault. It has
root privileges, so it can perform any operation within Vault.
:ship: Create more tokens using the `vault token create` command.
```bash
vault token create
```

:warning: By default, this will create a child token of your current token that
inherits all the same policies. 

:flashlight: When that parent token is revoked, children can also be revoked
all in one operation. 

:ship: Authenticate with a token
```bash
vault login VAULT_TOKEN
```

:ship: After a token is created, you can revoke it.
```bash
vault token revoke VAULT_TOKEN
```

:ship: Log back in with root token.
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

#### Recommended Patterns

In practice, operators should not use the `token create` command to generate
Vault tokens for users or machines. Instead, those users or machines should
authenticate to Vault using any of Vault's configured auth methods such as
GitHub, LDAP, AppRole, etc. For legacy applications which cannot generate their
own token, operators may need to create a token in advance. Auth methods are
discussed in more detail in the next section.

### Auth Methods

We can use GitHub Auth https://www.vaultproject.io/docs/auth/github

:ship: Authenticate via GitHub.
```bash
vault auth enable -path=github github
```

:exclamation: Just like secrets engines, auth methods default to their TYPE as
the PATH, so the following commands are equivalent.

```bash
vault auth enable github
```

:ship: Another example with custom path
```bash
vault auth enable -path=my-github github
```

:ship: Set the user's GitHub Org
```bash
vault write auth/github/config organization=my-github-org
```

:ship: Map policies to a team within the organization.
```bash
vault write auth/github/map/teams/my-team value=default,my-policy
```

This command tells Vault to map any users who are members of the team "my-team" (in the hashicorp organization) to the policies "default" and "my-policy".

:exclamation: These policies do not have to exist in the system yet - Vault will just produce a warning when you login.

:ship: As a user, you may want to find which auth methods are enabled and available.
```bash
vault auth list
```

:ship: To get help on Github Auth
```bash
vault auth help github
```

:ship: Request help information for the AWS auth method.
```bash
vault auth help aws
```

:ship: Request help information for the userpass auth method.
```bash
vault auth help userpass
```
:ship: Request help information for tokens.
```bash
vault auth help token
```

:ship: Create a GitHub personal access token
https://help.github.com/articles/creating-an-access-token-for-command-line-use/

:ship: Login with GitHub Auth
```bash
vault login -method=github
```

:ship: This new user we just created does not have many permissions in Vault.
To continue, re-authenticate with the root token.
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

:ship: You can revoke any logins from an auth method using `vault token revoke` with the `-mode` argument. For example:
```bash
vault token revoke -mode path auth/github
```

:ship: Alternatively, if you want to completely disable the GitHub auth method, execute the following command.
```bash
vault auth disable github
```

## Policies | https://learn.hashicorp.com/vault/getting-started/policies

Policies in Vault control what a user can access ie **authorization**. 

The `root` and `default` policies are required policies and cannot be deleted. 
- The `default` policy provides a common set of permissions and is included on
  all tokens by default. 
- The `root` policy gives a token super admin permissions, similar to a root
  user on a linux machine.

### Policy Format

Policies are authored in HCL https://github.com/hashicorp/hcl, but are JSON
compatible. 

:flashlight: Save to `my-policy.hcl` this example policy
```bash
    path "secret/data/*" {
      capabilities = ["create", "update"]
    }

    path "secret/data/foo" {
      capabilities = ["read"]
    }
```

With this policy, a user could write any secret to `secret/data/`, except to
`secret/data/foo`, where only read access is allowed. 

:flashlight: Policies default to deny, so any access to an unspecified path is
not allowed.

:ship: Check the policy for syntax errors
```bash
vault policy fmt my-policy.hcl
```

### Writing the Policy

:ship: Write the policy
```bash
vault policy write my-policy my-policy.hcl
```

:ship: An example of inline policy
```bash
vault policy write my-policy -<<EOF
    # Dev servers have version 2 of KV secrets engine mounted by default, so will
    # need these paths to grant permissions:
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

:ship: To view the contents of a specific policy
```bash
vault policy read my-policy
```

### Testing the Policy

:warning: First, check to verify that KV v2 secrets engine has not been enabled
at `secret/`.

:ship:
```bash
vault secrets list
```

:ship: If `secret/` is **not** listed, enable it before proceeding.
```bash
vault secrets enable -path=secret/ kv-v2
```

:ship: To use the policy, create a token and assign it to that policy.  
```bash
vault token create -policy=my-policy
```

:ship: Copy the generated token value and authenticate with Vault.  
```bash
vault login GENERATED_TOKEN
```

:exclamation: When you access the KV v2 secrets engine
https://www.vaultproject.io/docs/secrets/kv/kv-v2/ using the `vault kv` CLI
commands, you can omit `/data` in the secret path.

:ship: Verify that you can write any data to `secret/data/`.
```bash
vault kv put secret/creds password="my-long-password"
```

:flashlight: Since `my-policy` only permits read from the `secret/data/foo`
path, any attempt to write fails with "permission denied" error.

:ship: This will throw a `Error writing data to secret/data/foo`
```bash
vault kv put secret/foo robot=beepboop
```

:exclamation: You also do not have access to `sys` according to the policy, so
commands like `vault policy list` or `vault secrets list` will not work.

:ship: Re-authenticate with the initial root token to continue.  
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

### Mapping Policies to Auth Methods

:warning: Use the `vault path-help` system with your auth method to determine
how the mapping is done since it is specific to each auth method. 

:ship: For example, with GitHub, it is done per team using the
`map/teams/<team>` path.  
```bash
vault write auth/github/map/teams/default value=my-policy
```

## Deploy Vault https://learn.hashicorp.com/vault/getting-started/deploy

:flashlight: Up to this point, we have been working with the "dev" server,
which automatically authenticated us, setup in-memory storage, etc. 

Learn how to `configure` Vault, `start` Vault, the `seal/unseal` process, and
`scaling` Vault.

### Configuring Vault

Vault is configured using HCL https://github.com/hashicorp/hcl files. 

:ship: Save in `config.hcl` 
- using **Consul** as the storage backend and 
- setting up a listener for **API requests**
```hcl
storage "consul" {            
  address = "127.0.0.1:8500"
  path    = "vault/"
}

listener "tcp" {
 address     = "127.0.0.1:8200"
 tls_disable = 1
}
```

:ship: Consul Getting Started Guide
https://www.consul.io/intro/getting-started/install.html up to the point where
you have installed Consul and started it with this command:
```bash
consul agent -dev
```

### Starting the Server

:ship: Start Vault using the config specified 
```bash
vault server -config=config.hcl
```

:warning: If you get a warning message about mlock not being supported, that is
okay. However, for maximum security you should run Vault on a system that
supports mlock.

:exclamation: You'll notice that you can't execute any commands. We don't have
any auth information! When you first setup a Vault server, you have to start by
_initializing_ it.

### Initializing the Vault

**Initialization** is the process configuring the Vault. 

:flashlight: When running in HA mode, this happens once _per cluster_, not _per
server_.

:exclamation: During initialization, 
- the **encryption keys** are generated, 
- **unseal keys** are created, 
- and the **initial root token** is setup. 

:ship: To initialize Vault use `vault operator init`. This is an
_unauthenticated_ request, but it only works on brand new Vaults with no data 
```bash
vault operator init
```

    Unseal Key 1: 4jYbl2CBIv6SpkKj6Hos9iD32k5RfGkLzlosrrq/JgOm
    Unseal Key 2: B05G1DRtfYckFV5BbdBvXq0wkK5HFqB9g2jcDmNfTQiS
    Unseal Key 3: Arig0N9rN9ezkTRo7qTB7gsIZDaonOcc53EHo83F5chA
    Unseal Key 4: 0cZE0C/gEk3YHaKjIWxhyyfs8REhqkRW/CSXTnmTilv+
    Unseal Key 5: fYhZOseRgzxmJCmIqUdxEm9C3jB5Q27AowER9w4FC2Ck

    Initial Root Token: s.KkNJYWF5g0pomcCLEmDdOVCW

    Vault initialized with 5 key shares and a key threshold of 3. Please securely
    distribute the key shares printed above. When the Vault is re-sealed,
    restarted, or stopped, you must supply at least 3 of these keys to unseal it
    before it can start servicing requests.

    Vault does not store the generated master key. Without at least 3 key to
    reconstruct the master key, Vault will remain permanently sealed!

    It is possible to generate new unseal keys, provided you have a quorum of
    existing unseal keys shares. See "vault operator rekey" for more information.

:warning: This is the **only time ever** that all of this data is known by
Vault, and also the only time that the unseal keys should ever be so close
together.

:ship: For the purpose of this getting started guide, save all of these keys
somewhere, and continue. 

:flashlight: In a real deployment scenario, you would never save these keys
together. Instead, you would likely use Vault's PGP and Keybase.io support to
encrypt each of these keys with the users' PGP keys. This prevents one single
person from having all the unseal keys. 

See the documentation on using PGP, GPG, and Keybase
https://www.vaultproject.io/docs/concepts/pgp-gpg-keybase.html

### Seal/Unseal

Every initialized Vault server starts in the _sealed_ state. 

The process of teaching Vault how to decrypt the data is known as _unsealing_
the Vault.

Unsealing has to happen every time Vault starts. 

To unseal the Vault, you must have the _threshold_ number of unseal keys. In
the output above, notice that the "key threshold" is 3. 

:flashlight: Vault does not store any of the unseal key shards. Vault uses an
algorithm known as Shamir's Secret Sharing
https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing to split the master key
into shards. 

:ship: Begin unsealing the Vault: 
```bash
vault operator unseal
```

    Unseal Key (will be hidden):
    Key                Value
    ---                -----
    Seal Type          shamir
    Initialized        true
    Sealed             true
    Total Shares       5
    Threshold          3
    Unseal Progress    1/3                            # Note the unseal progress
    Unseal Nonce       d3d06528-aafd-c63d-a93c-e63ddb34b2a9
    Version            1.3.4
    HA Enabled         true

:flashlight: Also notice that the unseal process is stateful. You can go to another
computer, use `vault operator unseal`, and as long as it's pointing to the same
server, that other computer can continue the unseal process. 

:ship: Continue with `vault operator unseal` to complete unsealing the Vault.
To unseal the vault you must use three _different_ unseal keys, the same key
repeated will not work.  
```bash
vault operator unseal
```

:exclamation: When the value for `Sealed` changes to `false`, the Vault is
unsealed.

:ship: Finally, authenticate as the initial root token (it was included in the
output with the unseal keys)
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

:exclamation: As a root user, you can reseal the Vault with `vault operator
seal`. A single operator is allowed to do this. This lets a single operator
lock down the Vault in an emergency without consulting other operators.

When the Vault is sealed again, it clears all of its state (including the
encryption key) from memory. The Vault is secure and locked down from access.

## Using the HTTP APIs with Authentication | https://learn.hashicorp.com/vault/getting-started/apis

:flashlight: Vault CLI is subset of API

:ship: If in `dev mode`, validate the initialization status 
```bash
curl http://127.0.0.1:8200/v1/sys/init
```

### Accessing Secrets via the REST APIs

Machines that need access to information stored in Vault will most likely access Vault via its REST API. For example, if a machine were using 

[AppRole](https://www.vaultproject.io/docs/auth/approle.html)

 for authentication, the application would first authenticate to Vault which would return a Vault API token. The application would use that token for future communication with Vault.

:ship: For this guide, disable TLS and save to `config.hcl`
```bash
backend "file" {
  path = "vault"
}

listener "tcp" {
  tls_disable = 1
}
```

:warning: TLS is disabled here only for example purposes; it should never be disabled in production.

:ship: Start vault with new config
```bash
vault server -config=config.hcl
```

:ship: Start vault
```bash
curl \
       --request POST \
       --data '{"secret_shares": 1, "secret_threshold": 1}' \
       http://127.0.0.1:8200/v1/sys/init | jq
```

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

:ship: Store root token. 
```bash
export VAULT_TOKEN="s.Ga5jyNq6kNfRMVQk2LY1j9iu"
```

:warning: Do not store the root token in production

:ship: Using the unseal key (not the root token) from above, you can unseal the
Vault via the HTTP API:
```bash
curl \
       --request POST \
       --data '{"key": "/ye2PeRrd/qruh9Ppu9EyUjk1vLqIflg1qqw6w9OE5E="}' \
       http://127.0.0.1:8200/v1/sys/unseal | jq
```

:ship: Enable AppRole auth for now
https://www.vaultproject.io/docs/auth/approle.html
```bash
vault auth enable <auth_method_type>
```

:ship: To see the cURL equivalent of the CLI command to enable AppRole auth
method, use the `-output-curl-string` flag.

```bash
vault auth enable -output-curl-string approle
```

:ship: Enable the AppRole auth method by invoking the Vault API.
```bash
curl \
       --header "X-Vault-Token: $VAULT_TOKEN" \
       --request POST \
       --data '{"type": "approle"}' \
       http://127.0.0.1:8200/v1/sys/auth/approle
```

:ship: Create an ACL policies https://www.vaultproject.io/docs/concepts/policies.html
```bash
curl \
       --header "X-Vault-Token: $VAULT_TOKEN" \
       --request PUT \
       --data '{"policy":"# Dev servers have version 2 of KV secrets engine mounted by default, so will\n# need these paths to grant permissions:\npath \"secret/data/*\" {\n  capabilities = [\"create\", \"update\"]\n}\n\npath \"secret/data/foo\" {\n  capabilities = [\"read\"]\n}\n"}' \
       http://127.0.0.1:8200/v1/sys/policies/acl/my-policy
```

:ship: Since `my-policy` expects `secret/data` path to exist, enable KV v2 secrets engine at `secret/` using API.  
```bash
curl \
       --header "X-Vault-Token: $VAULT_TOKEN" \
       --request POST \
       --data '{ "type":"kv-v2" }' \
       http://127.0.0.1:8200/v1/sys/mounts/secret
```

:ship: The following command specifies that the tokens issued under the AppRole `my-role` should be associated with `my-policy`.  
```bash
curl \
       --header "X-Vault-Token: $VAULT_TOKEN" \
       --request POST \
       --data '{"policies": ["my-policy"]}' \
       http://127.0.0.1:8200/v1/auth/approle/role/my-role
```

:warning: The AppRole auth method expects a RoleID and a SecretID as its input.
The RoleID is similar to a username and the SecretID can be thought as the
RoleID's password.

:ship: fetch the RoleID of the role named `my-role` and scan output for `role_id`.
```bash
curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
         http://127.0.0.1:8200/v1/auth/approle/role/my-role/role-id | jq -r ".data"
```

:ship: create a new SecretID under the `my-role` and scan for `secret_id`.  
```bash
curl \
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        http://127.0.0.1:8200/v1/auth/approle/role/my-role/secret-id | jq -r ".data"
```

These two credentials can be supplied to the login endpoint to fetch a new Vault token.

:ship:
```bash
curl --request POST \
       --data '{"role_id": "ROLE_ID", "secret_id": "SECRET_ID"}' \
       http://127.0.0.1:8200/v1/auth/approle/login | jq -r ".auth"
```

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

:exclamation: The returned `client_token` can be used to authenticate with
Vault. 

:ship: The newly acquired token can be exported as the `VAULT_TOKEN`
environment variable value and used to authenticate subsequent Vault requests.
```bash
export VAULT_TOKEN="CLIENT_TOKEN"
```

:ship: Create a version 1 of secret named `creds` with a key `password` and its
value set to `my-long-password`.
```bash
curl \
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
```

:ship: You can stop the server and unset `VAULT_TOKEN` variable.
```bash
unset VAULT_TOKEN
```

HTTP APIs https://www.vaultproject.io/api/index.html

## Web UI | https://learn.hashicorp.com/vault/getting-started/ui

### Dev servers

When you start the Vault server in dev mode, Vault UI is automatically enabled
and ready to use.

:ship: Start dev server
```bash
vault server -dev
```

:ship: Open a web browser and enter `http://127.0.0.1:8200/ui` to launch the UI.

### Non-Dev servers

:warning: The Vault UI is not activated by default. 

:ship: To activate the UI, set the `ui` configuration option in the Vault
server configuration.
```hcl
ui = true

listener "tcp" {
  address = "10.0.1.35:8200" 
}

storage "consul" {
  
}
```

:warning: The UI runs on the same port as the Vault listener. As such, you must
configure at least one `listener` stanza in order to access the UI.

https://10.0.1.35:8200/ui