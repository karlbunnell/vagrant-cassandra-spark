# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.host_name = "cassandra-spark"
  config.vm.network :private_network, ip: "192.168.10.11"
  config.vm.network :forwarded_port, guest: 22, host: 1233
  config.vm.network :forwarded_port, guest: 9092, host: 9092
  config.vm.network :forwarded_port, guest: 4040, host: 4040
  config.vm.network :forwarded_port, guest: 7077, host: 7077
  config.vm.network :forwarded_port, guest: 8080, host: 8080
  config.vm.network :forwarded_port, guest: 8081, host: 8081

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "playbook.yml"
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "8092"]
    vb.cpus = 4
  end
end
