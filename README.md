#!/bin/bash
# scan_network.sh
#Below I present the Bash script, which finds the IP addresses of all devices on the network and scans them Using nmap, grouping open ports.

# Tarmoqdagi barcha IP-manzillarni topish
find_ips() {
    # Agar netdiscover yoki arp-scan o'rnatilgan bo'lsa, ular bilan tarmoq skaneri
    if command -v netdiscover &> /dev/null; then
        netdiscover -r 192.168.1.0/24 -P -N | grep '192.168.1.' | awk '{print $1}'
    elif command -v arp-scan &> /dev/null; then
        sudo arp-scan --interface=eth0 --localnet | grep '192.168.1.' | awk '{print $1}'
    else
        echo "netdiscover yoki arp-scan o'rnatilgan emas. Iltimos, ulardan birini o'rnating."
        exit 1
    fi
}

# Har bir IP manzilni nmap bilan skanerlash
scan_ports() {
    ip=$1
    echo "Skanerlanmoqda: $ip"
    open_ports=$(nmap -p- --open $ip | grep '/tcp' | awk '{print $1}' | tr '\n' ' ')
    if [ -n "$open_ports" ]; then
        echo "$ip: $open_ports"
    else
        echo "$ip: ochiq portlar topilmadi"
    fi
}

# IP-manzillarni topish
echo "Tarmoqdagi barcha IP-manzillarni topish..."
ips=$(find_ips)

# IP-manzillarni skanerlash va ochiq portlarni ko'rsatish
echo "Ochiq portlarni skanerlash va ko'rsatish..."
for ip in $ips; do
    scan_ports $ip
done
