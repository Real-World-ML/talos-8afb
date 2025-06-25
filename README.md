# How to get a real world 1 node Kubernetes cluster, in less than 10 minutes.

What is Talos Linux ?

A very versatile Linux distribution, which ships with Kubernetes, with an API which can be accessed with a CLI client.

How to get things going on real hardware that maybe sits around your house, like a Raspberry Pi 5. 

[![demo](demo.gif)](demo.gif)


## 1. The local tooling

```sh
brew install zstd talosctl
```


## 2. Download the disk image for RPI5 

Download ```v1.10.2-rpi5-pre3``` from https://github.com/talos-rpi5/talos-builder/releases/tag/v1.10.2-rpi5-pre3
The disk image is ```metal-arm64.raw.zst```


```sh
sudo dd if=./metal-arm64.raw of=/dev/disk4 conv=fsync bs=4M 
```


https://github.com/siderolabs/sbc-raspberrypi/issues/23#issuecomment-2799643923


## 1. Generate Talos config

```sh
mkdir talos-8afb && cd talos-8afb 
```

```
talosctl gen config ll-8afb https://192.168.1.155:6443 \
    --with-examples=false \
     --with-docs=false
```

```talosctl``` generated the YAML configuration under 2 files, a controlplane and a worker YAMLs.
In the ```controlplane.yaml``` you need to touch very little things for start. Go over it, set ```debug: true``` and ```persist: true``` and the disk where Talos config should be persisted.
We are going to use the same SD-card so it's ```disk: /dev/mmcblk0```

And apply it:

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig apply-config -f ./controlplane.yaml --insecure
```

## 2. Boostrap
```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig bootstrap 
```


## 3. Verify

```sh
talosctl --talosconfig=./talosconfig -n 192.168.1.155 -e 192.168.1.155 dashboard
```
