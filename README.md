# cni-bridge-vrf

A CNI plugin to assign a bridge to a [VRF](https://www.kernel.org/doc/html/latest/networking/vrf.html).

CNI networks of the [type `bridge`](https://www.cni.dev/plugins/current/main/bridge/), in `ipMasq` mode, use the host's routing table to route traffic. Therefore, it's impossible to implement network segregation without firewall rules and a custom routing table. This process has a major inconvenience. It needs to maintain a custom set of firewall rules for each CNI network that will mark traffic based on its source address and route it through a specific routing table.[^1]

This plugin simplifies this process by automatically putting the CNI bridge interface assigned to a VRF of your choice.

CNI has an [official plugin for  VRF](https://www.cni.dev/plugins/current/meta/vrf/). However, when applied to a `bridge` network, it puts the [veth interface](https://man7.org/linux/man-pages/man4/veth.4.html) of the container inside a VRF and not the bridge interface. 

##  Instalation
1. Download the right binary from our [latest release](https://github.com/MrSuicideParrot/cni-bridge-vrf/releases).
2. Untar the binary and move it to your CNI plugin location.
   * Podman default location: `/usr/lib/cni`.
3. You are ready to use this plugin!

## Build it yourself
Just run the following command after cloning this repo.
```
go build -o bridge-vrf
```

## Configuration

To configure a CNI network to use this plugin, you need to add the following JSON to the *plugins* array of your configuration file. The value of the *vrfname* key should be the name of the VRF that you want to assign. If the VRF doesn't exist, it will be created at the moment of the bridge creation.  

```json
 {
       "type": "bridge-vrf",
       "vrfname": "vrf-blue"
 }
```

A complete example of a network configuration with this plugin:

```json
{
   "cniVersion": "0.4.0",
   "name": "vrf-teste",
   "plugins": [
       {
              "type": "bridge",
              "bridge": "cni-podman1",
              "isGateway": true,
              "ipMasq": true,
              "hairpinMode": true,
              "ipam": {
              "type": "host-local",
              "ranges": [
                     [
                     {
                            "subnet": "10.89.0.0/24",
                            "gateway": "10.89.0.1"
                     }
                     ]
              ]
              }
       },
       {
        "type": "bridge-vrf",
        "vrfname": "vrf-blue"
      }
   ]
}
```

---
This project was inspired by the official [CNI vrf plugin](https://www.cni.dev/plugins/current/meta/vrf/).

[^1]: https://williamsbdev.com/posts/docker-connection-marking/