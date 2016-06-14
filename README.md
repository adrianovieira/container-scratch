# Linux Containers from scratch (container-scratch)

Learning Linux Containers from scratch

May be better known as operating system elvel virtualization (oslv)

## Sample network namespace

```bash
export CONTAINER_NAME=ex1
export BRIDGE=scratchbr0
export IP_ADDR=192.168.0.7/24
export GATEWAY=192.168.0.1
```

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
ip netns exec $(CONTAINER_NAME) systemd-nspawn -M $(CONTAINER_NAME) -D $(ROOTFS) -b #|| true
ip netns del $(CONTAINER_NAME)
```
