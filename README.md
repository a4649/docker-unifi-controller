# Unifi Controller Docker Container

This is Yet Another UniFi Controller.  It draws UniFi from the vendor-provided apt source and updates frequently.


## Update Policy

The image is updated on the following policy:

* Weekly, Sundays at 00:00: The `master` branch will be re-built and tagged with both `latest` and the relevant UniFi revision.  This allows for upstream packages to be updated.  It will always use the latest version of UniFi.
* Ad-hoc: As and when I make changes, I'll push to `dev`.  It'll probably work but it also might break.

For stability, choose `latest`.  For a specific UniFi revision, choose the tag, but beware the platform will be out of date and will need an update.  Some of the builds are quite old.  For whatever I'm poking, choose `dev`.


## Image Installation

From Docker Hub:

```sh
docker pull lumel/unifi-controller
```
From Source:

```sh
git clone https://github.com/HenryJS/docker-unifi-controller.git
cd docker-unifi-controller
docker build -t "unifi-controller:latest" --rm .
```


## Running the Container with network host mode

Create a volume to store the UniFi persistence data, then launch the 
container using the previously created volumes.

```sh
docker volume create --name unifi
docker run -d \
           --net=host \ 
           -v unifi:/usr/lib/unifi/data \
           --name lumel-unifi \
           lumel/unifi-controller
```

If, like me, you'd rather maintain state in a specific place in the local 
filesystem, do this instead:

```sh
DATA_PATH=/wherever/unifi-controller
mkdir -p $DATA_PATH
docker run -d \
           --net=host \
           -v ${DATA_PATH}:/usr/lib/unifi/data \
           --name lumel-unifi \
           lumel/unifi-controller
```
## Running the Container with specific ports

| Port | Function |
| ---- | -------- |
| 8443 | Unifi web admin port |
| 3478/udp | Unifi STUN port |
| 10001/udp | Required for AP discovery |
| 8080 | Required for device communication |
| 1900/udp | Required for Make controller discoverable on L2 network option |
| 8843 | Unifi guest portal HTTPS redirect port |
| 8880 | Unifi guest portal HTTP redirect port |
| 6789 | For mobile throughput test |
| 5514/udp | Remote syslog port |

```sh
DATA_PATH=/wherever/unifi-controller
mkdir -p $DATA_PATH
docker run -d --name unifi \ 
           -p 8443:8443 \
           -p 3478:3478/udp \
           -p 8080:8080 \
           -v ${DATA_PATH}:/usr/lib/unifi/data \
           lumel/unifi-controller
```

## Manual Update

If you'd like to update the package / distro manually, use the following:

```sh
docker exec -it unifi sh -c 'apt update && apt dist-upgrade'
```

Note this will also update the UniFi controller, so if you're pinning that for stability reasons, you'll have to hold the package first:

```sh
docker exec -it unifi sh -c 'apt-mark hold unifi && apt update && apt dist-upgrade'
```

## Device adoption

SSH into device (for switches the default IP Address is 192.168.1.20), default password is ```ubnt```

```sh
ssh ubnt@192.168.1.20
```

Add default gateway:

```sh
route add default gw 192.168.1.1 eth0
```

Send adoption request:

```sh
set-inform http://ip-of-controller:8080/inform
```

Accept the adoption request in the controller Web UI. The device will restart and loose the default gateway. You need to add it again, but, after first provisioning step, the default ssh password won't work anymore. You need to use your unifi account username. Check it on the controller Web UI at **Setting > Site > Device Authentication > Username and password** and use it:

```sh
ssh your-username@192.168.1.20
```

Add default gateway:

```sh
route add default gw 192.168.1.1 eth0
```

The device should continue the provisioning process.


## Troubleshooting

**Q: Adoption fails, reporting that my devices can't connect to the controller.**

**A:** When adopting a device, the controller needs to tell it where to talk 
back to; i.e. where the controller is.  By default, this is autodetected; as 
this controller runs in a container, the address it finds is the container's 
private IP - which on my system is in 172.16.0.0/12.   

To resolve, go to your Unifi Settings, hit Controller, then enter your 
Controller Hostname / IP and select *Override inform host with controller 
hostname/IP*.

Note that by using '--net=host' above, this _ought_ to be fixed, as the controller
will bind directly to the host's interface.


## Author
- Henry Southgate - [Github](https://github.com/lumel-uk/)

Distributed under the GPL 3 license. See ``LICENSE`` for more information.

