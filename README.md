# Linux network namespace

Create two namespace called red and green
```
sudo ip netns add green 
sudo ip netns add red
```
Create a bridge and up it
```
sudo ip link add br0 type bridge
sudo ip link set dev br0 up
```
Install all the packages
```
sudo apt install bridge-utils
sudo apt install iptables
sudo apt install net-tools
sudo apt install iproute2
sudo apt install iputils-ping
```
Bridge Information:
```
sudo brctl show
```
Now we need two link for connect them with bridge:
```
sudo ip link add veth-green type veth peer name veth-green-br0
sudo ip link add verh-red type veth peer name veth-red-br0
```
we already created two link now connect them with namespace and bridge

for red namespace
```
sudo ip link set veth-red netns red
sudo ip link set veth-red-br0 master br0
```
for green namespace
```
sudo ip link set veth-green netns green
sudo ip link set veth-green-br0 master br0
```
Now assign ip addresses to namespace interfaces and up them
For red
```
sudo ip netns exec red bash
ip addr add 192.168.10.1/24 dev veth-red
ip link set veth-red up
```

```
sudo ip netns exec green bash
ip addr add 192.168.10.2/24 dev veth-green
ip link set veth-green up
```
Now also up the interfaces that is connected to bridge:
```
sudo ip link set veth-red-br0 up
sudo ip link set veth-prod-br0 up
```
Now ping from both side:
```
sudo ip netns exec red ping 192.168.10.2
sudo ip netns exec green ping 192.168.10.1
```
Now ping is working but need to connect the root,
to connet the root need a gateway and here bridge will work as gateway;

lets add ip to the bridge maintaining subnet mask rules
```
sudo ip addr add 192.168.10.10/24 dev br0
```
Now lets go to red and add rules in route table
```
sudo ip route add default via 192.168.10.10
```
ping and see wah now its connect to the bridge and host.
ping from host , its getting data, and ping from green/red what happening? showing
network is unreachable.

so now we need snat because private network cant connect with private

add this at the host machine
```
sudo iptables --table nat -A POSTROUTING -s 192.168.10.0/24 -j MASQUERADE
```
lets ping

