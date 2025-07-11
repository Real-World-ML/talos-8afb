# How to get a real world 1 node Kubernetes cluster, in less than 10 minutes.

What is [Talos Linux](https://www.talos.dev/) ?

A very versatile Linux distribution, which ships with Kubernetes, with an API which can be accessed with a CLI client.

How to get things going on real hardware that maybe sits around your house, like a Raspberry Pi 5. 

[![demo](demo.gif)](demo.gif)


## 1. The local tooling

```sh
brew install zstd talosctl
```

## 2. Prepare the RPi

Find out on MacOS ```diskutil list``` which disk is your SD-Card.

Download, unpack ```metal-arm64.raw.zst``` from https://github.com/talos-rpi5/talos-builder/releases/tag/v1.10.2-rpi5-pre3 and ```dd``` to a SD-Card with:

```sh
sudo dd if=./metal-arm64.raw of=/dev/disk4 conv=fsync bs=4M
```

Plug-in the SD-Card and the network cable and power up the RPi. 
Let's assume the RPi gets an IP inside your network and that is 192.168.1.155.

Talos is now in a recovery mode, awaiting 2 more steps, one for configuration and one for boostraping Kubernetes. 


## 3. Generate Talos config

```sh
mkdir talos-8afb && cd talos-8afb 
```

```sh
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

Once the configuration is applied, which takes less than 2 minutes, you can verify it:

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig version
```

## 4. Boostrap

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig bootstrap 
```


## 5. Verify

At this moment also you can follow the Linux kernel logs via: 

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig dmesg -f 
```

Or see Talos dashboard:

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig dashboard  
```

## 6. Get Kubeconfig

```sh
export KUBECONFIG="kubeconfig"
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig kubeconfig
```

Verify with

```sh
kubectl get nodes -A -o wide 
```

## 7. Misc 

### Reset the node if you want to start all over again

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig reset
```

Also use the ```wipe: true```

https://github.com/Real-World-ML/talos-8afb/blob/df92dec48f1057e2a1e428089b1721ccf7f9d297/controlplane.yaml#L19-L20

---

Links:

https://github.com/siderolabs/sbc-raspberrypi/issues/23#issuecomment-2799643923
