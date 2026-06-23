<!--
SPDX-FileCopyrightText: © 2026 James C. Womack
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# netham-early-access

[Netham Lock](https://en.wikipedia.org/wiki/Netham_Lock) is the point at which boats from the River Avon can access the Bristol Floating Harbour.

`netham` is a command line tool for acquiring temporary AWS credentials for a role assumed using the STS API via an OAuth 2.0/OpenID Connect web identity token.

The primary use case is to enable service users with identities stored in an OAuth 2.0/OpenID Connect compliant IAM service (e.g. Keycloak) to acquire access to resources via an AWS-style API (e.g. S3) by assuming a role that has access to those resources.

## Early access version

This repository contains a minimal version of `netham` written in Python. This is being developed and refined in collaboration with users during an early access period, to address pain points and improve user experience.

The goal of development during the early access period is to produce a reference implementation, which can then be used as the basis for a clean reimplementation. Python was chosen for this initial version to produce a clear and readable reference.

## Installation

[Python 3.11](https://www.python.org/downloads/release/python-3110/) or greater is required. It is recommended to install this package in a Python virtual environment.

### venv and pip

Using standard Python tooling, create a virtual environment

```shell
python -m venv --upgrade-deps netham-venv
```

Activate the environment

```shell
source netham-venv/bin/activate
```

Install the package into the environment

```shell
python -m pip install git+https://github.com/bristol-supercomputing/netham-early-access.git
```

This will install the latest on the `main` branch of the repository.

To install a particular tagged release, append `@tag` to the URL, e.g.

```shell
python -m pip install git+https://github.com/bristol-supercomputing/netham-early-access.git@0.1.0
```

### uv

[uv](https://docs.astral.sh/uv/) is an excellent tool that simplifies managing Python projects, packages and virtual environments. To install with `uv`, first create a virtual environment:

```shell
uv venv --prompt netham-venv
```

This will create a venv in directory `.venv` in the current directory.

Next, install the package into the virtual environment

```shell
uv pip install git+https://github.com/bristol-supercomputing/netham-early-access.git
```

To install a particular tagged release, append `@tag` to the URL, e.g.

```shell
uv pip install git+https://github.com/bristol-supercomputing/netham-early-access.git@0.1.0
```

## Usage

For [venv and pip](#venv-and-pip) installations, ensure the virtual environment is activated. For [uv](#uv) installations, prepend `uv run` to all the following commands and ensure they are run in the same directory that the virtual environment (`.venv`) was created.

### Configuration

`netham` must be configured with information about the token-issuing identity provider, the S3-credential issuing S3/STS API server, and the IAM role to be assumed.

This information can be provided in a [TOML](https://toml.io/)-format configuration file, or via command-line arguments.

Example TOML configuration file:

```toml
issuer_url = "https://idp.example.com/realms/isambard"
client_id = "netham-client"
role_arn = "arn:partition:iam::account-id:role/example-role"
sts_endpoint_url = "https://sts.storage.example.com"
s3_endpoint_url = "https://s3.storage.example.com"
assumed_role_duration_minutes = 100
```

Run

```shell
netham auth --help
```

for a description of each setting and the command-line argument form.

`netham`'s configuration is obtained by merging values from the following sources in order of increasing precedence:

1. Default config file: `$XDG_CONFIG_HOME/netham/config.toml` (falling back to `~/.config/netham/config.toml` when `XDG_CONFIG_HOME` is unset)
2. Local config file: the default is `netham.toml` in the current working directory, but this can be overridden with the `netham --config` option
3. Command-line arguments passed to `netham auth`

### Acquiring S3 credentials

Run

```shell
netham auth
```

You will be prompted to open a verification URL for the identity provider in a web browser. In the web browser authenticate to the identity provider and authorize the application if prompted. 

After authenticating and authorizing the application, `netham` will receive a web identity token and contact the STS endpoint to exchange this for temporary S3 credentials for the configured IAM role.

The S3 credentials will be written to a file. By default this file is written to the current working directory and named `creds_env.sh`. An alternative path can be specified using the `netham auth --output` option.

## Use the S3 credentials

The file output by `netham auth` is a shell script that places S3 credentials into standard [`AWS_`-prefixed environment variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html) that can be used by many S3 clients (e.g. [AWS CLI](https://aws.amazon.com/cli/) and [rclone](https://rclone.org/)).

Add these variables to your current shell environment

```shell
source creds_env.sh
```

Then, use an S3 client to access S3 storage that the configured IAM role has access to, e.g.

```shell
aws s3 ls s3://example-role-bucket
```

Some tools may require additional configuration to communicate with the relevant S3 endpoints.
