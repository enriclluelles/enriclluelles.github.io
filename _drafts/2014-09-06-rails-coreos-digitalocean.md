---
layout: post
title: Deploying a Rails app in a CoreOS cluster at digitalocean
---

Two days ago, [Digital Ocean announced support for CoreOS images](https://www.digitalocean.com/company/blog/coreos-now-available-on-digitalocean/). As I had been willing to try spinning out a CoreOS cluster and play with it, I thought this was a good chance to do it.

So, let's get to it, here's the process that I followed:

1. Use Vagrant to spin up the machines
2. Edit the user-data file to let cloud-config set up the cluster
   automatically
3. Write the systemd unit files to and launch them with fleet

## Use Vagrant to spin up the machines

First of all install Vagrant if you haven't or update it if you have a
version older than 1.6.0. Once you've done that
First of all I cloned the ![coreos-vagrant
repo](https://github.com/coreos/coreos-vagrant) and modified it 
