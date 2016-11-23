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
- Debian
- CentOS

More infos in the role's metadata file.

## Dependencies

- [**`abessifi.java`**](https://galaxy.ansible.com/abessifi/docker/) - To install Oracle JDK ( version >= 7 ).
- [**`abessifi.docker`**](https://galaxy.ansible.com/abessifi/docker/) - To install Docker Engine.

## Role Variables

- **`fitnesse_firefox_version`** - Install specific Firefox version (default: `41.0`).
- **`fitnesse_firefox_set_default`** - If set to `true` the current `fitnesse_firefox_version` of firefox with be set as system default one (default: `false`).
- **`fitnesse_version`** - The FitNesse standalone JAR version (default: `20151230`).
- **`fitnesse_get_xebium`** - If set to true, Ansible will download and install the Xebium library within the `/usr/local/lib/fitnesse` lib directory (default: `false`).
- **`fitnesse_xebium_download_url`** - The URL to download the Xebium library (default: none).
- **`fitnesse_get_project`** - If set to true, Ansible will automatically download and install your FitNesse project which should be an archived `FitNesseRoot` folder (default: `false`).
- **`fitnesse_project_download_url`** - The URL to download the tarball of FitNesse tests (default: none).
- **`fitnesse_clean_install`** - Set this to `true` if you want to override an existing FitNesseRoot folder with new tests scripts (default: `false`).
- **`fitnesse_service_port`** - The default port number to which FitNesse service is binded. The port number should be >=1024 (default: `8080`).
- **`fitnesse_java_options`** - You can pass some java options to the java commande which runs FitNesse. E.g: `-Xmx2000M -Xms1000M -Duser.country=FR -Duser.language=fr -Dfile.encoding=UTF-8` (default: none).
- **`fitnesse_provision_mode`** - This parameter indicates the way fitnesse framework will be installed. The fitnesse service may be installed and configured directly on the targeted host(s) if you set the parameter to `on-host`. On the other hand, if the parameter is positioned to `docker`, Ansible will think provisioning the platform with a Docker container. Supported values are ['on-host', 'docker'] (default: `on-host`).
- **`fitnesse_docker_ctn_name`**  - The fitnesse docker container name (default: `fitnesse`).
- **`fitnesse_docker_img_name`**  - The fitnesse docker image name (default: `abessifi/fitnesse:latest`).
- **`fitnesse_docker_ctn_port`**  - The expose port of the fitnesse container (default: `8080`).
- **`fitnesse_docker_host_port`** - The host port to map to the fitnesse container expose port. This is useful if you want to reach fitnesse via the host IP (default: `8080`).

## Available tags

- **`fitnesse-setup`** - This tag is to perform a complete setup of FitNesse framework.
- **`fitnesse-start-service`** - Starts the FitNesse service.
- **`fitnesse-start-service`** - Stops the FitNesse service.
- **`fitnesse-restart-service`** - Restarts the FitNesse service.
- **`fitnesse-build-docker-image`** - Use this tag if you want to provision a Docker image for FitNesse.
- **`fitnesse-setup-container`** - If set, this tag starts up a Docker container for FitNesse.
- **`fitnesse-remove-container`** - Use this tag if you want to destroy the FitNesse container. It can be combined with `fitnesse-setup-container` to reinstall the FitNesse container.
- **`fitnesse-install-firefox`** - Use this tag if you want to install a custom Firefox version.

## Usage

### FitNesse on host

In order to install FitNesse framework in mode `on-host`, start by checking out the role from Ansible galaxy:

```bash
ansible-galaxy install abessifi.fitnesse
```

Here is a setup example of FitNesse on host from scratch:

```yaml
---
- hosts: all
  roles:
    # Install Oracle JDK 7 if provisioning mode is 'on-host'
    - role: abessifi.java
      java_jdk_type: 'oracle'
      become: yes
      when: fitnesse_provision_mode == 'on-host'
    # Install FitNesse
    - role: ansible-fitnesse
      fitnesse_firefox_version: '42.0'
      fitnesse_firefox_set_default: true
```

**NOTE:** The Ansible tags `install-java` and `fitnesse-setup` should be specified for this kind of installation.

### FitNesse with Docker

In the other hand, you can run FitNesse through a Docker container. We have prepared an image for you and you can build yours :)

#### Build FitNesse image

If you are a DevOps enthusiast, you can use the magic tool `Packer` and its plugin `packer-post-processor-docker-dockerfile` to provision, build, tag and push the Docker image. All actions listed in one simple and flat json file called a Packer template.

A sample template `test/packer/build_image.json` exists already to build the image `foobar/fitnesse:latest` and push it to the public Docker Hub registry. Feel free to adapt this template file to build and push your own Docker image to a public/private registry.

After installing Packer and the dockerfile plugin, you can build the image as follow:

```bash
cd ./test/packer/
packer build build_image.json
```

Here is an example of Ansible playbook that could be used to provision the image with FitNesse and Xebium library:

```yaml
---
- hosts: localhost
  roles:
    - role: ansible-fitnesse
      fitnesse_provision_mode: 'docker'
      fitnesse_get_xebium: true
      fitnesse_xebium_download_url: 'http://url/to/the/xebium/jar/archive'
```

**NOTE:** Make sure that the Ansible tag `fitnesse-build-docker-image` is specified within the Packer template file:

```json
...
  "extra_arguments": [
    "--tags=fitnesse-build-docker-image"
  ]
...
```

#### Run FitNesse container

Back to our fitnesse role ! This time, we want to provision the platform with FitNesse "dockerized". To do so, start by checking out the role Docker role from Ansible galaxy:

```bash
ansible-galaxy install abessifi.fitnesse
```

then, create a playbook that mentions the different roles and parameters to provision the platform. Suppose in the following example that we want to:

- Install Docker Engine in order to run the FitNesse container
- Setup FitNesse configuration and install Firefox `v42` and make it the default used version (host side)
- Get an existing FitNesse project (a fitnesse project is a set of UAT scripts written with FitNesse code style)
- Run the FitNesse container
- Bind container port to the host one (8080 => 9090)

```yaml
---
- hosts: all
  vars:
    fitnesse_provision_mode: 'docker'
  roles:
    # Install Docker Engine if provisioning mode is 'docker'
    - role: abessifi.docker
      when: fitnesse_provision_mode == 'docker'
    # Install FitNesse
    - role: ansible-fitnesse
      fitnesse_docker_host_port: 9090
      fitnesse_firefox_version: '42.0'
      fitnesse_firefox_set_default: true
      fitnesse_get_project: true
      fitnesse_project_download_url: 'http://url/to/existing/fitnesse/project/tarball'
```

**NOTE:** Please make sure you call Ansible with the following tags: `docker-setup`, `docker-install-dockerpy`, `fitnesse-install-firefox` and `fitnesse-setup-container`.

Your container is up and running fitnesse !
The fitnesse server should be reachable now on http://<HOST_IP>:9090/

## Development and testing

### Test with Vagrant

For quick tests, you can spin up a Ubuntu VM using Vagrant. In the project directory `test/vagrant/` you can customize the Vagrantfile that describes your virtual machine and adapt a set of Ansible inventory files that suit your environment (IP addresses, SSH credentials, etc).

>**Note:** The `ansible.fitnesse` role must be ran with a `sudo` user. In this example, the `vagrant` user has sudo privileges already configured.

Start by getting the project from Github or Ansible Glaxy, then enter the `test/vagrant/` directory.

Now, run and provision the VM:

```bash
vagrant up
```

Your testing VM is up and running with FitNesse framework !

### Run acceptance tests

Not yet implemented.

## Author Information

This role was created by [Ahmed Bessifi](https://www.linkedin.com/in/abessifi), a DevOps enthusiast.
