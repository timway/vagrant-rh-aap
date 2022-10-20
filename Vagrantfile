# -*- mode: ruby -*-


ENV["VAGRANT_NO_PARALLEL"] = "yes"


Vagrant.configure("2") do |config|
  config.vagrant.plugins = [ "vagrant-registration" => { :version => "1.3.4" } ]

  config.vm.box = "generic/rhel9"
  config.ssh.connect_timeout = 90

  config.vm.provider :libvirt do |libvirt|
    libvirt.qemu_use_session = false
  end

  config.vm.define "db" do |v|
    if Vagrant.has_plugin?("vagrant-registration")
      v.registration.username = ENV["SPONGE_RHSM_USERNAME"]
      v.registration.password = ENV["SPONGE_RHSM_PASSWORD"]
      v.registration.pools = [ ENV["SPONGE_RHSM_POOL_BASE"] ]
      v.registration.name = "aap-db.vagrant"
    end
    config.vm.provider "libvirt" do |h|
      h.cpus = 1
      h.memory = 2048
      h.management_network_domain = "vagrant.test"
    end
    v.vm.hostname = "db"
  end

  config.vm.define "sso", autostart: false do |v|
    if Vagrant.has_plugin?("vagrant-registration")
      v.registration.username = ENV["SPONGE_RHSM_USERNAME"]
      v.registration.password = ENV["SPONGE_RHSM_PASSWORD"]
      v.registration.pools = [ ENV["SPONGE_RHSM_POOL_BASE"], ENV["SPONGE_RHSM_POOL_AAP"] ]
      v.registration.name = "aap-sso.vagrant"
    end
    v.vm.box = "generic/rhel8"
    config.vm.provider "libvirt" do |h|
      h.cpus = 1
      h.memory = 2048
      h.management_network_domain = "vagrant.test"
    end
    v.vm.hostname = "sso"
    v.vm.network :forwarded_port, guest: 8443, host: 61443, host_ip: "0.0.0.0"
  end

  config.vm.define "hub.vagrant.test" do |v|
    if Vagrant.has_plugin?("vagrant-registration")
      v.registration.username = ENV["SPONGE_RHSM_USERNAME"]
      v.registration.password = ENV["SPONGE_RHSM_PASSWORD"]
      v.registration.pools = [ ENV["SPONGE_RHSM_POOL_BASE"], ENV["SPONGE_RHSM_POOL_AAP"] ]
      v.registration.name = "aap-hub.vagrant"
    end
    v.vm.hostname = "hub"
    v.vm.network :forwarded_port, guest: 443, host: 62443, host_ip: "0.0.0.0"
    v.vm.provider "libvirt" do |h|
      h.cpus = 1
      h.memory = 8320
    end
  end

  config.vm.define "hybcntrl" do |v|
    if Vagrant.has_plugin?("vagrant-registration")
      v.registration.username = ENV["SPONGE_RHSM_USERNAME"]
      v.registration.password = ENV["SPONGE_RHSM_PASSWORD"]
      v.registration.pools = [ ENV["SPONGE_RHSM_POOL_BASE"] ]
      v.registration.name = "aap-hybcntrl.vagrant"
    end
    config.vm.provider "libvirt" do |h|
      h.cpus = 1
      h.memory = 8320
    end
    v.vm.hostname = "hybcntrl"
    v.vm.network :forwarded_port, guest: 443, host: 63443, host_ip: "0.0.0.0"
    v.vm.provision "ansible" do |ansible|
      ansible.become = true
      ansible.compatibility_mode = "2.0"
      ansible.galaxy_role_file = "requirements.yml"
      ansible.groups = {
        "all:vars" => {
          :bundle_install => "False",
          :create_preload_data => "False",
          # FROM group_vars/all
          :tower_package_name => "automation-controller",
          :tower_package_version => "4.2.1",
#          :automationhub_package_name => "automation-hub",
#          :automationhub_package_version => "4.4.3",
#          :automation_platform_version => "2.2.0",
#          :automation_platform_channel => "ansible-automation-platform-2.1-for-rhel-8-x86_64-rpms",
          :minimum_ansible_version => "2.11",
          # FROM inventory
          "admin_password" => ENV["SPONGE_AAP_CONTROLLER_ADMIN_PASSWORD"],
          "pg_host" => "db",
          "pg_port" => "5432",
          "pg_database" => "awx",
          "pg_username" => "awx",
          "pg_password" => ENV["SPONGE_AAP_CONTROLLER_PG_PASSWORD"],
          "pg_sslmode" => "prefer",
          "registry_verify_ssl" => false,
          "registry_url" => "registry.redhat.io",
          "registry_username" => ENV["SPONGE_AAP_REGISTRY_USERNAME"],
          "registry_password" => ENV["SPONGE_AAP_REGISTRY_PASSWORD"],
          "receptor_listener_port" => 27199,
          "automationhub_admin_password" => ENV["SPONGE_AAP_HUB_ADMIN_PASSWORD"],
          "automationhub_pg_host" => "db",
          "automationhub_pg_port" => "5432",
          "automationhub_pg_database" => "automationhub",
          "automationhub_pg_username" => "automationhub",
          "automationhub_pg_password" => ENV["SPONGE_AAP_HUB_PG_PASSWORD"],
          "automationhub_pg_sslmode" => "prefer",
          "automationhub_upgrade" => "True",
          "sso_console_admin_password" => ENV["SPONGE_AAP_SSO_ADMIN_PASSWORD"],
          "sso_keystore_password" => ENV["SPONGE_AAP_SSO_KEYSTORE_PASSWORD"]
        },
        "automationcontroller" => ["hybcntrl"],
        "automationhub" => ["hub.vagrant.test"],
        "database" => ["db"],
        "sso" => ["sso"],
        "sso:vars" => {
        }
      }
      ansible.host_vars
      ansible.limit = "all,localhost"
      ansible.playbook = "playbook.yml"
    end
  end
end
