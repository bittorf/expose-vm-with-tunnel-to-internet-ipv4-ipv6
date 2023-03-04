=== idea ===

Use public IP-address (v4/v6) from host in datacenter  
and tunnel explicit ports using wireguard into VM running anywhere.

```
e.g.
                  SIDE A (IPv4 + IPv6)                           SIDE B
                 +--------------------+                 +---------------------+
TCP-22/80/443 ===> host in datacenter <=== wireguard ===> VM running anywhere |
                 +--------------------+                 +---------------------+
```

ToDo:  
https://github.com/anderspitman/awesome-tunneling


=== Step0: Side-B setup VM ===

qemu-img create -f qcow2 test.img 16G
URL=https://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-cd/debian-testing-amd64-netinst.iso
wget -O ISO "$URL"
qemu-system-x86_64 -enable-kvm -m 4G -hda test.img -boot d -cdrom ISO	# do minimal install
qemu-system-x86_64 -enable-kvm -m 4G -hda test.img			# for setup
qemu-system-x86_64 -enable-kvm -m 4G -hda test.img -nographic		# later: headless start


=== Step1: Side-A (with public reachable IP) ===

apt install wireguard
mkdir -p /etc/boot.d && cd /etc/boot.d
wget -O 99-tunnel https://setup.sh && chmod +x 99-tunnel
/etc/boot.d/99-tunnel setupA

=== Step3: Side-B (e.g. behind NAT) ===

apt install wireguard
mkdir -p /etc/boot.d && cd /etc/boot.d
wget -O 99-tunnel https://setup.sh && chmod +x 99-tunnel
/etc/boot.d/99-tunnel setupB

=== TODO: configure portforwardings on side-A ===

# netstat -tulpn | grep 0.0.0.0:[0-9] | awk '/^tcp/ {print $4}' | cut -d':' -f2 | sort -n | xargs
#        => 22 25 80 443 465 587 993 995 4190
for PORT in 22 25 80 443 465 587 993 995 4190; do
  # forward needed ports from outside to inside:
  [ $PORT = 22 ] || iptables  -t nat -I PREROUTING -i eth0 -p tcp --dport "$PORT" -j DNAT --to-destination 192.168.10.2
  [ $PORT = 22 ] || ip6tables -t nat -I PREROUTING -i eth0 -p tcp --dport "$PORT" -j DNAT --to-destination fc00::2
  # redirect from inside to outside:
  # ip6tables -t nat -I PREROUTING         -p tcp --dport "$PORT" -s fc00::2 -d $PUBIP6 -j DNAT --to-destination fc00::2
done
