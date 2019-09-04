# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
  :box_name => "centos/6",
  :net => [ {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net"}, ]
  },
  :centralRouter => {
  :box_name => "centos/7",
  :net => [  {ip: '192.168.254.2', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "router-net2"},
             {ip: '192.168.255.2', adapter: 3, netmask: "255.255.255.248", virtualbox__intnet: "router-net"},
             {ip: '192.168.0.1', adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "central-net"}, ]
  },
  :centralServer => {
  :box_name => "centos/7",
  :net => [  {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "central-net"}, ]
  },
  :inetRouter2 => {
    :box_name => "centos/6",
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
      ip route add 192.168.0.0/24 via 192.168.255.2
      ip route add 192.168.254.0/29 via 192.168.255.2
      SHELL
    when "centralRouter"
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      sysctl net.ipv4.conf.all.forwarding=1
      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
      echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth2
      systemctl restart network
      ip route delete default 2>&1 >/dev/null || true
      ip route add default via 192.168.255.1
      SHELL
    when "centralServer"
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
      echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
      systemctl restart network
      ip route delete default 2>&1 >/dev/null || true
      ip route add default via 192.168.0.1
      SHELL
    when "inetRouter2"
    box.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
      echo "GATEWAY=192.168.254.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
      systemctl restart network    
      ip route delete default 2>&1 >/dev/null || true
      ip route add 0.0.0.0/0 via 192.168.254.2
      SHELL
      end
     end
    end
   end


