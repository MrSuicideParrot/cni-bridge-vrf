# cni-bridge-vrf

A CNI plugin that assignes bridges to [VRFs](https://www.kernel.org/doc/html/latest/networking/vrf.html).

CNI networks of the type `bridge`, in `ipMasq` mode, use the routing table of the host to route traffic, which means that without firewall rules and a custom routing table it is not possible to implement network segregation. This process has a major incovenient, you need to maintain a custom set of firewall rules for each CNI network, that will mark traffic based on its source address and route it through a specific routing table.[^1]

This plugin tries to solve this complexity by putting automatically the bridge interface of a CNI network inside a VRF at your choice, with a separate routing table from your host.

CNI has an [official plugin for  VRF](https://www.cni.dev/plugins/current/meta/vrf/). However, when applied to a `bridge` network, it puts the [veth interface](https://man7.org/linux/man-pages/man4/veth.4.html) of the container inside a VRF and not the bridge interface. 

##  Instalation
* Download the [latest version](https://github.com/MrSuicideParrot/cni-bridge-vrf/releases) of the cni-bridge-vrf plugin.
* Move the binary to your CNI plugin location.
       * For Podman: `/usr/lib/cni`
* You are ready to use this plugin!

## Build it yourself
Just run the following command:
```
go build -o bridge-vrf
```

## Configuration

To configure a CNI network to use this plugin you need to add the following json to the *plugins* array of your configuration file. The value of the *vrfname* key should be the name of the vrf that you want to assign. If the VRF doesn't exists, it will be created on the moment of the bridge creation.  

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
   "name": "vtf-teste",
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