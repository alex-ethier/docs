---
layout: docs
title: CoreOS Quick Start
#redirect handled in alias_generator.rb
---

# Quick Start

If you don't have a CoreOS machine running, check out the guides on running CoreOS on [Vagrant][vagrant-guide], [Amazon EC2][ec2-guide], [QEMU/KVM][qemu-guide], [VMware][vmware-guide] and [OpenStack][openstack-guide]. With either of these guides you will have a machine up and running in a few minutes.

It's highly recommended that you set up a cluster of at least 3 machines &mdash; it's not as much fun on a single machine. If you don't want to break the bank, [Vagrant][vagrant-guide] allows you to run an entire cluster on your laptop. For a cluster to be properly bootstrapped, you have to provide cloud-config via user-data, which is covered in each platform's guide.

CoreOS gives you three essential tools: service discovery, container management and process management. Let's try each of them out.

First, connect to a CoreOS machine via SSH as the user `core`. For example, on Amazon, use:

```sh
ssh core@an.ip.compute-1.amazonaws.com
```

## Service Discovery with etcd

The first building block of CoreOS is service discovery with **etcd** ([docs][etcd-docs]). Data stored in etcd is distributed across all of your machines running CoreOS. For example, each of your app containers can announce itself to a proxy container, which would automatically know which machines should receive traffic. Building service discovery into your application allows you to add more machines and scale your services seamlessly.

If you used an example [cloud-config]({{site.url}}/docs/cluster-management/setup/cloudinit-cloud-config) from a guide linked in the first paragraph, etcd is automatically started on boot. The API is easy to use. From a CoreOS machine, you can simply use curl to set and retrieve a key from etcd:

Set a key `message` with value `Hello world`:

```sh
curl -L http://127.0.0.1:4001/v1/keys/message -d value="Hello world"
```

Read the value of `message` back:

```sh
curl -L http://127.0.0.1:4001/v1/keys/message
```

If you followed a guide to set up more than one CoreOS machine, you can SSH into another machine and can retrieve this same value.

#### More Detailed Information
<a class="btn btn-primary" href="{{ site.url }}/docs/distributed-configuration/getting-started-with-etcd/" data-category="More Information" data-event="Docs: Getting Started etcd">View Complete Guide</a>
<a class="btn btn-default" href="{{site.url}}/docs/distributed-configuration/etcd-api/">Read etcd API Docs</a>

## Container Management with docker

The second building block, **docker** ([docs][docker-docs]), is where your applications and code run. It is installed on each CoreOS machine. You should make each of your services (web server, caching, database) into a container and connect them together by reading and writing to etcd. You can quickly try out a Ubuntu container in two different ways:

Run a command in the container and then stop it: 

```sh
docker run busybox /bin/echo hello world
```

Open a shell prompt inside the container:

```sh
docker run -i -t busybox /bin/sh
```

#### More Detailed Information
<a class="btn btn-primary" href="{{ site.url }}/docs/launching-containers/building/getting-started-with-docker" data-category="More Information" data-event="Docs: Getting Started docker">View Complete Guide</a>
<a class="btn btn-default" href="http://docs.docker.io/">Read docker Docs</a>

## Process Management with systemd

The third building block of CoreOS is **systemd** ([docs][systemd-docs]) and it is installed on each CoreOS machine. You should use systemd to manage the life cycle of your docker containers. The configuration format for systemd is straightforward. In the example below, the Ubuntu container is set up to print text after each reboot:

First, you will need to run all of this as `root` since you are modifying system state:

```sh
sudo -i
```

Create a file called `/etc/systemd/system/hello.service`:

```ini
[Unit]
Description=My Service
After=docker.service
Requires=docker.service

[Service]
Restart=always
RestartSec=10s
ExecStart=/bin/bash -c '/usr/bin/docker start -a hello || /usr/bin/docker run --name hello busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"'
ExecStop=/usr/bin/docker stop -t 1 hello

[Install]
WantedBy=multi-user.target
```

See the [getting started with systemd]({{site.url}}/docs/launching-containers/launching/getting-started-with-systemd) page for more information on the format of this file.

Then run enable and start the unit:

```sh
sudo systemctl enable /etc/systemd/system/hello.service
sudo systemctl start hello.service
```

Your container is now started and is logging to the systemd journal. You can read the log by running:

```sh
journalctl -u hello.service -f
```

To stop the container, run:

```sh
sudo systemctl stop hello.service
```

#### More Detailed Information
<a class="btn btn-primary" href="{{ site.url }}/docs/launching-containers/launching/getting-started-with-systemd" data-category="More Information" data-event="Docs: Getting Started systemd">View Complete Guide</a>
<a class="btn btn-default" href="http://www.freedesktop.org/wiki/Software/systemd/">Read systemd Website</a>

#### Chaos Monkey
During our alpha period, Chaos Monkey (i.e. random reboots) is built in and will give you plenty of opportunities to test out systemd. CoreOS machines will automatically reboot after an update is applied unless you [configure them not to]({{site.url}}/docs/cluster-management/debugging/prevent-reboot-after-update).
