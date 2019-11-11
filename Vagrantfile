# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
  :box_name => "centos/7",
  :net => [ {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net"}, ]
  },
  :centralRouter => {
  :box_name => "centos/7",
  :net => [  {ip: '192.168.254.2', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net2"},
             {ip: '192.168.255.2', adapter: 3, netmask: "255.255.255.248", virtualbox__intnet: "router-net"},
             {ip: '192.168.20.1', adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "central-net"}, ]
  },
  :centralServer => {
  :box_name => "centos/7",
  :net => [  {ip: '192.168.20.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "central-net"}, ]
  },
  :inetRouter2 => {
    :box_name => "centos/7",
    :net => [ {ip: '192.168.254.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net2"},]
} }

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|
    box.vm.box = boxconfig[:box_name]
    box.vm.host_name = boxname.to_s
    boxconfig[:net].each do |ipconf|
    box.vm.network "private_network", ipconf
    end
    
    if boxconfig.key?(:public)
    box.vm.network "public_network", boxconfig[:public]
    end

    box.vm.provision "shell", inline: <<-SHELL
    mkdir -p ~root/.ssh
    cp ~vagrant/.ssh/auth* ~root/.ssh
    SHELL
        
    case boxname.to_s
    when "inetRouter"
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      sysctl net.ipv4.conf.all.forwarding=1
      iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
      ip route add 192.168.20.0/24 via 192.168.255.2
      ip route add 192.168.254.0/29 via 192.168.255.2
            # port knocking - 6699 9966 22 - 60 sec
            # разрешим соединения со статусом est и rel в conntracker
            iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
            # разрешим icmp
            iptables -A INPUT -p icmp --icmp-type any -j ACCEPT
            # создадим цепочку для port knocking
            iptables -N SSH_KNOCK
            # сделаем переход из input в цепочку для port knocking
            iptables -A INPUT -j SSH_KNOCK
            # заранее создадим цепочку для SSH SET
            iptables -N SSH_SET
            # если наш хост есть в списке SSH_STEP2 не более, чем 60 секунд, то пускаем...
            iptables -A SSH_KNOCK -m state --state NEW -m tcp -p tcp -m recent --rcheck --seconds 60 --dport 22 --name SSH_STEP2 -j ACCEPT
            # ... иначе удаляем его из списка (> 60 секунд)
            iptables -A SSH_KNOCK -m state --state NEW -m tcp -p tcp -m recent --name SSH_STEP2 --remove -j DROP
            # если наш хост стучался по порту 9966 и был в списке SSH_STEP1 ...
            iptables -A SSH_KNOCK -m state --state NEW -m tcp -p tcp -m recent --rcheck --dport 9966 --name SSH_STEP1 -j SSH_SET
            # ... то включить его в список SSH_STEP2
            iptables -A SSH_SET -m recent --set --name SSH_STEP2 -j DROP
            # ... иначе - удалить из списка SSH_STEP1
            iptables -A SSH_KNOCK -m state --state NEW -m tcp -p tcp -m recent --name SSH_STEP1 --remove -j DROP
            # если хост стучится по порту 6699, то добавить его в список SSH_STEP1
            iptables -A SSH_KNOCK -m state --state NEW -m tcp -p tcp -m recent --set --dport 6699 --name SSH_STEP1 -j DROP
            # по-умолчанию DROP внутри port knocking
            iptables -A SSH_KNOCK -j DROP
            # по-умолчанию DROP со стороны, откуда будет идти проверка
            iptables -A INPUT -i eth1 -j DROP
      SHELL
    when "centralRouter"
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      sysctl net.ipv4.conf.all.forwarding=1	
      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
      echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth2
      systemctl restart network
      sysctl -p
      ip route delete default 2>&1 >/dev/null || true
      ip route add default via 192.168.255.1
      SHELL
    when "centralServer"
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
      echo "GATEWAY=192.168.20.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
      systemctl restart network
      ip route delete default 2>&1 >/dev/null || true
      ip route add default via 192.168.20.1
      yum install -y epel-release
      yum install -y nginx
      systemctl enable nginx
      systemctl start nginx
     man # yum install -y nmap
      SHELL
    when "inetRouter2"
    config.vm.network "private_network", ip: "192.168.200.1"
    box.vm.network 'forwarded_port', guest: 8080, host: 8080, host_ip: '127.0.0.1'
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
      echo "GATEWAY=192.168.254.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
      systemctl restart network    
      ip route delete default 2>&1 >/dev/null || true
      ip route add 192.168.255.0/29 via 192.168.254.2
      ip route add 192.168.20.0/24 via 192.168.254.2
      sysctl net.ipv4.conf.all.forwarding=1
      # Сделаем проброс портов
      iptables -t nat -A PREROUTING --dst 10.0.2.15/32 -p tcp --dport 8080 -j DNAT --to-destination 192.168.20.2:80
      # И обратную подмену, т.к. маскарада у нас нет (дополнительное задание)
       iptables -t nat -A POSTROUTING --dst 192.168.20.2 -p tcp --dport 80 -j SNAT --to-source 192.168.254.1
       echo -e "192.168.254.2 centralRouter" >> /etc/hosts   
      SHELL
      end
     end
    end
   end


