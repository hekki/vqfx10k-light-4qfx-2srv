# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.ssh.insert_key = false

    ##########################
    ## Routing Engine  #######
    ##########################
    # topology: https://www.juniper.net/documentation/en_US/junos/topics/topic-map/bgp-local-preference.html
    # LineNumber:
    ## R1<->R2: 1
    ## R1<->R3: 2
    ## R2<->R4: 3
    ## R3<->R4: 4
    ## R1<->srv1: server_vqfx1
    ## R4<->srv2: server_vqfx1

    # vqfx1..4 common provision
    (1..4).each do |id|
        re_name  = ( "vqfx" + id.to_s ).to_sym

        config.vm.define re_name do |vqfx|
          vqfx.vm.hostname = "vqfx#{id}"
          vqfx.vm.box = 'juniper/vqfx10k-re'
          vqfx.vm.synced_folder '.', '/vagrant', disabled: true

          # Management port (em1 / em2)
          vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "vqfx_internal_#{id}"
          vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "reserved-bridge"
        end
    end

    # vqfx1 provision
    config.vm.define 'vqfx1' do |vqfx|
      # (em3, em4)
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg1'
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg2'

      # Dataplane ports (em5)
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'server_vqfx1'

    end

    # vqfx2 provision
    config.vm.define 'vqfx2' do |vqfx|
      # (em3, em4)
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg1'
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg3'
    end

    # vqfx3 provision
    config.vm.define 'vqfx3' do |vqfx|
      # (em3, em4)
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg2'
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg4'
    end

    # vqfx4 provision
    config.vm.define 'vqfx4' do |vqfx|
      # (em3, em4)
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg4'
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'seg6'

      # Dataplane ports (em5)
      vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: 'server_vqfx2'
    end

    ##########################
    ## Server          #######
    ##########################
    (1..2).each do |id|
      srv_name  = ( "srv" + id.to_s ).to_sym

      config.vm.define srv_name do |srv|
        srv.vm.box = "robwc/minitrusty64"
        srv.vm.hostname = "server#{id}"
        srv.vm.network 'private_network', ip: "10.10.#{id}.2", virtualbox__intnet: "server_vqfx#{id}"
        srv.ssh.insert_key = true
        srv.vm.provision "shell",
        inline: "sudo route add -net 10.10.0.0 netmask 255.255.0.0 gw 10.10.#{id}.1"
        end
    end

    ##############################
    ## Box provisioning    #######
    ##############################
    if !Vagrant::Util::Platform.windows?
        config.vm.provision "ansible" do |ansible|
            ansible.groups = {
                "vqfx10k" => ["vqfx1", "vqfx2", "vqfx3", "vqfx4"],
                "server"  => ["server1", "server2" ],
                "all:children" => ["vqfx10k", "server"]
            }
            ansible.playbook = "provisioning/deploy-config.p.yaml"
            ansible.compatibility_mode = "2.0"
        end
    end
end
