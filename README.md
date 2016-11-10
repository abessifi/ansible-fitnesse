# Ansible FitNesse

## Description

This is an Ansible role to install FitNesse acceptance testing framework on GNU/Linux servers.

## Requirements

### Software Requirements

- **Python 2.7** or higher (available in the targeted servers)
- **Ansible 2.0** or higher (can be easily installed via `pip`. E.g: `sudo pip install ansible==2.0.0.2`)
- **[Vagrant](https://www.vagrantup.com) 1.8** or higher (for testing purposes)
- **Virtualbox** (for testing purposes with Vagrant)
- **[Oh-my-box](https://github.com/abessifi/oh-my-box)** tool, optional, if you want to quickly provision and package a Vagrant base box with **Ansible** and **Ruby** pre-installed (recommanded to write and run acceptance tests with test-kitchen against the Ansible role).

## Supported Systems

- Ubuntu

More infos in the role's metadata file.

## Dependencies

- [**`abessifi.docker`**](https://galaxy.ansible.com/abessifi/docker/) - Install Docker Engine.

## Role Variables

- **`fitnesse_firefox_version`** - Install specific Firefox version (default: `41.0`)
- **`fitnesse_firefox_set_default`** - If set to `true` the current `fitnesse_firefox_version` of firefox with be set as system default one (default: `false`)

## Available tags

- **`fitnesse-setup`** - This tag is to perform a complete setup of FitNesse framework.
- **`fitnesse-install-firefox`** - Use this tag is you want to install a custom Firefox version.

## Usage

In order to install FitNesse framework, start by checking out the role from Ansible galaxy:

```bash
ansible-galaxy install abessifi.fitnesse
```

Finally call the role within you Ansible playbook:

```yaml
---
- hosts: all
  roles:
    - ansible-fitnesse
```

## Development and testing

### Test with Vagrant

For quick tests, you can spin up a Ubuntu VM using Vagrant. In the project directory `test/vagrant/` you can customize the Vagrantfile that describes your virtual machine and adapt a set of Ansible inventory files that suit your environment (IP addresses, SSH credentials, etc).

>**Note:** The `ansible-fitnesse` role must be ran with a `sudo` user. In this example,
the `vagrant` user has sudo privileges already configured.

Start by getting the project from Github or Ansible Glaxy, then enter the `test/vagrant/` directory.

Now, run and provision the VM:

```bash
vagrant up
```

Your testing VM is up and running with Docker !

### Run acceptance tests

Acceptance/Integration tests could be ran against the role using the magic `test-kitchen` tool. All written acceptance tests are in the **./test/integration/** directory.

The `.kitchen.yml` file describes the testing configuration and the list of suite tests to run. By default, the instances will be converged with Ansible and ran in Vagrant virtual machines.

To list the instances:

    $ kitchen list

    Instance                    Driver   Provisioner      Verifier  Transport  Last Action
    default-ubuntu-1404-x64     Vagrant  AnsiblePlaybook  Busser    Ssh        <Not Created>
    ...

To run the default test suite, for instance, on a Ubuntu Trusty platform, run the following command:

    $ kitchen test default-ubuntu-1404-x64


## Author Information

This role was created by [Ahmed Bessifi](https://www.linkedin.com/in/abessifi), a DevOps enthusiast.
