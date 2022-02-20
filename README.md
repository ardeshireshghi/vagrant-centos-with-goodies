# vagrant-centos-with-goodies

This is a simple Vagrant setup to spin up a CentOS 7 VM with Docker, Python, Nginx and Ruby running on it.

## Installation

### Dependencies

1. [Vagrant (VM lifecycle manager)](https://www.vagrantup.com/docs/installation)
2. Virtualbox

### Running VM

Assuming `vagrant` is up and running (try `vagrant -v`), run:

```bash
$ vagrant up
```

This will go through `Vagrantfile` and setup the VM. The current file installs CentOs7 with the following packages:

- Nginx
- Ruby
- Python 3
- Docker and Docker compose (Use `DOCKER_USER` and `DOCKER_PASSWORD` env vars to login to Docker hub)
- Vim
- A test web service using Python's `FastAPI` and a `systemd` unit to run it on boot

Also if you want to have a convinient way of logging in to vm from the host command line add this to your shell profile:

`alias linux="pushd path-to-this-repo > /dev/null && ./login; popd > /dev/null"`

and then you can use `linux` as a command to login.
