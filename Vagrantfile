# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "CentOS_6_7"
  config.vm.hostname = "golem"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 3000, host: 3000

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"
  config.vm.network "public_network", ip: "192.168.135.230"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false
  
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL

  TARGETS = [
    :tools,
    :mysql,
    :postgresql,
    :ntp,
    :memcached,
    :redis,
    # :ffmpeg, # repo消失のため
    :jdk,
    :elasticsearch,
    :nginx,
    :tmux,
    :rbenv,
    :nvm,
    :user_setting,
  ]

  FUNCTION_INSTALL =<<-EOS
    function install {
      echo installing $1
      shift
      yum -y install "$@" >/dev/null 2>&1
    }
  EOS

  CONF_DIR = '/vagrant/conf.d'

  # development tools etc...
  if TARGETS.include?(:tools)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      yum -y update >/dev/null 2>&1

      install "development tools" gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel zlib zlib-devel libpng-devel
      install "Git" git
      install "sqlite" sqlite sqlite-devel
      install "Nokogiri dependencies" libxml2 libxslt libxml2-devel libxslt-devel
      install "ImageMagick" ImageMagick ImageMagick-devel
      install "vim" vim-common vim-enhanced

      cp /etc/localtime /etc/localtime.org
      ln -sf  /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
      echo "ZONE=\"Asia/Tokyo\"" > /etc/sysconfig/clock
      service crond restart
    SHELL
  end

  # MySQL5.6
  if TARGETS.include?(:mysql)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      yum install -y https://dev.mysql.com/get/mysql57-community-release-el6-11.noarch.rpm >/dev/null 2>&1
      install "MySQL" mysql mysql-server mysql-devel
      chkconfig --add mysqld
      chkconfig --level 345 mysqld  on

      echo "Start and Initialize MySQL"
      service mysqld start >/dev/null 2>&1

      mpw=$(cat /var/log/mysqld.log | grep "A temporary password is generated for root@localhost" | awk '{print $NF}')
      mysql -uroot -p$mpw --connect-expired-password <<SQL
-- CONFIG PASSWORD POLICY --
SET PASSWORD FOR root@localhost=password('passwordPASSWORD@999');
SET GLOBAL validate_password_length=4;
SET GLOBAL validate_password_policy=LOW;
-- SET ROOT PASSWORD --
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('root');
-- REMOVE ANONYMOUS USERS --
DELETE FROM mysql.user WHERE User='';
-- REMOVE REMOTE ROOT --
DELETE FROM mysql.user
WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
-- REMOVE TEST DATABASE --
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
-- RELOAD PRIVILEGE TABLES --
FLUSH PRIVILEGES;
CREATE USER 'vagrant'@'localhost' IDENTIFIED BY 'vagrant';
GRANT ALL PRIVILEGES ON *.* to 'vagrant'@'localhost';
SQL
    SHELL
  end

  # PostgreSQL9.4
  # http://tokibito.hatenablog.com/entry/20150131/1422712122
  if TARGETS.include?(:postgresql)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      echo installing postgresql-9.4
      #{FUNCTION_INSTALL}
      mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.org
      cp #{CONF_DIR}/postgresql/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
      yum -y localinstall https://download.postgresql.org/pub/repos/yum/9.4/redhat/rhel-6-x86_64/pgdg-centos94-9.4-3.noarch.rpm
      install "PostgreSQL" postgresql94-server postgresql94-devel postgresql94-contrib
      service postgresql-9.4 initdb
      mv /var/lib/pgsql/9.4/data/pg_hba.conf /var/lib/pgsql/9.4/data/pg_hba.conf.org
      cp #{CONF_DIR}/postgresql/pg_hba.conf /var/lib/pgsql/9.4/data/pg_hba.conf
      service postgresql-9.4 start
      chkconfig postgresql-9.4 on
      echo 'export PATH=/usr/pgsql-9.4/bin:$PATH' >> /home/vagrant/.bashrc
    SHELL
  end

  # ntp
  if TARGETS.include?(:ntp)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      install "ntp" ntp
      chkconfig ntpd on
      echo "Start and Initialize ntp"
      service ntpd start >/dev/null 2>&1
    SHELL
  end

  # memcached
  if TARGETS.include?(:memcached)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      install "memcached" memcached memcached-devel
      chkconfig --add memcached
      chkconfig --level 345 memcached  on
      echo "Start and Initialize memcached"
      service memcached start >/dev/null 2>&1
    SHELL
  end

  # redis
  if TARGETS.include?(:redis)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      install "redis" redis --enablerepo=epel
      chkconfig redis on
      echo "Start and Initialize redis"
      service redis start >/dev/null 2>&1
    SHELL
  end

  # ffmpeg
  # http://blog.kaiman.net/?p=1230
  if TARGETS.include?(:ffmpeg)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      echo installing ffmpeg
      rpm --import http://packages.atrpms.net/RPM-GPG-KEY.atrpms
      rpm -ivh http://dl.atrpms.net/all/atrpms-repo-6-7.el6.x86_64.rpm
      yum -y --enablerepo=atrpms install ffmpeg ffmpeg-devel
    SHELL
  end

  # JDK
  if TARGETS.include?(:jdk)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      echo installing JDE
      install "JDK" java-1.8.0-openjdk java-1.8.0-openjdk-devel
    SHELL
  end

  # Elasticsearch
  if TARGETS.include?(:elasticsearch)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      echo installing Elasticsearch
      rpm -ivh https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.1.rpm
      chkconfig --add elasticsearch
      service elasticsearch start
    SHELL
  end

  # nginx
  if TARGETS.include?(:nginx)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      echo installing nginx
      #{FUNCTION_INSTALL}
      install "nginx" nginx
      cp #{CONF_DIR}/nginx/default.conf /etc/nginx/conf.d/default.conf
      service nginx start
      chkconfig nginx on
      rm -Rf /usr/share/nginx/html
      ln -s /vagrant/html /usr/share/nginx/html
    SHELL
  end

  if TARGETS.include?(:tmux)
    config.vm.provision "shell", privileged: true, inline: <<-SHELL
      #{FUNCTION_INSTALL}
      echo installing tmux
    
      install "libevent" libevent2-devel
      install "ncurses" ncurses-devel
    
      cd /usr/local/src
      wget https://github.com/tmux/tmux/releases/download/2.3/tmux-2.3.tar.gz
      tar -xvf tmux-2.3.tar.gz
      cd tmux-2.3
      ./configure LDFLAGS=-L/usr/local/lib/ && make
      make install
    SHELL
  end

  # rbenv
  if TARGETS.include?(:rbenv)
    GUEST_RUBY_VERSION = '2.3.3'

    config.vm.provision "shell", privileged: false, inline: <<-SHELL
      echo installing rbenv
      git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
      git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
      echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
      echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
      source ~/.bash_profile
      echo 'gem: --no-ri --no-rdoc' >> ~/.gemrc
      echo installing ruby#{GUEST_RUBY_VERSION}
      RUBY_BUILD_CURL_OPTS=--tlsv1.2 rbenv install #{GUEST_RUBY_VERSION}
      rbenv global #{GUEST_RUBY_VERSION}
      echo installing Bundler
      gem install bundler -N >/dev/null 2>&1
      echo installing ruby-gemset
      cd ~/.rbenv/plugins/
      git clone https://github.com/jf/rbenv-gemset.git    
      rbenv gemset create #{GUEST_RUBY_VERSION} logica
      cd ~
      echo logica > .rbenv-gemsets
      rbenv rehash
      gem install bundler
    SHELL
  end

  # nvm
  if TARGETS.include?(:nvm)
    config.vm.provision "shell", privileged: false, inline: <<-SHELL
      echo installing nvm
      cd
      curl -o- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
      export NVM_DIR="/home/vagrant/.nvm"
      [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
      nvm install 6.9.1
      nvm alias default 6.9.1
      npm update -g npm
      npm install -g grunt-cli gulp webpack yarn pm2 node
    SHELL
  end

  # user setting
  if TARGETS.include?(:user_setting)
    config.vm.provision "shell", privileged: false, inline: <<-SHELL
      echo user setting
      git config --global color.ui auto
      cd
      wget -O .git-completion.bash https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash
      echo source ~/.git-completion.bash >> ~/.bashrc
      echo finish user setting
    SHELL
  end
end
