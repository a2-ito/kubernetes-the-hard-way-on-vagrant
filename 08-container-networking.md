# Networking
The procedures before doesn't include a virtual network configuration, which is CNI.
Therefore, containers deployed on a node cannot be connected to conatiners in other nodes without routing.
These following route configrations on the host would be required to connect containers on other nodes.
It would be the easiest way to configure container network among worker nodes.

## worker1
```
sudo route add -net 10.200.2.0 netmask 255.255.255.0 gw 192.168.33.112
```

## worker2
```
sudo route add -net 10.200.1.0 netmask 255.255.255.0 gw 192.168.33.111
```

## Script
```
for instance in 1 2
do
  case "${instance}" in
  1)
  ssh worker${instance} "\
    sudo sh -c \"route add -net 10.200.2.0 netmask 255.255.255.0 gw 192.168.33.112\"
  "
  ;;
  2)
  ssh worker${instance} "\
    sudo sh -c \"route add -net 10.200.1.0 netmask 255.255.255.0 gw 192.168.33.111\"
  "
  ;;
  *)
    ;;
  esac
done
```

## Option: permanently
```
for instance in 1 2
do
  case "${instance}" in
  1)
  ssh worker${instance} "\
    sudo sh -c \"route add -net 10.200.2.0 netmask 255.255.255.0 gw 192.168.33.112\"
    sudo sh -c \"echo up route add -net 10.200.2.0 netmask 255.255.255.0 gw 192.168.33.112 dev eth1 >> /etc/network/interfaces\"
  "
  ;;
  2)
  ssh worker${instance} "\
    sudo sh -c \"route add -net 10.200.1.0 netmask 255.255.255.0 gw 192.168.33.112\"
    sudo sh -c \"echo up route add -net 10.200.1.0 netmask 255.255.255.0 gw 192.168.33.111 dev eth1 >> /etc/network/interfaces\"
  "
  ;;
  *)
    ;;
  esac
done
```
