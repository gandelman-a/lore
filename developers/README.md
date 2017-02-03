# Developers

## Table of Contents

* [Contributing](#contributing)
* [Discussion](#discussion)
* [Ansible Environments](#ansible-environments)
  * [Production](#production)
  * [Multinode Nodepool](#multinode-nodepool)
  * [Docker](#docker)
  * [Vagrant](#vagrant)
* [Local Development Environment](#local-development-environment)
  * [Local Testing](#local-testing)

## Contributing

Please see the [contributing documentation](contributing) for more information.

A brief note on repository naming...
> The puns get made, or the plank gets walked.
>
> ~Jesse Keating

## Discussion

For discussions about BonnyCI, join us on [freenode](https://freenode.net) #BonnyCi!

## Ansible Environments

The primary ansible playbook in hoist is `./install-ci.yml`, and the result can be validated with the `./tests/validate-ci.yml` playbook.  These playbooks need to support four different environments:

* [Production](#production)
* [Multinode Nodepool](#multinode-nodepool)
* [Docker](#docker)
* [Vagrant](#vagrant)

### Production

A live deployment of [hoist](www.github.com/BonnyCI/hoist) running a CI service, and uses the hoist inventory file `./inventory/ci`.

### Multinode Nodepool

Hoist is tested with a job running on multiple single-use nodepool nodes. The test job deploys hoist on the nodes and validates the result for every pull request. It uses the dynamic inventory script `./inventory/nodepool.py`, and closely resembles production.

### Docker

We have a TravisCI test that deploys hoist in a docker container and validates the result. This uses the `./inventory/allinone` inventory to deploy all components in a single container.

### Vagrant

The Vagrant environment closely resembles production and is only used for [local testing](#local-testing) using the `./inventory/vagrant` inventory.

## Local Development Environment

Before hacking on BonnyCI, we'll need to set up a development environment on our local machine.

Supported development platforms:

* [Ubuntu](dev-environment/ubuntu.md)
* [Apple's macOS](dev-environment/macOS.md)

### Local Testing

You can test changes to BonnyCI locally by exercising some tools defined in [hoist](www.github.com/BonnyCI/hoist). Hoist is a set of ansible playbooks that automate the deployment of a BonnyCI environment. Be sure that you have completed the `Virtualization Tools` section of your OS's [Development Environment](#development-environment) page before proceeding.

To use Vagrant for local testing, refer to [Testing with Vagrant](dev-environment/vagrant.md).
