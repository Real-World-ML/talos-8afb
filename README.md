## 1. The local tooling

```sh
brew install zstd talosctl
```


## 2. Download the disk image for RPI5 

Download ```v1.10.2-rpi5-pre3``` from https://github.com/talos-rpi5/talos-builder/releases/tag/v1.10.2-rpi5-pre3
The disk image is ```metal-arm64.raw.zst```


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

## 2.

```sh
sudo dd if=./metal-arm64.raw of=/dev/disk4 conv=fsync bs=4M 
```

## 
```sh
talosctl -n 192.168.1.155 -e 192.168.1.155 --talosconfig=./talosconfig bootstrap 
```


## 

```sh
talosctl --talosconfig=./talosconfig -n 192.168.1.155 -e 192.168.1.155 dashboard
```

