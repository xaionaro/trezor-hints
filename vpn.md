server:
```
apt-get install -y uml-utilities bridge-utils ifupdown

cat >/etc/network/interfaces.d/sshvpn0 <<EOF
auto  tap8
iface tap8 inet manual
	pre-up tunctl -u user -t $IFACE
	post-down tunctl -d $IFACE
	up ifconfig $IFACE 0.0.0.0 up
	down ifconfig $IFACE down

auto  sshvpn0
iface sshvpn0 inet static
	address 10.0.0.1
	netmask 255.255.255.0
	bridge-stp off
	bridge-ports tap8
	mtu 1300

ifup tap8
ifup sshvpn0
```

client:
```
apt-get install -y uml-utilities bridge-utils ifupdown

cat >/etc/network/interfaces.d/sshvpn0 <<EOF
auto tap8
iface tap8 inet manual
	pre-up tunctl -u user -t $IFACE
	post-down tunctl -d $IFACE
	up ifconfig $IFACE 0.0.0.0 up
	down ifconfig $IFACE down

auto sshvpn0
iface sshvpn0 inet static
	address 10.0.0.2
	netmask 255.255.255.0
	bridge-stp off
	bridge-ports tap8
	mtu 1300
EOF

cat >~/.ssh/config <<EOF
Host vpn.example.com
	Tunnel=Ethernet
	TunnelDevice=8:8
EOF

ifup tap8
ifup sshvpn0
trezor-agent -e ed25519 -c vpn.example.com user@machine
```
