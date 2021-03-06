Vagrant.configure("2") do |config|
  config.ssh.username = "vagrant"
  config.ssh.insert_key = false

  config.vm.define :installer1 do |uosi|
    uosi.vm.hostname = :installer1
    uosi.vm.box = "generic/ubuntu2004"
    uosi.vm.provider :virtualbox do |virthost|
      virthost.customize ["modifyvm", :id, "--nictype1", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic2", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype2", "virtio"]
      virthost.customize ["modifyvm", :id, "--nictype3", "virtio"]
      virthost.memory = 2048
      virthost.cpus = 2
    end

    uosi.vm.provider :libvirt do |virthost|
      virthost.memory = 2048
      virthost.cpus = 2
      virthost.driver = 'kvm'
      virthost.disk_bus = 'scsi'
      #virthost.disk_driver_opts :cache => "writeback", :io => "native", :discard => "unmap"
    end

    # Mgmt openstack vlan with multiples id (10 / 20), for internal flux like db, storage, messaging, ...
    uosi.vm.network :private_network, libvirt__network_name: "os_mgmt", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.80.20", netmask: "255.255.255.128", virtualbox__intnet: "os_mgmt"
    # Default input ip, used mainly for ansible reachablity
    uosi.vm.network :private_network, ip: "192.168.95.20", netmask: "255.255.255.128"
  end

  config.vm.define :infra1 do |uosc|
    uosc.vm.hostname = :infra1
    uosc.vm.box = "generic/ubuntu2004"
    uosc.vm.provider :virtualbox do |virthost|
      # Virtio adapter for all eth interfaces and ensure internal networks for openstack vlans
      virthost.customize ["modifyvm", :id, "--nictype1", "virtio"]
      virthost.customize ["modifyvm", :id, "--natsettings1", "1400,128,128,128,128"]
      virthost.customize ["modifyvm", :id, "--nic2", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype2", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic3", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype3", "virtio"]
      virthost.customize ["modifyvm", :id, "--nictype4", "virtio"]
      # interfaces 2 and 3 are vlan-bridges, need promiscious mode enabled (because virtualbox dont transmit vlan id between vm)
      virthost.customize ["modifyvm", :id, "--nicpromisc2", "allow-vms"]
      virthost.customize ["modifyvm", :id, "--nicpromisc3", "allow-vms"]

      # RAM / CPU
      virthost.memory = 16384
      virthost.cpus = 4

      # lxc containers filesystem
      unless File.exist?("/usr/local/vdis/os/disk_lxc.vdi")
        virthost.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata", "--controller", "IntelAhci", "--portcount", "2", "--bootable", "off", "--hostiocache", "on"]
        virthost.customize ["createmedium", "--filename", "/usr/local/vdis/os/disk_lxc.vdi", "--size", "53600", "--variant", "Fixed"]
      end
      virthost.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 0, "--device", 0, "--type", "hdd", "--medium", "/usr/local/vdis/os/disk_lxc.vdi", "--nonrotational", "on", "--discard", "on"]
     end

    uosc.vm.provider :libvirt do |virthost|
      virthost.memory = 16384
      virthost.cpus = 4
      virthost.driver = 'kvm'
      virthost.disk_bus = 'scsi'
      #virthost.disk_driver_opts :cache => "writeback", :io => "native", :discard => "unmap"
      virthost.storage :file, size: '150G', type: 'qcow2', path: 'disk_lxc.img', bus: "scsi", device: "sdb", cache: "writeback", io: "native", discard: "unmap"
    end

    # Mgmt openstack vlan with multiples id (10 / 20), for internal flux like db, storage, messaging, ...
    uosc.vm.network :private_network, libvirt__network_name: "os_mgmt", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.80.70", netmask: "255.255.255.128", virtualbox__intnet: "os_mgmt"
    # Neutron network vlan for openstack network stacks management
    uosc.vm.network :private_network, libvirt__network_name: "os_net", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.85.70", netmask: "255.255.255.128", virtualbox__intnet: "os_net", auto_config: false
    # Default input ip, used mainly for ansible reachablity
    uosc.vm.network :private_network, ip: "192.168.95.70", netmask: "255.255.255.128"
  end

  config.vm.define :compute1 do |uosnv|
    uosnv.vm.hostname = :compute1
    uosnv.vm.box = "generic/ubuntu2004"
    uosnv.vm.provider :virtualbox do |virthost|
      # Virtio adapter for all eth interfaces and ensure internal networks for openstack vlans
      virthost.customize ["modifyvm", :id, "--nictype1", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic2", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype2", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic3", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype3", "virtio"]
      virthost.customize ["modifyvm", :id, "--nictype4", "virtio"]

      # interface 3 is vlan-bridges, need promiscious mode enabled (because virtualbox dont transmit vlan id between vm)
      virthost.customize ["modifyvm", :id, "--nicpromisc3", "allow-vms"]

      # disable dns server from host
      virthost.auto_nat_dns_proxy = false

      # RAM / CPU
      virthost.memory = 20480
      virthost.cpus = 6

      # Nova libvirt filesystem
      unless File.exist?("/usr/local/vdis/os/disk_nova1.vdi")
        virthost.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata", "--controller", "IntelAhci", "--portcount", "2", "--bootable", "off", "--hostiocache", "on"]
        virthost.customize ["createmedium", "--filename", "/usr/local/vdis/os/disk_nova1.vdi", "--size", "409600", "--variant", "Standard"]
      end
      virthost.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 0, "--device", 0, "--type", "hdd", "--medium", "/usr/local/vdis/os/disk_nova1.vdi", "--nonrotational", "on", "--discard", "on"]

      # disable nested virtualization (not still usable, too much instability)
      virthost.customize ["modifyvm", :id, "--nested-hw-virt", "off"]
    end

    uosnv.vm.provider :libvirt do |virthost|
      virthost.memory = 20480
      virthost.cpus = 6
      virthost.nested = true
      virthost.driver = 'kvm'
      virthost.cpu_mode = "host-model"
      virthost.cpu_feature :name => 'vmx', :policy => 'require'
      virthost.disk_bus = 'scsi'
      #virthost.disk_driver_opts :cache => "writeback", :io => "native", :discard => "unmap"
      virthost.storage :file, size: '400G', type: 'qcow2', path: 'disk_nova1.img', bus: "scsi", device: "sdb", cache: "writeback", io: "native", discard: "unmap"
    end

    # Mgmt openstack vlan with multiples id (10 / 20), for internal flux like db, storage, messaging, ...
    uosnv.vm.network :private_network, libvirt__network_name: "os_mgmt", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.80.75", netmask: "255.255.255.128", virtualbox__intnet: "os_mgmt"
    # Neutron network vlan for openstack network stacks management
    uosnv.vm.network :private_network, libvirt__network_name: "os_net", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.85.75", netmask: "255.255.255.128", virtualbox__intnet: "os_net", auto_config: false

    # Default input ip, used mainly for ansible reachablity
    uosnv.vm.network :private_network, :ip => "192.168.95.75", :netmask => "255.255.255.128"
  end

  config.vm.define :compute2 do |uosnv2|
    uosnv2.vm.hostname = :compute2
    uosnv2.vm.box = "generic/ubuntu2004"
    uosnv2.vm.provider :virtualbox do |virthost|
      # Virtio adapter for all eth interfaces and ensure internal networks for openstack vlans
      virthost.customize ["modifyvm", :id, "--nictype1", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic2", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype2", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic3", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype3", "virtio"]
      virthost.customize ["modifyvm", :id, "--nictype4", "virtio"]

      # interface 3 is vlan-bridges, need promiscious mode enabled (because virtualbox dont transmit vlan id between vm)
      virthost.customize ["modifyvm", :id, "--nicpromisc3", "allow-vms"]

      # disable dns server from host
      virthost.auto_nat_dns_proxy = false

      # RAM / CPU
      virthost.memory = 20480
      virthost.cpus = 6

      # Nova libvirt filesystem
      unless File.exist?("/usr/local/vdis/os/disk_nova2.vdi")
        virthost.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata", "--controller", "IntelAhci", "--portcount", "2", "--bootable", "off", "--hostiocache", "on"]
        virthost.customize ["createmedium", "--filename", "/usr/local/vdis/os/disk_nova2.vdi", "--size", "409600", "--variant", "Standard"]
      end
      virthost.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 0, "--device", 0, "--type", "hdd", "--medium", "/usr/local/vdis/os/disk_nova2.vdi", "--nonrotational", "on", "--discard", "on"]

      # disable nested virtualization (not still usable, too much instability)
      virthost.customize ["modifyvm", :id, "--nested-hw-virt", "off"]
    end

    uosnv2.vm.provider :libvirt do |virthost|
      virthost.memory = 20480
      virthost.cpus = 6
      virthost.nested = true
      virthost.driver = 'kvm'
      virthost.cpu_mode = "host-model"
      virthost.cpu_feature :name => 'vmx', :policy => 'require'
      virthost.disk_bus = 'scsi'
      #virthost.disk_driver_opts :cache => "writeback", :io => "native", :discard => "unmap"
      virthost.storage :file, size: '400G', type: 'qcow2', path: 'disk_nova2.img', bus: "scsi", device: "sdb", cache: "writeback", io: "native", discard: "unmap"
    end

    # Mgmt openstack vlan with multiples id (10 / 20), for internal flux like db, storage, messaging, ...
    uosnv2.vm.network :private_network, libvirt__network_name: "os_mgmt", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.80.76", netmask: "255.255.255.128", virtualbox__intnet: "os_mgmt"
    # Neutron network vlan for openstack network stacks management
    uosnv2.vm.network :private_network, libvirt__network_name: "os_net", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.85.76", netmask: "255.255.255.128", virtualbox__intnet: "os_net", auto_config: false

    # Default input ip, used mainly for ansible reachablity
    uosnv2.vm.network :private_network, :ip => "192.168.95.76", :netmask => "255.255.255.128"
 end

  config.vm.define :storage1 do |uosst|
    uosst.vm.hostname = :storage1
    uosst.vm.box = "generic/ubuntu2004"
    uosst.vm.provider :virtualbox do |virthost|
      # Virtio adapter for all eth interfaces and ensure internal networks for openstack vlans
      virthost.customize ["modifyvm", :id, "--nictype1", "virtio"]
      virthost.customize ["modifyvm", :id, "--nic2", "intnet"]
      virthost.customize ["modifyvm", :id, "--nictype2", "virtio"]
      virthost.customize ["modifyvm", :id, "--nictype3", "virtio"]

      # disable dns server from host
      virthost.auto_nat_dns_proxy = false

      # RAM / CPU
      virthost.memory = 3072
      virthost.cpus = 2

      # Cinder LVM filesystem
      unless File.exist?("/usr/local/vdis/os/disk_cinder.vdi")
        virthost.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata", "--controller", "IntelAhci", "--portcount", "2", "--bootable", "off", "--hostiocache", "on"]
        virthost.customize ["createmedium", "--filename", "/usr/local/vdis/os/disk_cinder.vdi", "--size", "409600", "--variant", "Standard"]
      end
      virthost.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 0, "--device", 0, "--type", "hdd", "--medium", "/usr/local/vdis/os/disk_cinder.vdi", "--nonrotational", "on", "--discard", "on"]
    end

    uosst.vm.provider :libvirt do |virthost|
      virthost.memory = 3072
      virthost.cpus = 2
      virthost.driver = 'kvm'
      virthost.disk_bus = 'scsi'
      #virthost.disk_driver_opts :cache => "writeback", :io => "native", :discard => "unmap"
      virthost.storage :file, size: '400G', type: 'qcow2', path: 'disk_cinder.img', bus: "scsi", device: "sdb", cache: "writeback", io: "native", discard: "unmap"
    end

    # Mgmt openstack vlan with multiples id (10 / 20), for internal flux like db, storage, messaging, ...
    uosst.vm.network :private_network, libvirt__network_name: "os_mgmt", libvirt__dhcp_enabled: false, libvirt__forward_mode: "veryisolated", ip: "192.168.80.80", netmask: "255.255.255.128", virtualbox__intnet: "os_mgmt"
    # Default input ip, used mainly for ansible reachablity
    uosst.vm.network :private_network, :ip => "192.168.95.80", :netmask => "255.255.255.128"
  end
end
