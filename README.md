# Linux Containers from scratch (container-scratch)

Learning *Kernel* Linux Containers from scratch

May be better known as operating system level virtualization (oslv)

## Sample network namespace

```bash
export CONTAINER_NAME=ex1
export BRIDGE=scratchbr0
export IP_ADDR=192.168.0.7/24
export GATEWAY=192.168.0.1
export CONTAINER_ROOTFS="$HOME/$CONTAINER_NAME/"
```

```bash
cat <<EOF > /etc/sysconfig/network-scripts/ifcfg-$BRIDGE
NAME=$BRIDGE
IPADDR=$GATEWAY
NETMASK=255.255.255.0
TYPE=Bridge
BOOTPROTO=none
DEVICE=$BRIDGE
NM_MANAGED=no
ONBOOT=yes
EOF

ifup $BRIDGE && ip addr show $BRIDGE

```

### Container *shell*

```bash
ip netns add $(CONTAINER_NAME)
ip link add oslv-$(CONTAINER_NAME) type veth peer name xoslv-$(CONTAINER_NAME)
ip link set xoslv-$(CONTAINER_NAME) netns $(CONTAINER_NAME)
brctl addif $(BRIDGE) oslv-$(CONTAINER_NAME)
ip link set oslv-$(CONTAINER_NAME) up
ip netns exec $(CONTAINER_NAME) ip link set xoslv-$(CONTAINER_NAME) name eth0
ip netns exec $(CONTAINER_NAME) ip link set eth0 up
ip netns exec $(CONTAINER_NAME) ip a add $(IP_ADDR) dev eth0
ip netns exec $(CONTAINER_NAME) ip r add default via $(GATEWAY)
ip netns exec $(CONTAINER_NAME) bash
```

### Container *boot*

```bash
ip netns add $(CONTAINER_NAME)
ip link add oslv-$(CONTAINER_NAME) type veth peer name xoslv-$(CONTAINER_NAME)
ip link set xoslv-$(CONTAINER_NAME) netns $(CONTAINER_NAME)
brctl addif $(BRIDGE) oslv-$(CONTAINER_NAME)
ip link set oslv-$(CONTAINER_NAME) up
ip netns exec $(CONTAINER_NAME) ip link set xoslv-$(CONTAINER_NAME) name eth0
ip netns exec $(CONTAINER_NAME) ip link set eth0 up
ip netns exec $(CONTAINER_NAME) ip a add $(IP_ADDR) dev eth0
ip netns exec $(CONTAINER_NAME) ip r add default via $(GATEWAY)
ip netns exec $(CONTAINER_NAME) systemd-nspawn -M $(CONTAINER_NAME) -D $(CONTAINER_ROOTFS) -b #|| true
ip netns del $(CONTAINER_NAME)
```

# References

Linux Kernel Networking Implementation and Theory. Rosen, Rami. 2014

Namespaces in operation, part 1: namespaces overview. Kerrisk, Michael. part 1 - 7. January 4, 2013 at <http://lwn.net/Articles/531114/>
