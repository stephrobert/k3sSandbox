# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  base_ip_str = "10.240.0.1"
  cpu_master = 1
  mem_master = 1792
  number_workers = 2 # Number of workers nodes kubernetes
  cpu_worker = 1
  mem_worker = 1024
  config.vm.box = "generic/ubuntu2204" # Image for all installations
  kubectl_version = "1.28.4-00"

  nodes = []
  (0..number_workers).each do |i|
    case i
      when 0
        nodes[i] = {
          "name" => "master#{i + 1}",
          "ip" => "#{base_ip_str}#{i}"
        }
      when 1..number_workers
        nodes[i] = {
          "name" => "worker#{i }",
          "ip" => "#{base_ip_str}#{i}"
        }
    end
  end

# Provision VM
  nodes.each do |node|
    config.vm.define node["name"] do |machine|
      machine.vm.hostname = node["name"]
      machine.vm.provider "libvirt" do |lv|
        if (node["name"] =~ /master/)
          lv.cpus = cpu_master
          lv.memory = mem_master
        else
          lv.cpus = cpu_worker
          lv.memory = mem_worker
        end
      end
      machine.vm.synced_folder '.', '/vagrant', disabled: true
      machine.vm.network "private_network", ip: node["ip"]
      machine.vm.provision "ansible" do |ansible|
        ansible.playbook = "provision.yml"
        ansible.groups = {
          "master" => ["master1"],
          "workers" => ["worker[1:#{number_workers}]"],
          "kubernetes:children" => ["masters", "workers"],
          "all:vars" => {
            "base_ip_str" => "#{base_ip_str}",
            "kubectl_version" => "#{kubectl_version}"
          }
        }
      end
    end
  end
  config.push.define "local-exec" do |push|
    push.inline = <<-SCRIPT
      ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory master.yml
      ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory slave.yml
    SCRIPT
  end
end


