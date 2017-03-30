# Dockerized PXE
A Docker image serving as a standalone [PXE](https://en.wikipedia.org/wiki/Preboot_Execution_Environment) (running Dnsmasq). This server can be placed in an existing network infrastructure with an already configured DHCP server or in a network without any DHCP server.

This PXE currently serves:
- [MemTest86+](http://www.memtest86.com/)
- [Ubuntu Server 16.04](http://releases.ubuntu.com/16.04)

## Dependencies
These are the dependencies required to build and run the box:
- Docker 1.12+
- [Ubuntu Server 16.04 ISO](http://releases.ubuntu.com/16.04/ubuntu-16.04.2-server-amd64.iso)

## How to run
The `ENTRYPOINT` of this image is set to run `dnsmasq` in `no-daemon` mode. Add your desired `dhcp-range`s as command line options (see [dnsmasq documentation](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html) for details). Note that you can specify more than one range by adding multiple `dhcp-range` options.

The Ubuntu Server iso needs to be extracted and its root mounted as a volume in `/var/lib/tftpboot/ubuntu/16.04/16.04.2-server-amd64` when running the Docker container.

The easiest way to use instances of this image to provide a PXE in an existing network is to run a container based on it with the `--net=host` option.

To sum up: `docker run -it --rm --net=host -v /path/to/ubuntu-server/extracted/iso:/var/lib/tftpboot/ubuntu/16.04/16.04.2-server-amd64 ferrarimarco/pxe`

If you want to inspect the container just run it overwriting the entrypoint: `--entrypoint=/bin/bash`

### Integrated DHCP server
If you want to enable the integrated DHCP server for a given IP address range add a `dhcp-range` option: `dhcp-range=x.x.x.x,y.y.y.y,z.z.z.z` where `x.x.x.x` is the start of the range, `y.y.y.y` is the end and `z.z.z.z` is the subnet mask.

### Standalone DHCP server
If you want to use an existing DHCP server and let dnsmasq handle only the PXE, add a `dhcp-range` option: `dhcp-range=x.x.x.x,proxy` where `x.x.x.x` is the IP address of the server running dnsmasq.

## How to modify the configuration

All the configuration files can be modified at will. Just look at the Dockerfile to see where they are (mainly in `/etc` and `/var/lib/tftpboot`) and overwrite them with your own (mounting volumes from the Docker host or rebuilding the image).

## Testing and validating the setup
### Dependencies

1. Virtualbox 5.1.16+
1. Vagrant 1.9.3+

### How to run

After running the container with a suitable DHCP configuration (see above for instructions) and the `--net=host` option, you can run `vagrant up` from the root of the project. A Virtualbox VM (with a NATed network adapter) will boot from the given PXE.

Note that you should check the IP address ranged configured by the Virtualbox DHCP server (if enabled) and configure your `dhcp-range` and `/var/lib/tftpboot/pxelinux.cfg/default` accordingly.

#### Example

If Virtualbox DHCP server assigns addresses in the `192.168.56.0/24` subnet, then the `dhcp-range` could be: `--dhcp-range=192.168.56.2,proxy`, where `192.168.56.2` is the address assigned to the PXE server. Remember to also update the ip addresses in `/var/lib/tftpboot/pxelinux.cfg/default` if you serve any content from the TFTP server (like a `preseed.cfg` for example) to point to the ip address of the container running this PXE. **For this reason it could be useful to manually assign (or reserve) ip addresses for containers running this PXE.**

## Contributions
If you have suggestions, please create a new GitHub issue or a pull request.
