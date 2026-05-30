# ============================================================
#  AOT — Automation Orchestration Toolkit — Vagrantfile
#  Membuat 3 VM VirtualBox + provisioning Ansible otomatis
#
#  Cara pakai:
#    vagrant up          → buat semua VM + install tools
#    vagrant halt        → matikan semua VM
#    vagrant up          → hidupkan kembali
#    vagrant destroy -f  → hapus semua VM
#
#  Prasyarat:
#    - VirtualBox (https://www.virtualbox.org)
#    - Vagrant    (https://www.vagrantup.com)
# ============================================================

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_check_update = false

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  # ══════════════════════════════════════════
  #  VM 1 — Wazuh Server
  # ══════════════════════════════════════════
  config.vm.define "wazuh" do |wazuh|
    wazuh.vm.hostname = "aot-wazuh"
    wazuh.vm.network "private_network", ip: "192.168.56.10"
    wazuh.vm.provider "virtualbox" do |vb|
      vb.name   = "aot-wazuh"
      vb.memory = 8192
      vb.cpus   = 4
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
    end
    wazuh.vm.provision "shell", inline: <<-SHELL
      echo "PasswordAuthentication yes" > /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
      echo "vagrant:vagrant" | chpasswd
      systemctl restart ssh
    SHELL
  end

  # ══════════════════════════════════════════
  #  VM 2 — DFIR-IRIS
  # ══════════════════════════════════════════
  config.vm.define "iris" do |iris|
    iris.vm.hostname = "aot-iris"
    iris.vm.network "private_network", ip: "192.168.56.11"
    iris.vm.provider "virtualbox" do |vb|
      vb.name   = "aot-iris"
      vb.memory = 4096
      vb.cpus   = 2
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
    end
    iris.vm.provision "shell", inline: <<-SHELL
      echo "PasswordAuthentication yes" > /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
      echo "vagrant:vagrant" | chpasswd
      systemctl restart ssh
    SHELL
  end

  # ══════════════════════════════════════════
  #  VM 3 — Shuffle
  #  Ansible dijalankan DI SINI setelah VM ke-3 siap
  # ══════════════════════════════════════════
  config.vm.define "shuffle" do |shuffle|
    shuffle.vm.hostname = "aot-shuffle"
    shuffle.vm.network "private_network", ip: "192.168.56.12"
    shuffle.vm.provider "virtualbox" do |vb|
      vb.name   = "aot-shuffle"
      vb.memory = 4092
      vb.cpus   = 2
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
    end

    shuffle.vm.provision "shell", inline: <<-SHELL
      echo "PasswordAuthentication yes" > /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
      echo "vagrant:vagrant" | chpasswd
      systemctl restart ssh
      apt-get update && apt-get install -y sshpass
    SHELL

    shuffle.vm.provision "ansible_local" do |ansible|
      ansible.playbook           = "ansible/playbook.yml"
      ansible.inventory_path     = "ansible/inventory/hosts"
      ansible.limit              = "all"
      ansible.compatibility_mode = "2.0"
      ansible.install            = true
      ansible.extra_vars         = {
        ansible_python_interpreter: "/usr/bin/python3"
      }
    end
  end

end
