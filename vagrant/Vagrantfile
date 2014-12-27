# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos64_ja"
  config.vm.box_url = "https://dl.dropboxusercontent.com/u/3657281/centos64_ja.box"
  config.vm.hostname = "lapp-sample"

  config.vm.network :private_network, ip: "192.168.33.30"
  #config.vm.synced_folder "../", "/share", :nfs => true
  config.vm.synced_folder "../", "/share", \
        create: true, owner: 'vagrant', group: 'vagrant', \
        mount_options: ['dmode=777,fmode=666']

  config.vm.provider :virtualbox do |vb|
    vb.name = "lapp-sample"
    vb.gui = true
    #vb.customize ["modifyvm", :id, "--memory", 1024]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  config.vm.provision :shell, :inline => <<-EOT
    #
    # iptables off
    #
    /sbin/iptables -F
    /sbin/service iptables stop
    /sbin/chkconfig iptables off
    #
    # yum repository
    #
    yum -y update ca-certificates
    rpm -ivh http://ftp.riken.jp/Linux/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
    rpm -ivh http://dl.iuscommunity.org/pub/ius/stable/CentOS/6/x86_64/ius-release-1.0-11.ius.centos6.noarch.rpm
    #yum -y update
    #
    # ntp
    #
    yum -y install ntp
    /sbin/service ntpd start
    /sbin/chkconfig ntpd on
    #
    # php
    #
    yum -y install php54 php54-cli php54-pdo php54-mbstring php54-mcrypt php54-pecl-memcache php54-mysql php54-devel php54-common php54-pgsql php54-pear php54-gd php54-xml php54-pecl-xdebug php54-pecl-apc
    touch /var/log/php.log && chmod 666 /var/log/php.log
    cp -a /vagrant/php.ini /etc/php.ini
    #
    # Apache
    #
    cp -a /vagrant/httpd.conf /etc/httpd/conf/
    /sbin/service httpd restart
    /sbin/chkconfig httpd on
    #
    # PostgreSQL
    #
    rpm -ivh http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-1.noarch.rpm
    yum -y install postgresql93-server
    if [ ! -f /var/lib/pgsql/9.3/data/postgresql.conf ]; then
      su postgres -c '/usr/pgsql-9.3/bin/initdb --no-locale -D /var/lib/pgsql/9.3/data'
      if [ -f /vagrant/postgresql.conf ]; then
        cp -a /vagrant/postgresql.conf /var/lib/pgsql/9.3/data/
      fi
    fi
    /sbin/service postgresql-9.3 restart
    /sbin/chkconfig postgresql-9.3 on

    /usr/bin/createuser -d -A -S -U postgres vagrant
    /usr/bin/createdb -Uvagrant -E UTF-8 -T template0 app
    /usr/bin/createdb -Uvagrant -E UTF-8 -T template0 app_test
    #
    # phpPgAdmin
    #
    sudo yum -y install phpPgAdmin
    cp -a /vagrant/phpPgAdmin.conf /etc/httpd/conf.d/
    /sbin/service httpd restart
    #
    # Composer
    #
    if [ -f /share/composer.json ]; then
      cd /share && curl -s http://getcomposer.org/installer | php
      /usr/bin/php /share/composer.phar install
    fi
    cp -a /share/app/Config/database.php.default /share/app/Config/database.php
    cp -a /share/app/Config/bootstrap.php.default /share/app/Config/bootstrap.php
    cp -a /share/app/Config/email.php.default /share/app/Config/email.php
  EOT
end
