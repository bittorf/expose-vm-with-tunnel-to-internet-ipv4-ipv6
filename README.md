### idea

Use public IP-address (v4/v6) from host in datacenter  
and tunnel explicit ports using wireguard into VM running anywhere (e.g. behind NAT).
```
e.g.
                  SIDE A (IPv4 + IPv6)                           SIDE B
                 +--------------------+                 +---------------------+
TCP-22/80/443 ===> host in datacenter <=== wireguard ===> VM running anywhere |
                 +--------------------+                 +---------------------+
```

### Step0: Side-B (e.g. behind NAT) setup a VM which supports wireguard
```
qemu-img create -f qcow2 test.img 16G
URL=https://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-cd/debian-testing-amd64-netinst.iso
wget -O ISO "$URL"
qemu-system-x86_64 -enable-kvm -m 4G -hda test.img -boot d -cdrom ISO	# do minimal install
qemu-system-x86_64 -enable-kvm -m 4G -hda test.img			# for setup
qemu-system-x86_64 -enable-kvm -m 4G -hda test.img -nographic		# later: headless start
```

### Step1: Side-A (with public reachable IP)
```
apt install wireguard
mkdir -p /etc/boot.d && cd /etc/boot.d
URL="https://raw.githubusercontent.com/bittorf/expose-vm-with-tunnel-to-internet-ipv4-ipv6/main/99-tunnel"
wget "$URL" && chmod +x 99-tunnel && /etc/boot.d/99-tunnel setupA
```

### Step2: Side-B (e.g. behind NAT)
```
apt install wireguard
mkdir -p /etc/boot.d && cd /etc/boot.d
URL="https://raw.githubusercontent.com/bittorf/expose-vm-with-tunnel-to-internet-ipv4-ipv6/main/99-tunnel"
wget "$URL" && chmod +x 99-tunnel && /etc/boot.d/99-tunnel setupB
```

### ToDo
* maybe add to https://github.com/anderspitman/awesome-tunneling
* make it work with: https://github.com/WireGuard/wireguard-go
* make it work with: https://github.com/cloudflare/boringtun
* make it work with: userspace UML linux: https://github.com/bittorf/kritis-linux

