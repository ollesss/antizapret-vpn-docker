# AntiZapret VPN in Docker

Easy-to-use Docker image based upon original [AntiZapret LXD image](https://bitbucket.org/anticensority/antizapret-vpn-container/src/master/) for self-hosting.


# Improvements

- Patches: [Apple](./rootfs/etc/knot-resolver/kresd.conf#L53-L61), [IDN](./rootfs/root/patches/parse.patch#L16), [RU](./rootfs/etc/knot-resolver/kresd.conf#L63-L73)
- [Community-driven list](./rootfs/root/antizapret/config/include-hosts-dist.txt) with geoblocked and unlisted domains
- Option to use [openvpn-dco](https://openvpn.net/as-docs/tutorials/tutorial--turn-on-openvpn-dco.html), a kernel extension for improving performance
- Option to [forwarding queries](./rootfs/init.sh#L21-L35) to an external resolver


# Installation

> Quick start: use the example from [docker-compose.yml](./docker-compose.yml) to run the container; it will be pulled from Docker Hub

To run this container you need to install [Docker Engine](https://docs.docker.com/engine/install/):

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

If you wanna run the container from source code, run the following commands:

```bash
git clone https://github.com/xtrime-ru/antizapret-vpn-docker.git antizapret
cd antizapret
docker compose up -d --build
```

After initialization of the container, you can pull `.ovpn` configs from [keys/client](./keys/client) directory.
There will be UDP and TCP configurations.
Use UDP for better performance.
Use TCP in unstable conditions.


# Documentation

## Adding Domains/IPs
Any domains or IPs can be added or excluded from routing with [config files](./config).
These lists are added/excluded to/from automatically generated lists of domains and IP's.
Reboot container and wait few minutes for applying changes.


## Keys Persistence

Client and server keys are stored in [keys](./keys).
They are persistent between container and host restarts.

To regenerating the keys use the following commands:
```shell
docker compose down
rm -rf keys/{client,server}/keys/*.{crt,key}
docker compose up -d
```


## Environment Variables

You can define these variables in docker-compose.yml file for your needs:

- `DOMAIN=example.com` — will be used as a server address in .ovpn profiles upon keys generation (default: your server's IP)
- `PORT=1194` — will be used as a server port in .ovpn profiles upon keys generation (default: 1194)
- `DNS=1.1.1.1` — DNS server to resolve domains (default: host DNS server)
- `DNS_RU=77.88.8.8` — russian DNS server; used to fix issues with geo zones mismatch for domains like [apple.com](apple.com)


## Enable OpenVPN Data Channel Offload (DCO)
[OpenVPN Data Channel Offload (DCO)](https://openvpn.net/as-docs/openvpn-dco.html) provides performance improvements by moving the data channel handling to the kernel space, where it can be handled more efficiently and with multi-threading.
**tl;dr** it increases speed and reduces CPU usage on a server.

Kernel extensions can be installed only on <u>a host machine</u>, not in a container.

### Ubuntu 24.04
```bash
sudo apt update
sudo apt upgrade # reboot your system after upgrade
sudo apt install -y efivar
sudo apt install -y openvpn-dco-dkms
```

### Ubuntu 20.04
```bash
deb=openvpn-dco-dkms_0.0+git20231103-1_all.deb
sudo apt update
sudo apt upgrade # reboot your system after upgrade
sudo apt install -y efivar dkms linux-headers-$(uname -r)
wget http://archive.ubuntu.com/ubuntu/pool/universe/o/openvpn-dco-dkms/$deb
sudo dpkg -i $deb
```


# Credits
- [ProstoVPN](https://antizapret.prostovpn.org) — the original project
- [AntiZapret VPN Container](https://bitbucket.org/anticensority/antizapret-vpn-container/src/master/) — source code of the LXD-based container
- [AntiZapret PAC Generator](https://bitbucket.org/anticensority/antizapret-pac-generator-light/src/master/) — proxy auto-configuration generator to bypass censorship of Russian Federation
- [No Thought Is a Crime](https://ntc.party) — a forum about technical, political and economical aspects of internet censorship in different countries