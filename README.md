# JohnTheRipper MPI Cluster

Ansible project to automatically deploy a [John The Ripper](http://www.openwall.com/john/) CPU-based password cracking cluster on APT-based hosts - built on top of my [Ansible MPI project](https://github.com/FlyingTopHat/Ansible-MPI-Cluster).

![Bash terminal showing master and slave nodes running JtR](https://pbs.twimg.com/media/DDkeuIFXkAElI0t.png:large)

*I should caveat the above by saying that I've only tested the scripts against Ubuntu v14.04. If you discover any problems using the playbook be sure to issue a pull-request.*

## Setup

To configure your Ubuntu machines as an MPI cluster first create an [inventory file](http://docs.ansible.com/ansible/intro_inventory.html) with a `master` node and `slaves` group.

```yml
master 10.211.55.178

[slaves]
10.211.55.177
10.211.55.175
10.211.55.176
```
Execute the Playbooks against the hosts, to setup MPI and install John The Ripper:

```bash
cd ./provisioning
ansible-playbook site.yml -i <PATH TO INVENTORY>
```

## Usage

The setup will provide you with a shared directory (by default `/srv/mpi_data`) inside of which you put your hashes and wordlists etc. The slaves will then be able to access them along with the JtR binary also in the directory.

```bash
$ scp /home/walsingham/mary_hashes mpiuser@master:/srv/mpi_data
```

SSH into the master as `mpiuser` and start JohnTheRipper going by pointing MPI to the file listing the hosts and the JtR binary with your cracking options:

```bash
$ ssh mpiuser@master 'mpiexec \
    -hostfile /etc/mpi_hosts \
    /srv/mpi_data/JohnTheRipper/run/john /srv/mpi_data/mary_hashes'
```

*Don't forget to adjust the number of available processors on each host in the [hostfile](https://www.open-mpi.org/faq/?category=running#mpirun-hostfile) `/etc/mpi_hosts`*

## Development

[Vagrant](https://www.vagrantup.com/) is used to create development VMs against which you can test changes to the Ansible scripts.

1. [Install Vagrant](https://www.vagrantup.com/docs/installation/).

2. Provision and configure
the local cluster with:

```bash
vagrant up
```

Once provisioned the [Ansible Provisioner](https://www.vagrantup.com/docs/provisioning/ansible.html) creates a inventory which can be used manually with the `ansible-playbook` utility:

```bash
cd ./provisioning
ansible-playbook site.yml -i ../.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
```

Or you can let Vagrant do the provisioning:

```bash
vagrant provision
```
