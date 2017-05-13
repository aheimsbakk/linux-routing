# -*- mode: ruby -*-
# vi: set ft=ruby :

router_script="""
echo *** Install UCARP
apt-get install -y ucarp
echo *** Create vip up and down scripts
echo -e '#!/bin/bash'\\\\n'ip addr add $2/32 dev $1 2>/dev/null'\\\\n'exit 0' > /etc/vip-up.sh
echo -e '#!/bin/bash'\\\\n'ip addr del $2/32 dev $1 2>/dev/null'\\\\n'exit 0' > /etc/vip-down.sh
chmod +x /etc/vip-up.sh /etc/vip-down.sh
echo *** Enable forwarding 
sysctl -w net.ipv4.ip_forward=1
"""

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.box_check_update = true

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = 1
  end

  # Workaround for "Inappropriate ioctl..."
  # https://github.com/mitchellh/vagrant/issues/7368
  config.vm.provision :shell, run: :always, inline: "(grep -q 'mesg n' /root/.profile && sed -i '/mesg n/d' /root/.profile && echo 'Ignore the previous error, fixing this now...') || exit 0;"


  # Outside to mimic a network configured with a satic route.
  # "Network" address space in 10.0.0.0/24.
  config.vm.define :outside do |c|
    c.vm.hostname = :outside
    c.vm.network "private_network", ip: "10.0.0.10"
    c.vm.provision :shell, run: :always, inline: <<-SHELL
      echo *** Mimic network with static route by adding static route
      echo *** to 172.162.0.0/24 through «router1» and «router2»
      ip route replace 172.16.0.0/24 nexthop via 10.0.0.2 nexthop via 10.0.0.3 2> /dev/null || exit 0
    SHELL
  end

  # Linux box to act as router1
  # Just ip_forward enabled.
  config.vm.define :router1 do |c|
    c.vm.hostname = :router1
    c.vm.network "private_network", ip: "10.0.0.2"
    c.vm.network "private_network", ip: "172.16.0.2"
    c.vm.provision :shell, run: :always, inline: router_script
    c.vm.provision :shell, run: :always, inline: <<-SHELL
      echo *** Start UCARP
      pgrep ucarp || ucarp -i enp0s9 -s 172.16.0.2 -a 172.16.0.1 -p router -v 1 -u /etc/vip-up.sh -d /etc/vip-down.sh -B
    SHELL
  end

  # Linux box to act as router2
  # Just ip_forward enabled.
  config.vm.define :router2 do |c|
    c.vm.hostname = :router2
    c.vm.network "private_network", ip: "10.0.0.3"
    c.vm.network "private_network", ip: "172.16.0.3"
    c.vm.provision :shell, run: :always, inline: router_script
    c.vm.provision :shell, run: :always, inline: <<-SHELL
      echo *** Start UCARP
      pgrep ucarp || ucarp -i enp0s9 -s 172.16.0.3 -a 172.16.0.1 -p router -v 1 -u /etc/vip-up.sh -d /etc/vip-down.sh -B
    SHELL
  end

  # Inside with default gateway to router.
  config.vm.define :inside do |c|
    c.vm.hostname = :inside
    c.vm.network "private_network", ip: "172.16.0.10"
    c.vm.provision :shell, run: :always, inline: <<-SHELL
      echo *** Change default route to «router» 172.16.0.1
      ip route replace default via 172.16.0.1
      echo *** Ping address on «outside» 10.0.0.10
      ping -c 10 10.0.0.10
   SHELL
  end
end
