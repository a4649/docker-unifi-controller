# Unifi Controller Docker Container

This is a fork of brijohn's unifi-controller which has been updated to the latest version.

> Stable 5.x branch of [Unifi Controller](https://www.ubnt.com/download/unifi/)
> This release: v5.12.72

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


## Running the Container

Create a volume to store the unifi persistence data.

```sh
docker volume create --name unifi
```

Next launch the container using the previously created volumes.

```sh
docker run -d -p 8080:8080 -p 8443:8443 -p 3478:3478/udp \
-p 10001:10001/udp -v unifi:/usr/lib/unifi/data \
--name unifi lumel/unifi-controller
```


## Authors
- Henry Southgate - [Github](https://github.com/HenryJS/)
- Brian Johnson - [Github](https://github.com/brijohn/) - brijohn@gmail.com

Distributed under the GPL 3 license. See ``LICENSE`` for more information.
