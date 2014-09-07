---
layout: post
title: Deploying a Rails app in a CoreOS cluster at digitalocean
---

Some days ago, [Digital Ocean announced support for CoreOS images](https://www.digitalocean.com/company/blog/coreos-now-available-on-digitalocean/). As I had been willing to try spinning out a CoreOS cluster and play with it, I thought this was a good chance to do it.

So, let's get to it, here's the process that I followed:

1. Set up Vagrant and the Digital Ocean provider
2. Edit the user-data file and spin up the machines
3. Write the systemd unit files and launch them with fleet

</br>

## Set up Vagrant and the Digital Ocean provider

First of all install Vagrant if you haven't or update it if you have a
version older than 1.6.0. You should get it from [http://vagrantup.com](http://vagrantup.com).

Then install the `digital_ocean` plugin and patch it to include some
unpublished changes that I added to it to make user-data work

```bash
vagrant plugin install vagrant-digitalocean --plugin-version 0.7.0

cd ~/.vagrant.d/gems/vagrant-digitalocean-0.7.0

curl -O https://gist.githubusercontent.com/enriclluelles/86e474e890408dfb2f38/raw/0194a53aa03de83d2f7f02fee1942ce8aa5dcab4/vg-digitalocean.patch

patch -p1 < vg-digitalocean.patch
```

Once you've done that clone the repo that I forked off [the coreos-vagrant repo](https://github.com/coreos/coreos-vagrant) and modified it:

```bash
git clone git://github.com/enriclluelles/coreos-vagrant

cd coreos-vagrant
```
</br>

## Edit the user-data file and spin up the machines


Copy the user-data.sample file to user-data and edit it as you see fit.
It's where the cluster settings and the initial services that systemd
will fire up are defined.
Keep in mind that `vagrant up` will obtain a new token and set in place every single time, but the rest of settings will be untouched.

```bash
cp user-data.sample user-data
```

Then set an env variable with your Digital Ocean token for the provider
to be able to perform operations against the API.If you don't have one
you can create one in the Apps & API

```bash
export DIGITALOCEAN_API_TOKEN=MYTOKEN
```

And maybe set the number of instances that will be part of the cluster

```bash
export NUM_INSTANCES=3
```

Now you're ready to spin up the machines:

```bash
vagrant up --provider digital_ocean
```

And you should see something like this for each machine. Write down the ip:

```
==> core-03: Using existing SSH key: Vagrant
==> core-03: Creating a new droplet...
==> core-03: Assigned IP address: 178.62.22.220
```
</br>

## Write the systemd unit files and launch them with Fleet

Once the CoreOS cluster is up we want to run some services in it.
We'll use fleet to do that, so we first need to install Fleet on our
machine.
Here's how it's done on a Mac:

```bash
brew install fleetctl
```

You'll also need to define the endpoint for fleet to connect to. You can
point to any of the machines that are part of the cluster:

```bash
export FLEETCTL_ENDPOINT=http://178.62.22.220:4001
```

Now if you run `fleetctl list-machines` something like this should be the output:

```bash
MACHINE		IP		METADATA
af84e017...	178.62.22.159	-
e99e8080...	178.62.22.220	-
ff425669...	178.62.22.119	-
```

What I did was scaffold a [simple Rails app](https://github.com/enriclluelles/todo-app) and dockerized it
