# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :"sysd" => {
        :box_name => "generic/centos8",
        :ip_addr => '192.168.56.141'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.synced_folder "scripts", "/shfile"

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "2048"]
          end
          
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
           # cd /etc/yum.repos.d/
           # sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
           # sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
           # yum update -y
            yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
            cp /shfile/watchlog /etc/sysconfig
            cp /shfile/alert.sh /opt
            cp /shfile/watchlog.log /var/log
            cp /shfile/watchlog.service /etc/systemd/system
	    cp /shfile/watchlog.sh /opt
	    cp /shfile/watchlog.timer /etc/systemd/system
            cp /shfile/first.conf /etc/httpd/conf
	    cp /shfile/httpd-first /etc/sysconfig
	    cp /shfile/httpd-second /etc/sysconfig
            cp /shfile/spawn-fcgi /etc/sysconfig
            cp /shfile/spawn-fcgi.service /etc/systemd/system 
	    cp '/shfile/httpd@first.service' /etc/systemd/system
	    cp '/shfile/httpd@second.service' /etc/systemd/system
	    cp /shfile/second.conf /etc/httpd/conf
	    cp /shfile/tail_add.sh /opt
	    chmod +x /opt/*.sh
            systemctl start watchlog.timer
            systemctl start watchlog.service
            systemctl start spawn-fcgi
            systemctl start httpd@first
            systemctl start httpd@second            
          SHELL

      end
  end
end
