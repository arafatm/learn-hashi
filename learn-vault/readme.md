# Learn Vault

https://learn.hashicorp.com/vault#getting-started

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

## Dynamic Secrets | Vault - HashiCorp Learn

Now that you've experimented with the `kv` secrets engine, it is time to explore another feature of Vault: _dynamic secrets_.

Unlike the `kv` secrets where you had to put data into the store yourself, dynamic secrets are generated when they are accessed. Dynamic secrets do not exist until they are read, so there is no risk of someone stealing them or another client using the same secrets. Because Vault has built-in revocation mechanisms, dynamic secrets can be revoked immediately after use, minimizing the amount of time the secret existed.

:flashlight: Before starting this page, please register for an 

[AWS account](https://aws.amazon.com/)

. We won't be using any features that cost money, so you shouldn't be charged for anything. However, we are not responsible for any charges you may incur.

### Enable the AWS secrets engine

Unlike the `kv` secrets engine which is enabled by default, the AWS secrets engine must be enabled before use. This step is usually done via a configuration management system.

:ship:
```bash
vault secrets enable -path=aws aws
```

    Success! Enabled the aws secrets engine at: aws/

The AWS secrets engine is now enabled at `aws/`. As we covered in the previous sections, different secrets engines allow for different behavior. In this case, the AWS secrets engine generates dynamic, on-demand AWS access credentials.

### Configure the AWS secrets engine

After enabling the AWS secrets engine, you must configure it to authenticate and communicate with AWS. This requires privileged account credentials. If you are unfamiliar with AWS, use your root account keys.

:warning: Do not use your root account keys in production. This is a getting started guide and is not a best practices guide for production installations.

:ship:
```bash
vault write aws/config/root \
```
        access_key=AKIAI4SGLQPBX6CSENIQ \
        secret_key=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 \
        region=us-east-1

    Success! Data written to: aws/config/root

These credentials are now stored in this AWS secrets engine. The engine will use these credentials when communicating with AWS in future requests.

### Create a role

The next step is to configure a _role_. Vault knows how to create an IAM user via the AWS API, but it does not know what permissions, groups, and policies you want to attach to that user. This is where roles come in - a role in Vault is a human-friendly identifier to an action.

For example, here is an IAM policy that enables all actions on EC2, but not IAM or other AWS services.

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

:exclamation: If you are not familiar with AWS' IAM policies, that is okay - just use this one for now.

We need to map this policy document to a named role. To do that, write to `aws/roles/:name` where `:name` is your unique name that describes the role (such as `aws/roles/my-role`):

:ship:
```bash
vault write aws/roles/my-role \
```
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
    Success! Data written to: aws/roles/my-role

We just told Vault:

> When I ask for a credential for "my-role", create it and attach the IAM policy `{ "Version": "2012..." }`.

### Generate the secret

Now that the AWS secrets engine is enabled and configured with a role, we can ask Vault to generate an access key pair for that role by reading from `aws/creds/:name`, where `:name` corresponds to the name of an existing role:

:ship:
```bash
vault read aws/creds/my-role
```
    Key                Value
    ---                -----
    lease_id           aws/creds/my-role/0bce0782-32aa-25ec-f61d-c026ff22106e
    lease_duration     768h
    lease_renewable    true
    access_key         AKIAJELUDIANQGRXCTZQ
    secret_key         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
    security_token     <nil>

Success! The access and secret key can now be used to perform any EC2 operations within AWS. Notice that these keys are new, they are not the keys you entered earlier. If you were to run the command a second time, you would get a new access key pair. Each time you read from `aws/creds/:name`, Vault will connect to AWS and generate a new IAM user and key pair.

Copy the full path of this `lease_id` value found in the output. This value is used for renewal, revocation, and inspection.

### Revoke the secret

Vault will automatically revoke this credential after 768 hours (see `lease_duration` in the output), but perhaps we want to revoke it early. Once the secret is revoked, the access keys are no longer valid.

To revoke the secret, use `vault revoke` with the lease ID that was outputted from `vault read` when you ran it:

:ship:
```bash
vault lease revoke aws/creds/my-role/0bce0782-32aa-25ec-f61d-c026ff22106
```
    Success! Revoked lease: aws/creds/my-role/0bce0782-32aa-25ec-f61d-c026ff22106e

Done! If you login to your AWS account, you will see that no IAM users exist. If you try to use the access keys that were generated, you will find that they no longer work.

### Next

On this page we experienced our first dynamic secret, and we also saw the revocation system in action. Dynamic secrets are incredibly powerful. As time goes on, we expect that more systems will support some sort of API to create access credentials, and Vault will be ready to get the most value out of this practice.

Before going further, we're going to take a quick detour to learn about the built-in help system.> Vault has a built-in help system to learn about the available paths in Vault and how to use them.

## Built-in Help | Vault - HashiCorp Learn

You've now worked with `vault write` and `vault read` for multiple paths: the `kv` secrets engine with `kv/` and dynamic AWS credentials with the AWS secrets engine provider at `aws/`. In both cases, the structure and usage of each secrets engines differed, for example the AWS backend has special paths like `aws/config`.

Instead of having to memorize or reference documentation constantly to determine what paths to use, Vault has a built-in help system. This help system can be accessed via the API or the command-line and generates human-readable help for any path.

### Secrets Engines Overview

This section assumes you have the AWS secrets engine enabled at `aws/`. If you do not, enable it before continuing:

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

## Authentication | Vault - HashiCorp Learn

Now that we know how to use the basics of Vault, it is important to understand how to authenticate to Vault itself. Up to this point, we have not logged in to Vault. When starting the Vault server in `dev` mode, it automatically logs you in as the root user with admin permissions. In a non-dev setup, you would have had to authenticate first.

On this page, we'll talk specifically about authentication. On the next page, we talk about 

[authorization](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies)

. Authentication is the mechanism of assigning an identity to a Vault user. The access control and permissions associated with an identity are authorization, and will not be covered on this page.

Vault has pluggable auth methods, making it easy to authenticate with Vault using whatever form works best for your organization. On this page we will use the token auth method and the GitHub auth method.

### Background

Authentication is the process by which user or machine-supplied information is verified and converted into a Vault token with matching policies attached. The easiest way to think about Vault's authentication is to compare it to a website.

When a user authenticates to a website, they enter their username, password, and maybe 2FA code. That information is verified against external sources (a database most likely), and the website responds with a success or failure. On success, the website also returns a signed cookie that contains a session id which uniquely identifies that user for this session. That cookie and session id are automatically carried by the browser to future requests so the user is authenticated. Can you imagine how terrible it would be to require a user to enter their login credentials on each page?

Vault behaves very similarly, but it is much more flexible and pluggable than a standard website. Vault supports many different authentication mechanisms, but they all funnel into a single "session token", which we call the "Vault token".

Authentication is simply the process by which a user or machine gets a Vault token.

### Tokens

Token authentication is enabled by default in Vault and cannot be disabled. When you start a dev server with `vault server -dev`, it prints your _root token_. The root token is the initial access token to configure Vault. It has root privileges, so it can perform any operation within Vault.

You can create more tokens using the `vault token create` command.

:ship:
```bash
vault token create
```

    Key                  Value
    ---                  -----
    token                s.iyNUhq8Ov4hIAx6snw5mB2nL
    token_accessor       maMfHsZfwLB6fi18Zenj3qh6
    token_duration       ∞
    token_renewable      false
    token_policies       ["root"]
    identity_policies    []
    policies             ["root"]

By default, this will create a child token of your current token that inherits all the same policies. The "child" concept here is important: tokens always have a parent, and when that parent token is revoked, children can also be revoked all in one operation. This makes it easy when removing access for a user, to remove access for all sub-tokens that user created as well.

To authenticate with a token, execute the `vault login` command.

:ship:
```bash
vault login s.iyNUhq8Ov4hIAx6snw5mB2nL
```

    Success! You are now authenticated. The token information displayed below
    is already stored in the token helper. You do NOT need to run "vault login"
    again. Future Vault requests will automatically use this token.

    Key                  Value
    ---                  -----
    token                s.iyNUhq8Ov4hIAx6snw5mB2nL
    token_accessor       maMfHsZfwLB6fi18Zenj3qh6
    token_duration       ∞
    token_renewable      false
    token_policies       ["root"]
    identity_policies    []
    policies             ["root"]

This authenticates with Vault. It will verify your token and let you know what access policies the token is associated with.

After a token is created, you can revoke it.

:ship:
```bash
vault token revoke s.V6T0DxxIg5FbBSre61y1WLgm
```

    Success! Revoked token (if it existed)

In a previous section, we used the `vault lease revoke` command. This command is only used for revoking _leases_. For revoking _tokens_, use `vault token revoke`.

Log back in with root token.

:ship:
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

#### 

[»](#recommended-patterns)

Recommended Patterns


In practice, operators should not use the `token create` command to generate Vault tokens for users or machines. Instead, those users or machines should authenticate to Vault using any of Vault's configured auth methods such as GitHub, LDAP, AppRole, etc. For legacy applications which cannot generate their own token, operators may need to create a token in advance. Auth methods are discussed in more detail in the next section.

### Auth Methods

Vault supports many auth methods, but they must be enabled before use. Auth methods give you flexibility. Enabling and configuring auth methods are typically performed by a Vault operator or security team. As an example of a human-focused auth method, let's authenticate via GitHub.

First, enable the GitHub auth method.

:ship:
```bash
vault auth enable -path=github github
```

    Success! Enabled github auth method at: github/

:exclamation: Just like secrets engines, auth methods default to their TYPE as the PATH, so the following commands are equivalent.

:ship:
```bash
vault auth enable -path=github github
```

:ship:
```bash
vault auth enable github
```

Unlike secrets engines which are enabled at the root router, auth methods are always prefixed with `auth/` in their path. So the GitHub auth method we just enabled is accessible at `auth/github`. As another example:

:ship:
```bash
vault auth enable -path=my-github github
```

    Success! Enabled github auth method at: my-github/

This would make the GitHub auth method accessible at `auth/my-github`. You can use `vault path-help` to learn more about the paths.

Each auth method has different configuration options, so please see the documentation for the full details.

With the GitHub auth method enabled, we next tell it which organization users must be a part of.

:ship:
```bash
vault write auth/github/config organization=hashicorp
```

    Success! Data written to: auth/github/config

The above command configured Vault to pull authentication data from the "hashicorp" organization on GitHub.

Map policies to a team within the organization.

:ship:
```bash
vault write auth/github/map/teams/my-team value=default,my-policy
```

    Success! Data written to: auth/github/map/teams/my-team

This command tells Vault to map any users who are members of the team "my-team" (in the hashicorp organization) to the policies "default" and "my-policy".

:exclamation: These policies do not have to exist in the system yet - Vault will just produce a warning when you login.

As a user, you may want to find which auth methods are enabled and available.

:ship:
```bash
vault auth list
```

    Path       Type      Description
    ----       ----      -----------
    github/    github    n/a
    token/     token     token based credentials

The `vault auth list` command will list all enabled auth methods. To learn more about how to authenticate to a particular auth method via the CLI, use the `vault auth help` command with the PATH or TYPE of an auth method.

:ship:
```bash
vault auth help github
```

    Usage: vault login -method=github [CONFIG K=V...]

      The GitHub auth method allows users to authenticate using a GitHub
      personal access token. Users can generate a personal access token from the
      settings page on their GitHub account.

      Authenticate using a GitHub token:

          $ vault login -method=github token=abcd1234

    Configuration:

      mount=<string>
          Path where the GitHub credential method is mounted. This is usually
          provided via the -path flag in the "vault login" command, but it can be
          specified here as well. If specified here, it takes precedence over the
          value for -path. The default value is "github".

      token=<string>
          GitHub personal access token to use for authentication.

Similarly, you can ask for help information about any CLI auth method, _even if it is not enabled_:

Request help information for the AWS auth method.

:ship:
```bash
vault auth help aws
```

Request help information for the userpass auth method.

:ship:
```bash
vault auth help userpass
```

Request help information for tokens.

:ship:
```bash
vault auth help token
```

As per the help output, authenticate to GitHub using the `vault login` command. Enter your 

[GitHub personal access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)

 and Vault will authenticate you.

:ship:
```bash
vault login -method=github
```

    GitHub Personal Access Token (will be hidden):
    Success! You are now authenticated. The token information displayed below
    is already stored in the token helper. You do NOT need to run "vault login"
    again. Future Vault requests will automatically use this token.

    Key                    Value
    ---                    -----
    token                  s.DNtKCjVQ1TxAzgMqtDuwjjC2
    token_accessor         e7zLJuPg2tLpav66ZSu5AyDC
    token_duration         768h
    token_renewable        true
    token_policies         [default my-policy]
    token_meta_org         hashicorp
    token_meta_username    my-user

Success! As the output indicates, Vault has already saved the resulting token in its token helper, so you do not need to run `vault login` again. However, this new user we just created does not have many permissions in Vault. To continue, re-authenticate with the root token.

:ship:
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

You can revoke any logins from an auth method using `vault token revoke` with the `-mode` argument. For example:

:ship:
```bash
vault token revoke -mode path auth/github
```

Alternatively, if you want to completely disable the GitHub auth method, execute the following command.

:ship:
```bash
vault auth disable github
```

    Success! Disabled the auth method (if it existed) at: github/

This will also revoke any logins for that auth method.

### Next

In this page you learned about how Vault authenticates users. You learned about the built-in token system as well as enabling other auth methods. At this point you know how Vault assigns an _identity_ to a user.

The auth methods Vault provides let you choose the most appropriate authentication mechanism for your organization. Next, you will learn about policies to control client authorization.> Policies in Vault control what a user can access.

## Policies | Vault - HashiCorp Learn

Policies in Vault control what a user can access. In the last section, we learned about _authentication_. This section is about _authorization_.

For authentication Vault has multiple options or methods that can be enabled and used. Vault always uses the same format for both authorization and policies. All auth methods map identities back to the core policies that are configured with Vault.

There are some built-in policies that cannot be removed. For example, the `root` and `default` policies are required policies and cannot be deleted. The `default` policy provides a common set of permissions and is included on all tokens by default. The `root` policy gives a token super admin permissions, similar to a root user on a linux machine.

### Policy Format

Policies are authored in 

[HCL](https://github.com/hashicorp/hcl)

, but are JSON compatible. Here is an example policy:



    path "secret/data/*" {
      capabilities = ["create", "update"]
    }

    path "secret/data/foo" {
      capabilities = ["read"]
    }

With this policy, a user could write any secret to `secret/data/`, except to `secret/data/foo`, where only read access is allowed. Policies default to deny, so any access to an unspecified path is not allowed.

Do not worry about getting the exact policy format correct. Vault includes a command that will format the policy automatically according to specification. It also reports on any syntax errors.

:ship:
```bash
vault policy fmt my-policy.hcl
```

The policy format uses a prefix matching system on the API path to determine access control. The most specific defined policy is used, either an exact match or the longest-prefix glob match. Since everything in Vault must be accessed via the API, this gives strict control over every aspect of Vault, including enabling secrets engines, enabling auth methods, authenticating, as well as secret access.

### Writing the Policy

:exclamation: An interactive tutorial is also available to perform the steps described in this guide. Click the **Show Tutorial** button to launch the tutorial.

To write a policy using the command line, specify the path to a policy file to upload.

:ship:
```bash
vault policy write my-policy my-policy.hcl
```

Here is an example you can copy-paste in the terminal.

:ship:
```bash
vault policy write my-policy -<<EOF
```
    # Dev servers have version 2 of KV secrets engine mounted by default, so will
    # need these paths to grant permissions:
    path "secret/data/*" {
      capabilities = ["create", "update"]
    }

    path "secret/data/foo" {
      capabilities = ["read"]
    }
    EOF

To see the list of policies, execute the following command.

:ship:
```bash
vault policy list
```

    default
    my-policy
    root

To view the contents of a policy, execute the `vault policy read` command.

:ship:
```bash
vault policy read my-policy
```

    # Dev servers have version 2 of KV secrets engine mounted by default, so will
    # need these paths to grant permissions:
    path "secret/data/*" {
      capabilities = ["create", "update"]
    }

    path "secret/data/foo" {
      capabilities = ["read"]
    }

### Testing the Policy

First, check to verify that KV v2 secrets engine has not been enabled at `secret/`.

:ship:
```bash
vault secrets list
```

    Path          Type         Accessor              Description
    ----          ----         --------              -----------
    cubbyhole/    cubbyhole    cubbyhole_b81986c7    per-token private secret storage
    identity/     identity     identity_33dc5c7d     identity store
    sys/          system       system_ad432442       system endpoints used for control, policy and debugging

If `secret/` is **not** listed, enable it before proceeding.

:ship:
```bash
vault secrets enable -path=secret/ kv-v2
```

To use the policy, create a token and assign it to that policy.

:ship:
```bash
vault token create -policy=my-policy
```

    Key                  Value
    ---                  -----
    token                s.X6gvFko7chPilgV0lpWXsdeu
    token_accessor       kwgaMv7lz09a5oULHIT3UQ6z
    token_duration       768h
    token_renewable      true
    token_policies       ["default" "my-policy"]
    identity_policies    []
    policies             ["default" "my-policy"]

Copy the generated token value and authenticate with Vault.

:ship:
```bash
vault login s.X6gvFko7chPilgV0lpWXsdeu
```

    Success! You are now authenticated. The token information displayed below
    is already stored in the token helper. You do NOT need to run "vault login"
    again. Future Vault requests will automatically use this token.

    Key                  Value
    ---                  -----
    token                s.X6gvFko7chPilgV0lpWXsdeu
    token_accessor       kwgaMv7lz09a5oULHIT3UQ6z
    token_duration       767h59m40s
    token_renewable      true
    token_policies       ["default" "my-policy"]
    identity_policies    []
    policies             ["default" "my-policy"]

Verify that you can write any data to `secret/data/`.

:exclamation: When you access the 

[KV v2 secrets engine](https://www.vaultproject.io/docs/secrets/kv/kv-v2/)

 using the `vault kv` CLI commands, you can omit `/data` in the secret path.

:ship:
```bash
vault kv put secret/creds password="my-long-password"
```

    Key              Value
    ---              -----
    created_time     2018-05-22T18:05:42.537496856Z
    deletion_time    n/a
    destroyed        false
    version          1

Since `my-policy` only permits read from the `secret/data/foo` path, any attempt to write fails with "permission denied" error.

:ship:
```bash
vault kv put secret/foo robot=beepboop
```

    Error writing data to secret/data/foo: Error making API request.

    URL: PUT http://127.0.0.1:8200/v1/secret/data/foo
    Code: 403. Errors:

    * permission denied

You also do not have access to `sys` according to the policy, so commands like `vault policy list` or `vault secrets list` will not work.

Re-authenticate with the initial root token to continue.

:ship:
```bash
vault login $VAULT_DEV_ROOT_TOKEN_ID
```

### Mapping Policies to Auth Methods

Vault itself is the single policy authority, unlike authentication where you can enable multiple auth methods. Any enabled auth method must map identities to these core policies.

Use the `vault path-help` system with your auth method to determine how the mapping is done since it is specific to each auth method. For example, with GitHub, it is done per team using the `map/teams/<team>` path.

:ship:
```bash
vault write auth/github/map/teams/default value=my-policy
```

    Success! Data written to: auth/github/map/teams/default

For GitHub, the `default` team is the default policy set that everyone is assigned to no matter what team they're on.

Other auth methods use alternate, but likely similar mechanisms for mapping policies to identity.

### Next

Policies are an important part of Vault. While using the root token is easiest to get up and running, you will want to restrict access to Vault very quickly, and the policy system is the way to do this.

The syntax and function of policies is easy to understand and work with, and because auth methods all must map to the central policy system, you only have to learn this policy system.

You will learn how to create a Vault server configuration file to customize the server settings.> Learn how to deploy Vault, including configuring, starting, initializing, and unsealing it.

## Deploy Vault | Vault - HashiCorp Learn

Up to this point, we have been working with the "dev" server, which automatically authenticated us, setup in-memory storage, etc. Now that you know the basics of Vault, it is important to learn how to deploy Vault into a real environment.

On this page, we'll cover how to configure Vault, start Vault, the seal/unseal process, and scaling Vault.

### Configuring Vault

Vault is configured using 

[HCL](https://github.com/hashicorp/hcl)

 files. The configuration file for Vault is relatively simple:

    storage "consul" {
      address = "127.0.0.1:8500"
      path    = "vault/"
    }

    listener "tcp" {
     address     = "127.0.0.1:8200"
     tls_disable = 1
    }

Within the configuration file, there are two primary configurations:

*   

[`storage`](#storage) - This is the physical backend that Vault uses for storage. Up to this point the dev server has used "inmem" (in memory), but the example above uses [Consul](https://www.consul.io/)

, a much more production-ready backend.

*   

[`listener`](#listener)

 - One or more listeners determine how Vault listens for API requests. The example above listens on localhost port 8200 without TLS. In your environment set `VAULT_ADDR=http://127.0.0.1:8200` so the Vault client will connect without TLS.

For now, copy and paste the configuration above to a file called `config.hcl`. It will configure Vault to expect an instance of Consul running locally.

Starting a local Consul instance takes only a few minutes. Just follow the 

[Consul Getting Started Guide](https://www.consul.io/intro/getting-started/install.html)

 up to the point where you have installed Consul and started it with this command:

:ship:
```bash
consul agent -dev
```

### Starting the Server

With the configuration in place, starting the server is simple, as shown below. Modify the `-config` flag to point to the proper path where you saved the configuration above.

:ship:
```bash
vault server -config=config.hcl
```

    ==> Vault server configuration:

                 Api Address: http://127.0.0.1:8200
                         Cgo: disabled
             Cluster Address: https://127.0.0.1:8201
                  Listener 1: tcp (addr: "127.0.0.1:8200", cluster address: "127.0.0.1:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
                   Log Level: info
                       Mlock: supported: false, enabled: false
               Recovery Mode: false
                     Storage: consul (HA available)
                     Version: Vault v1.3.2

    ==> Vault server started! Log data will stream in below:

:exclamation: If you get a warning message about mlock not being supported, that is okay. However, for maximum security you should run Vault on a system that supports mlock.

Vault outputs some information about its configuration, and then blocks. This process should be run using a resource manager such as systemd or upstart.

You'll notice that you can't execute any commands. We don't have any auth information! When you first setup a Vault server, you have to start by _initializing_ it.

On Linux, Vault may fail to start with the following error:

:ship:
```bash
vault server -config=config.hcl
```

    Error initializing core: Failed to lock memory: cannot allocate memory

    This usually means that the mlock syscall is not available.
    Vault uses mlock to prevent memory from being swapped to
    disk. This requires root privileges as well as a machine
    that supports mlock. Please enable mlock on your system or
    disable Vault from using it. To disable Vault from using it,
    set the `disable_mlock` configuration option in your configuration
    file.

For guidance on dealing with this issue, see the discussion of `disable_mlock` in 

[Server Configuration](https://www.vaultproject.io/docs/configuration/index.html)

.

### Initializing the Vault

Initialization is the process configuring the Vault. This only happens once when the server is started against a new backend that has never been used with Vault before. When running in HA mode, this happens once _per cluster_, not _per server_.

During initialization, the encryption keys are generated, unseal keys are created, and the initial root token is setup. To initialize Vault use `vault operator init`. This is an _unauthenticated_ request, but it only works on brand new Vaults with no data:

:ship:
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

Initialization outputs two incredibly important pieces of information: the _unseal keys_ and the _initial root token_. This is the **only time ever** that all of this data is known by Vault, and also the only time that the unseal keys should ever be so close together.

For the purpose of this getting started guide, save all of these keys somewhere, and continue. In a real deployment scenario, you would never save these keys together. Instead, you would likely use Vault's PGP and Keybase.io support to encrypt each of these keys with the users' PGP keys. This prevents one single person from having all the unseal keys. Please see the documentation on 

[using PGP, GPG, and Keybase](https://www.vaultproject.io/docs/concepts/pgp-gpg-keybase.html)

 for more information.

### Seal/Unseal

Every initialized Vault server starts in the _sealed_ state. From the configuration, Vault can access the physical storage, but it can't read any of it because it doesn't know how to decrypt it. The process of teaching Vault how to decrypt the data is known as _unsealing_ the Vault.

Unsealing has to happen every time Vault starts. It can be done via the API and via the command line. To unseal the Vault, you must have the _threshold_ number of unseal keys. In the output above, notice that the "key threshold" is 3. This means that to unseal the Vault, you need 3 of the 5 keys that were generated.

:flashlight: Vault does not store any of the unseal key shards. Vault uses an algorithm known as 

[Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)

 to split the master key into shards. Only with the threshold number of keys can it be reconstructed and your data finally accessed.

Begin unsealing the Vault:

:ship:
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
    Unseal Progress    1/3
    Unseal Nonce       d3d06528-aafd-c63d-a93c-e63ddb34b2a9
    Version            1.3.4
    HA Enabled         true

After pasting in a valid key and confirming, you'll see that the Vault is still sealed, but progress is made. Vault knows it has 1 key out of 3. Due to the nature of the algorithm, Vault doesn't know if it has the _correct_ key until the threshold is reached.

Also notice that the unseal process is stateful. You can go to another computer, use `vault operator unseal`, and as long as it's pointing to the same server, that other computer can continue the unseal process. This is incredibly important to the design of the unseal process: multiple people with multiple keys are required to unseal the Vault. The Vault can be unsealed from multiple computers and the keys should never be together. A single malicious operator does not have enough keys to be malicious.

Continue with `vault operator unseal` to complete unsealing the Vault. To unseal the vault you must use three _different_ unseal keys, the same key repeated will not work.

:ship:
```bash
vault operator unseal
```

    Unseal Key (will be hidden):
    # ...

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

Feel free to play around with entering invalid keys, keys in different orders, etc. in order to understand the unseal process.

Finally, authenticate as the initial root token (it was included in the output with the unseal keys):

:ship:
```bash
vault login s.KkNJYWF5g0pomcCLEmDdOVCW
```

    Success! You are now authenticated. The token information displayed below
    is already stored in the token helper. You do NOT need to run "vault login"
    again. Future Vault requests will automatically use this token.

    Key                  Value
    ---                  -----
    token                s.KkNJYWF5g0pomcCLEmDdOVCW
    token_accessor       9jZ4x92RbzLqwLPlb9rVAMY9
    token_duration       ∞
    token_renewable      false
    token_policies       ["root"]
    identity_policies    []
    policies             ["root"]

As a root user, you can reseal the Vault with `vault operator seal`. A single operator is allowed to do this. This lets a single operator lock down the Vault in an emergency without consulting other operators.

When the Vault is sealed again, it clears all of its state (including the encryption key) from memory. The Vault is secure and locked down from access.

### Next

You now know how to configure, initialize, and unseal/seal Vault. This is the basic knowledge necessary to deploy Vault into a real environment. Once the Vault is unsealed, you access it as you have throughout this getting started guide (which worked with an unsealed Vault).> HTTP APIs can control authentication and access to secrets.

## Using the HTTP APIs with Authentication | Vault

All of Vault's capabilities are accessible via the HTTP API in addition to the CLI. In fact, most calls from the CLI actually invoke the HTTP API. In some cases, Vault features are not available via the CLI and can only be accessed via the HTTP API.

Once you have started the Vault server, you can use Client URL (cURL) or any other http client to make API calls. For example, if you started the Vault server in 

[dev mode](https://www.vaultproject.io/docs/concepts/dev-server.html)

, you could validate the initialization status like this:

:ship:
```bash
curl http://127.0.0.1:8200/v1/sys/init
```

This will return a JSON response:

    {
      "initialized": true
    }

### Accessing Secrets via the REST APIs

Machines that need access to information stored in Vault will most likely access Vault via its REST API. For example, if a machine were using 

[AppRole](https://www.vaultproject.io/docs/auth/approle.html)

 for authentication, the application would first authenticate to Vault which would return a Vault API token. The application would use that token for future communication with Vault.

For the purpose of this guide, we will use the following configuration which disables TLS and uses a file-based backend. TLS is disabled here only for example purposes; it should never be disabled in production.

    backend "file" {
      path = "vault"
    }

    listener "tcp" {
      tls_disable = 1
    }

Save this file on disk as `config.hcl`. Stop the Vault server instance that you previously started and then start a new instance using the newly created configuration.

:ship:
```bash
vault server -config=config.hcl
```

At this point, we can use Vault's API for all our interactions. For example, we can initialize Vault like this.

:exclamation: This example uses 

[jq](https://stedolan.github.io/jq/download/)

 to process the JSON output for readability.

:ship:
```bash
curl \
```
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

:ship:
```bash
export VAULT_TOKEN="s.Ga5jyNq6kNfRMVQk2LY1j9iu"
```

Using the unseal key (not the root token) from above, you can unseal the Vault via the HTTP API:

:ship:
```bash
curl \
```
        --request POST \
        --data '{"key": "/ye2PeRrd/qruh9Ppu9EyUjk1vLqIflg1qqw6w9OE5E="}' \
        http://127.0.0.1:8200/v1/sys/unseal | jq

:exclamation: that you should replace `/ye2PeRrd/qru...` with the generated key from your output. This will return a JSON response:

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

Now any of the available auth methods can be enabled and configured. For the purposes of this guide lets enable 

[AppRole](https://www.vaultproject.io/docs/auth/approle.html)

 authentication.

The 

[Authentication](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/authentication#auth-methods)

 guide showed how to enable the GitHub auth method using Vault CLI.

:ship:
```bash
vault auth enable <auth_method_type>
```

To see the cURL equivalent of the CLI command to enable AppRole auth method, use the `-output-curl-string` flag.

:ship:
```bash
vault auth enable -output-curl-string approle
```

Enable the AppRole auth method by invoking the Vault API.

:ship:
```bash
curl \
```
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{"type": "approle"}' \
        http://127.0.0.1:8200/v1/sys/auth/approle

Notice that the request to enable the AppRole endpoint needed an authentication token. In this case we are passing the root token generated when we started the Vault server. We could also generate tokens using any other authentication mechanisms, but we will use the root token for simplicity.

Now create an AppRole with desired set of 

[ACL policies](https://www.vaultproject.io/docs/concepts/policies.html)

.

The 

[Policies](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies)

 guide used CLI to create `my-policy`. In this guide, use the `/sys/policies/acl` endpoint to create the same policy via Vault API.

:ship:
```bash
curl \
```
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request PUT \
        --data '{"policy":"# Dev servers have version 2 of KV secrets engine mounted by default, so will\n# need these paths to grant permissions:\npath \"secret/data/*\" {\n  capabilities = [\"create\", \"update\"]\n}\n\npath \"secret/data/foo\" {\n  capabilities = [\"read\"]\n}\n"}' \
        http://127.0.0.1:8200/v1/sys/policies/acl/my-policy

Since `my-policy` expects `secret/data` path to exist, enable KV v2 secrets engine at `secret/` using API.

:ship:
```bash
curl \
```
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{ "type":"kv-v2" }' \
        http://127.0.0.1:8200/v1/sys/mounts/secret

The following command specifies that the tokens issued under the AppRole `my-role` should be associated with `my-policy`.

:ship:
```bash
curl \
```
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        --data '{"policies": ["my-policy"]}' \
        http://127.0.0.1:8200/v1/auth/approle/role/my-role

The AppRole auth method expects a RoleID and a SecretID as its input. The RoleID is similar to a username and the SecretID can be thought as the RoleID's password.

The following command fetches the RoleID of the role named `my-role`.

:ship:
```bash
curl \
```
        --header "X-Vault-Token: $VAULT_TOKEN" \
         http://127.0.0.1:8200/v1/auth/approle/role/my-role/role-id | jq -r ".data"

The response will include the `role_id`:

    {
      "role_id": "3c301960-8a02-d776-f025-c3443d513a18"
    }

This command creates a new SecretID under the `my-role`.

:ship:
```bash
curl \
```
        --header "X-Vault-Token: $VAULT_TOKEN" \
        --request POST \
        http://127.0.0.1:8200/v1/auth/approle/role/my-role/secret-id | jq -r ".data"

The response will include the `secret_id`:

    {
      "secret_id": "22d1e0d6-a70b-f91f-f918-a0ee8902666b",
      "secret_id_accessor": "726ab786-70d0-8cc4-e775-c0a75070e5e5"
    }

These two credentials can be supplied to the login endpoint to fetch a new Vault token.

:ship:
```bash
curl --request POST \
```
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

The returned client token (`s.p5NB4dTlsPiUU94RA5IfbzXv`) can be used to authenticate with Vault. This token will be authorized with specific capabilities on all the resources encompassed by the `default` and `my-policy` policies. (As it was mentioned in the 

[Policies](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault/getting-started/policies) guide, the `default` policy is attached to all tokens by default. )



The newly acquired token can be exported as the `VAULT_TOKEN` environment variable value and used to authenticate subsequent Vault requests.

:ship:
```bash
export VAULT_TOKEN="s.p5NB4dTlsPiUU94RA5IfbzXv"
```

Create a version 1 of secret named `creds` with a key `password` and its value set to `my-long-password`.

:ship:
```bash
curl \
```
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

:ship:
```bash
unset VAULT_TOKEN
```

You can see the documentation on the 

[HTTP APIs](https://www.vaultproject.io/api/index.html)

 for more details on other available endpoints.

Congratulations! You now know all the basics needed to get started with Vault.> Vault comes with support for a user-friendly and functional web UI out of the box. In this guide we will explore the Vault UI.

## Web UI | Vault - HashiCorp Learn

Vault features a user interface (web interface) for interacting with Vault. Easily create, read, update, and delete secrets, authenticate, unseal, and more with the Vault UI.

### Dev servers

When you start the Vault server in dev mode, Vault UI is automatically enabled and ready to use.

:ship:
```bash
vault server -dev
```
    ...

Open a web browser and enter `http://127.0.0.1:8200/ui` to launch the UI.

Enter the initial root token to sign in.

!

[Sign in](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/img/vault-autounseal-3.png)



### Non-Dev servers

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

### Web UI Wizard

Vault UI has a built-in guide to navigate you through the common steps to operate various Vault features.

!

[Sign in](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/img/vault-ui-wizard.png)

> Resources and further tracks now that you're confident using Vault.

## Next Steps | Vault - HashiCorp Learn



[](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault)





[Learn Vault](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault)



/Getting Started

*   

[Getting Started](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=getting-started#getting-started)


*   

[Kubernetes](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=getting-started-k8s#getting-started-k8s)


*   

[Product Certification Exam Prep](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=certification#certification)


*   

[Vault 1.4 Release Highlights](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=new-release#new-release)


*   

[Day 1: Deploying Your First Vault Cluster](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=day-one#day-one)


*   

[Secrets Management](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=secrets-management#secrets-management)


*   

[Advanced Data Protection](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=ADP#ADP)


*   

[Data Encryption](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=encryption-as-a-service#encryption-as-a-service)


*   

[Access Management](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=identity-access-management#identity-access-management)


*   

[Monitoring & Troubleshooting](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=monitoring#monitoring)


*   

[Security](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=security#security)


*   

[Operations](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=operations#operations)


*   

[Developer](chrome-extension://cjedbglnccaioiolemnfhjncicchinao/vault?track=developer#developer)



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

*   

[Documentation](https://www.vaultproject.io/docs/index.html)

 - The documentation is an in-depth reference guide to all the features of Vault.

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