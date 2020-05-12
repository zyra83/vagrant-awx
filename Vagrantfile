#http://www.inanzzz.com/index.php/post/vaqo/transferring-secret-files-and-content-with-ansible-vault-and-vagrant
$msg = <<MSG
------------------------------------------------------
AWX

Pour activer les guest sur les VM : https://shanemcd.org/2018/12/15/installing-virtualbox-guest-additions-in-a-debian-vagrant-box-on-windows-10/
Le tuto que j'ai suivi : https://www.linuxtechi.com/install-ansible-awx-docker-compose-centos-8/

URLS:
 - HTTP   - http://192.168.0.166/
 - HTTPS  - https://192.168.0.166/

------------------------------------------------------
MSG

Vagrant.configure("2") do |config|

  config.vm.box = "centos/8"
  # config.vm.network "forwarded_port", guest: 80, host: 8080, id: 'gitlab_http'
  # config.vm.network "forwarded_port", guest: 443, host: 4443, id: 'gitlab_https'

  config.vm.network "public_network", ip: "192.168.0.166", :bridge => 'enp5s0'

  # ça c'est pour les droits des clés quand le projet est stocké dans une partoche NTFS
  config.ssh.private_key_path = ["~/.ssh/id_rsa", "~/.vagrant.d/insecure_private_key"]
  
  # pas besoin des guest additions, bug sur centos8...
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.ssh.insert_key = false
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      # echo #{ssh_pub_key} >> /root/.ssh/authorized_keys 
      dnf install epel-release -y
      dnf install git gcc gcc-c++ nodejs gettext device-mapper-persistent-data lvm2 bzip2 python3-pip -y
      dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
      dnf install docker-ce-3:18.09.1-3.el7 -y
      systemctl enable --now docker.service
      alternatives --set python /usr/bin/python3
      pip3 install docker-compose
      dnf install ansible -y

      git clone https://github.com/ansible/awx.git
      cd awx/installer/
      echo "tymy"

      sed -E 's/.*postgres_data_dir=.*/postgres_data_dir="\/var\/lib\/awxpgdocker"/gm' --in-place inventory
      sed -E 's/.*awx_official=.*/awx_official=true/gm' --in-place inventory
      # localhost ansible_connection=local ansible_python_interpreter="/usr/bin/env python3"
      sed -E 's/.*project_data_dir=.*/project_data_dir=\/var\/lib\/awx\/projects/gm' --in-place inventory
      
      sed -E 's/.*awx_alternate_dns_servers=.*/awx_alternate_dns_servers="4\.2\.2\.1,4\.2\.2\.2"/gm' --in-place inventory
      sed -E 's/.*secret_key=.*/secret_key=SGYsSWciI5yTIMYZuEm5wW98pQeJMG+ACABPsGfC"/gm' --in-place inventory
      
      ansible-playbook -i inventory install.yml
      
    SHELL
  end

  config.vm.provider "virtualbox" do |vb|
    vb.name = "AWX - 192.168.0.166"
    vb.memory = "4096"
    vb.cpus = 2
  end 

  config.vm.post_up_message = $msg
end
