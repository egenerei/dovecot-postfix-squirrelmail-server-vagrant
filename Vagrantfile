#Function 
def create_client(config, name, ip, host_port,upgrade = nil)
  config.vm.box = "debian/bullseye64"
  config.vm.define name do |client|
    client.vm.network :private_network,
      ip: ip
    client.vm.hostname = name
    client.vm.network "forwarded_port", guest: 22, host: host_port, id: 'ssh'
    client.vm.network "forwarded_port", guest: 80, host: 8080
    config.ssh.insert_key = false
  if upgrade
    client.vm.provision "shell", name: upgrade, inline: <<-script
      apt-get update
      apt-get install debconf apache2 php7.4 libapache2-mod-php7.4  bind9 bind9-dnsutils -y
      debconf-set-selections <<< "postfix postfix/mailname string jdelrey.local"
      debconf-set-selections <<< "postfix postfix/main_mailer_type string 'localhost'"
      apt-get install postfix dovecot-core dovecot-imapd -y
      cp /vagrant/postfix_config/main.cf /etc/postfix/
      cp /vagrant/dovecot/10-auth.conf /etc/dovecot/conf.d/
      cp /vagrant/dovecot/10-mail.conf /etc/dovecot/conf.d/
      systemctl restart postfix
      systemctl enable postfix
      systemctl restart dovecot
      systemctl enable dovecot
      cp /vagrant/website_config/mail.conf /etc/apache2/sites-available/mail.conf
      a2dissite 000-default.conf
      a2ensite mail.conf
      cp -r /vagrant/squirrel /usr/local/
      ln -s /usr/local/squirrel /var/www/mail
      mkdir -p /var/local/squirrelmail/data
      chown www-data:www-data /var/local/squirrelmail/data
      mkdir -p /var/local/squirrelmail/attach
      chown www-data:www-data /var/local/squirrelmail/attach
      locale-gen es_ES
      a2enmod php7.4
      systemctl start apache2
      systemctl reload apache2
      systemctl enable apache2
      cp /vagrant/db.aula.izv /etc/bind/zones/db.aula.izv
      cp /vagrant/db.192.168.56 /var/lib/bind/db.192.168.56
      cp /vagrant/named.conf.local /etc/bind/named.conf.local
      systemctl restart bind9.service
    script
  end
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = 1
  end
  end
end

Vagrant.configure("2") do |config|
  create_client(config, "mail.aula.izv", "192.168.56.101",2201,"mail")
  #create_client(config, "web", "192.168.56.102",2202)
  #create_client(config, "tierra.sistema.sol", "192.168.56.103",2203)
  #create_client(config, "marte.sistema.sol", "192.168.56.104",2204)
end
