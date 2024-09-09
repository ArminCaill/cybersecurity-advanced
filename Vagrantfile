
HOST_ONLY_NETWORK = "VirtualBox Host-Only Ethernet Adapter #2"
#HOST_ONLY_NETWORK = "vboxnet1" # Typically on Linux/Mac
#HOST_ONLY_NETWORK = "VirtualBox Host-Only Ethernet Adapter #2" # Typically on Windows


Vagrant.configure("2") do |config|
    config.vm.define "companyrouter" do |host|
        host.vm.box = "almalinux/9"
        host.vm.hostname = "companyrouter"

        host.vm.network "private_network", ip: "192.168.62.253", netmask: "255.255.255.0", name: HOST_ONLY_NETWORK
        host.vm.network "private_network", ip: "172.30.255.254", netmask: "255.255.0.0", virtualbox__intnet: "internal-company-lan"

        host.vm.provider :virtualbox do |v|
            v.name = "companyrouter"
            v.cpus = "1"
            v.memory = "1024"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Default gateway
            nmcli connection modify "System eth1" ipv4.gateway 192.168.62.254
            nmcli connection down "System eth1" && nmcli connection up "System eth1"
        SHELL
    end

    config.vm.define "dns" do |host|
        host.vm.box = "almalinux/9"
        host.vm.hostname = "dns"

        host.vm.network "private_network", ip: "172.30.0.4", netmask: "255.255.255.0", virtualbox__intnet: "internal-company-lan"

        host.vm.provider :virtualbox do |v|
            v.name = "dns"
            v.cpus = "1"
            v.memory = "1024"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Default gateway
            nmcli connection modify "System eth1" ipv4.gateway 172.30.255.254
            nmcli connection down "System eth1" && nmcli connection up "System eth1"
        SHELL
    end

    config.vm.define "web" do |host|
        host.vm.box = "almalinux/9"
        host.vm.hostname = "web"

        host.vm.network "private_network", ip: "172.30.0.10", netmask: "255.255.255.0", virtualbox__intnet: "internal-company-lan"

        host.vm.provider :virtualbox do |v|
            v.name = "web"
            v.cpus = "1"
            v.memory = "1024"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Default gateway
            nmcli connection modify "System eth1" ipv4.gateway 172.30.255.254
            nmcli connection down "System eth1" && nmcli connection up "System eth1"
        SHELL
    end

    config.vm.define "database" do |host|
        host.vm.box = "almalinux/9"
        host.vm.hostname = "database"

        host.vm.network "private_network", ip: "172.30.0.15", netmask: "255.255.255.0", virtualbox__intnet: "internal_network"

        host.vm.provider :virtualbox do |v|
            v.name = "database"
            v.cpus = "1"
            v.memory = "1024"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Default gateway
            nmcli connection modify "System eth1" ipv4.gateway 172.30.255.254
            nmcli connection down "System eth1" && nmcli connection up "System eth1"
        SHELL
    end

    config.vm.define "isprouter" do |host|
        host.vm.box = "generic/alpine318"
        host.vm.hostname = "isprouter"

        host.vm.network "private_network", ip: "192.168.62.254", netmask: "255.255.255.0", name: HOST_ONLY_NETWORK

        host.vm.provider :virtualbox do |v|
            v.name = "isprouter"
            v.cpus = "1"
            v.memory = "256"
        end

        host.vm.provision "shell", inline: <<-SHELL
            apk --no-cache add python3 # For ansible
        SHELL
    end

    config.vm.define "red" do |host|
        host.vm.box = "kalilinux/rolling"
        host.vm.hostname = "red"

        host.vm.network "private_network", ip: "192.168.62.66", netmask: "255.255.255.0", name: HOST_ONLY_NETWORK

        host.vm.provider :virtualbox do |v|
            v.name = "red"
            v.cpus = "1"
            v.memory = "2048"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Ansible controller node
            apt-get update
            apt-get install --assume-yes ansible

            # Default gateway
            echo "gateway 192.168.62.254" >> /etc/network/interfaces
        SHELL
    end

    config.vm.define "homerouter" do |host|
        host.vm.box = "almalinux/9"
        host.vm.hostname = "homerouter"

        host.vm.network "private_network", ip: "192.168.62.42", netmask: "255.255.255.0", name: HOST_ONLY_NETWORK
        host.vm.network "private_network", ip: "172.10.10.254", netmask: "255.255.255.0", virtualbox__intnet: "employee-home-lan"

        host.vm.provider :virtualbox do |v|
            v.name = "homerouter"
            v.cpus = "1"
            v.memory = "1024"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Default gateway
            nmcli connection modify "System eth1" ipv4.gateway 192.168.62.254
            nmcli connection down "System eth1" && nmcli connection up "System eth1"
        SHELL
    end

    config.vm.define "employee" do |host|
        host.vm.box = "almalinux/9"
        host.vm.hostname = "employee"

        host.vm.network "private_network", ip: "172.10.10.123", netmask: "255.255.255.0", virtualbox__intnet: "employee-home-lan"

        host.vm.provider :virtualbox do |v|
            v.name = "employee"
            v.cpus = "1"
            v.memory = "1024"
        end

        host.vm.provision "shell", inline: <<-SHELL
            # Default gateway
            nmcli connection modify "System eth1" ipv4.gateway 172.10.10.254
            nmcli connection down "System eth1" && nmcli connection up "System eth1"
        SHELL
    end
end
