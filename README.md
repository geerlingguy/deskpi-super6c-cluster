# DeskPi Super6c 6-node Pi Cluster

[![CI](https://github.com/geerlingguy/deskpi-super6c-cluster/workflows/CI/badge.svg?branch=master&event=push)](https://github.com/geerlingguy/deskpi-super6c-cluster/actions?query=workflow%3ACI)

This repository contains examples and automation used in DeskPi Super6c-related videos on [Jeff Geerling's YouTube channel](https://www.youtube.com/c/JeffGeerling).

You might also be interested in another Raspberry-Pi cluster I've maintained for years, the [Raspberry Pi Dramble](https://www.pidramble.com), which is a Kubernetes Pi cluster in my basement that hosts [www.pidramble.com](https://www.pidramble.com). I also set up a [Turing Pi 2 Cluster](https://github.com/geerlingguy/turing-pi-2-cluster) in a similar fashion.

## Usage

  1. Make sure you have [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) installed.
  2. Copy the `example.hosts.ini` inventory file to `hosts.ini`. Make sure it has the `control_plane` and `node`s configured correctly (for my examples I named my nodes `deskpi[1-6].local`).
  3. Copy the `example.config.yml` file to `config.yml`, and modify the variables to your liking.

### Raspberry Pi Setup

I am running Raspberry Pi OS (64-bit, lite) on a set of six Raspberry Pi Compute Module 4s with 8GB of RAM and no built-in eMMC. I am using [32 GB SanDisk Extreme microSD cards](https://amzn.to/3G35QbY) to boot each node.

I flashed Raspberry Pi OS to the Pis using Raspberry Pi Imager.

To make network discovery and integration easier, I edited the advanced configuration in Imager (press Shift + Ctrl + X), and set the following options:

  - Set hostname: `deskpi1.local` (set to `2` for node 2, `3` for node 3, etc.)
  - Enable SSH: 'Allow public-key', and paste in my public SSH key(s)

After setting all those options, making sure the hostname is unique to each node (and matches what is in `hosts.ini`), I inserted the microSD cards into the respective slots on the underside of the board, and booted the cluster.

### SSH connection test

To test the SSH connection from my Ansible controller (my main workstation, where I'm running all the playbooks), I connected to each server individually, and accepted the hostkey:

```
ssh pi@deskpi1.local
```

This ensures Ansible will also be able to connect via SSH in the following steps. You can test Ansible's connection with:

```
ansible all -m ping
```

It should respond with a 'SUCCESS' message for each node.

### Cluster configuration

Run the playbook:

```
ansible-playbook main.yml
```

SSH into your control_plane node 

```
ssh pi@deskpi1.local
```

Open `cepham` shell

```
sudo cepham shell
```

Create a file with a password for the admin user

```
echo "mysupersecurepassword" > password.txt
```

Set the new password to the admin user and then delete the file

```
ceph dashboard ac-user-set-password admin -i password.txt && rm password.txt
```

Now you can login into https://deskpi1.local:8443 using `admin` and the password you just set and then follow up with this [Install Ceph in a Raspberry Pi 4 Cluster -> Expanding the Cluster With the Dashboard](https://ceph.io/en/news/blog/2022/install-ceph-in-a-raspberrypi-4-cluster/#expanding-the-cluster-with-the-dashboard)


### Upgrading the cluster

Run the upgrade playbook:

```
ansible-playbook upgrade.yml
```

### Monitoring the cluster

TODO.

### Shutting down the cluster

The safest way to shut down the cluster is to run the following command:

```
ansible all -B 500 -P 0 -a "shutdown now" -b
```

Then after you confirm the nodes are shut down, hold down the power button for about 5 seconds, and the board will power off the CM4 slots. Then you can switch off or disconnect your power supply.

## Author

The repository was created in 2022 by [Jeff Geerling](https://www.jeffgeerling.com), author of [Ansible for DevOps](https://www.ansiblefordevops.com), [Ansible for Kubernetes](https://www.ansibleforkubernetes.com), and [Kubernetes 101](https://www.kubernetes101book.com).
