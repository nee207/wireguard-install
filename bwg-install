#!/bin/bash

#生成随机端口
rand(){
    min=$1
    max=$(($2-$min+1))
    num=$(cat /dev/urandom | head -n 10 | cksum | awk -F ' ' '{print $1}')
    echo $(($num%$max+$min))  
}

wireguard_update(){
    yum upgrade -y wireguard-dkms wireguard-tools
    echo "更新完成"
}

wireguard_remove(){
    wg-quick down wg0
    yum remove -y wireguard wireguard-dkms wireguard-tools
    rm -rf /etc/wireguard/
    echo "卸载完成"
}

config_client(){
cat > /etc/wireguard/client.conf <<-EOF
[Interface]
PrivateKey = $c1
Address = 10.10.10.2/24 
DNS = 8.8.8.8, 8.8.4.4
MTU = 1420

[Peer]
PublicKey = $s2
Endpoint = $serverip:$port
AllowedIPs = 0.0.0.0/0, ::0/0
PersistentKeepalive = 25
EOF

}

#安装wireguard
wireguard_install(){
    yum install -y wireguard wireguard-dkms wireguard-tools
    yum install -y qrencode
    mkdir /etc/wireguard
    cd /etc/wireguard
    wg genkey | tee sprivatekey | wg pubkey > spublickey
    wg genkey | tee cprivatekey | wg pubkey > cpublickey
    s1=$(cat sprivatekey)
    s2=$(cat spublickey)
    c1=$(cat cprivatekey)
    c2=$(cat cpublickey)
    serverip=$(curl ipv4.icanhazip.com)
    port=$(rand 10000 60000)
    eth=$(ls /sys/class/net | awk '/^e/{print}')
    chmod 777 -R /etc/wireguard
#    service stop firewalld
#    service disable firewalld
#    yum install -y iptables-services 
#    service enable iptables 
#    service start iptables 
#    iptables -P INPUT ACCEPT
#    iptables -P OUTPUT ACCEPT
#    iptables -P FORWARD ACCEPT
#    iptables -F
#    service iptables save
#    service iptables restart
#    echo 1 > /proc/sys/net/ipv4/ip_forward
#    echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
#    sysctl -p
cat > /etc/wireguard/wg0.conf <<-EOF
[Interface]
PrivateKey = $s1
Address = 10.10.10.1/24 
PostUp   = echo 1 > /proc/sys/net/ipv4/ip_forward; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o $eth -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o $eth -j MASQUERADE
ListenPort = $port
DNS = 8.8.8.8, 8.8.4.4
MTU = 1420
[Peer]
PublicKey = $c2
AllowedIPs = 10.10.10.2/32
EOF

    config_client
    wg-quick up wg0
    chkconfig wg-quick@wg0 on
    content=$(cat /etc/wireguard/client.conf)
    echo "电脑端请下载client.conf，手机端可直接使用软件扫码"
    echo "${content}" | qrencode -o - -t UTF8
}
add_user(){
    echo -e "\033[37;41m给新用户起个名字，不能和已有用户重复\033[0m"
    read -p "请输入用户名：" newname
    cd /etc/wireguard/
    cp client.conf $newname.conf
    wg genkey | tee temprikey | wg pubkey > tempubkey
    ipnum=$(grep Allowed /etc/wireguard/wg0.conf | tail -1 | awk -F '[ ./]' '{print $6}')
    newnum=$((10#${ipnum}+1))
    sed -i 's%^PrivateKey.*$%'"PrivateKey = $(cat temprikey)"'%' $newname.conf
    sed -i 's%^Address.*$%'"Address = 10.10.10.$newnum\/24"'%' $newname.conf

cat >> /etc/wireguard/wg0.conf <<-EOF
[Peer]
PublicKey = $(cat tempubkey)
AllowedIPs = 10.10.10.$newnum/32
EOF
    wg set wg0 peer $(cat tempubkey) allowed-ips 10.10.10.$newnum/32
    echo -e "\033[37;41m添加完成，文件：/etc/wireguard/$newname.conf\033[0m"
	content=$(cat /etc/wireguard/$newname.conf)
    	echo "${content}" | qrencode -o - -t UTF8
    rm -f temprikey tempubkey
}

#开始菜单
start_menu(){
    clear
    echo "========================="
    echo " Setup Wireguard"
    echo "========================="
    echo "1. 安装wireguard"
    echo "2. 升级wireguard"
    echo "3. 卸载wireguard"
    echo "4. 显示客户端二维码"
    echo "5. 增加用户"
    echo "0. 退出脚本"
    echo
    read -p "请输入数字:" num
    case "$num" in
    1)
	wireguard_install
	;;
	2)
	wireguard_update
	;;
	3)
	wireguard_remove
	;;
	4)
	content=$(cat /etc/wireguard/client.conf)
    	echo "${content}" | qrencode -o - -t UTF8
	;;
	5)
	add_user
	;;
	0)
	exit 1
	;;
	*)
	clear
	echo "请输入正确数字"
	sleep 5s
	start_menu
	;;
    esac
}

start_menu
