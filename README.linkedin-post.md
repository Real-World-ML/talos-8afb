How to get a real world 1 node Kubernetes cluster, in less than 10 minutes.

What is Talos Linux ?

A very versatile Linux distribution, which ships with Kubernetes, with an API which can be accessed with a CLI client.

How to get things going on real hardware that maybe sits around your house, like a Raspberry Pi 5. 

Step 1. Prepare the RPi

Download, unpack ```metal-arm64.raw.zst``` https://github.com/talos-rpi5/talos-builder/releases/tag/v1.10.2-rpi5-pre3 and ```dd``` to a SD-Card with ```sudo dd if=./metal-arm64.raw of=/dev/disk4 conv=fsync bs=4M```

Plug-in the SD-Card and the network cable and power up the RPi. 
Let's assume the RPi gets an IP inside your network and that is 192.168.1.155.

Step 2. Configure and Boostrap Talos.

Talos is now in a recovery mode awaiting 2 more steps, one for configuration and one for boostrap-ing Kubernetes. 

```sh
mkdir talos-8afb && cd talos-8afb
talosctl gen config ll-8afb https://192.168.1.155:6443 --with-examples=false --with-docs=false
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

And lastly, boostraping our 1 node Kubernetes cluster: 

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig bootstap
```

At this moment also you can follow the Linux kernel logs via: 

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig dmesg -f 
```

Or see Talos dashboard

```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig dashboard  
```

---

Binaries on MacOS via brew:

```
brew install zstd talosctl
```

Checkout our repository at: 

https://github.com/Real-World-ML/talos-8afb

---

Links:

https://www.talos.dev/
https://www.talos.dev/v1.10/reference/cli/