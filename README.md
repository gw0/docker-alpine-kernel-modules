docker-alpine-kernel-modules
============================

***docker-alpine-kernel-modules*** can install missing [*Alpine Linux*](https://alpinelinux.org/) kernel modules from a privileged Docker container. Its main use case is to customize the native VM used in *Docker for Mac* or *Docker for Windows* (eg. to add USB over IP support).


Usage
=====

First build the Docker image for your kernel:

```bash
$ docker build --build-arg KERNELVER=$(uname -r  | cut -d '-' -f 1) -t alpine-kernel-modules .
```

To insert the kernel module for USB over IP support:

```
$ docker run -it --rm --privileged alpine-kernel-modules modprobe vhci-hcd
```

Otherwise you may configure your kernel, build, and insert modules using:

```bash
$ docker run -it --rm --privileged alpine-kernel-modules sh
/srv/linux-4.9.4 # make menuconfig
/srv/linux-4.9.4 # make modules_prepare
/srv/linux-4.9.4 # make modules
/srv/linux-4.9.4 # make modules_install
/srv/linux-4.9.4 # modprobe vhci-hcd
(or /srv/linux-4.9.4 # insmod ./drivers/usb/usbip/vhci-hcd.ko)
/srv/linux-4.9.4 # lsmod
Module                  Size  Used by    Not tainted
vhci_hcd               24576  0
usbip_core             16384  1 vhci_hcd
usbcore               159744  1 vhci_hcd
usb_common             16384  2 vhci_hcd,usbcore
```

For a permanent solution you would need to put the module in `/lib/modules` in your VM and configure `/etc/modules` accordingly. To enter the Docker VM on *Docker for Mac/Windows* use something like `docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh`.

An USB over IP solution for *Docker for Mac/Windows* that could work this way (untested) is [VirtualHere](https://www.virtualhere.com/). You run the *VirtualHere USB Server* on your host and *VirtualHere USB Client* in a privileged Docker container with mounted `/dev` (first as a service `./vhclientx86_64 -n` and then configure it to connect `./vhclientx86_64 -t 'MANUAL HUB ADD,192.168.65.1'`). A list of USB devices on the host should be visible with `./vhclientx86_64 -t 'LIST'`.


Feedback
========

If you encounter any bugs or have feature requests, please file them in the [issue tracker](http://github.com/gw0/docker-alpine-kernel-modules/issues/) or even develop it yourself and submit a pull request over [GitHub](http://github.com/gw0/docker-alpine-kernel-modules/).


License
=======

Copyright &copy; 2017 *gw0* [<http://gw.tnode.com/>] &lt;<gw.2017@ena.one>&gt;

This library is licensed under the [GNU Affero General Public License 3.0+](LICENSE_AGPL-3.0.txt) (AGPL-3.0+). Note that it is mandatory to make all modifications and complete source code of this library publicly available to any user.
