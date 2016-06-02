# coding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.require_version ">= 1.7.2"

# Benötigte Plugins installieren
def ensure_plugins(plugins)
  logger = Vagrant::UI::Colored.new
  result = false
  plugins.each do |p|
    pm = Vagrant::Plugin::Manager.new(Vagrant::Plugin::Manager.user_plugins_file)
    plugin_hash = pm.installed_plugins
    next if plugin_hash.has_key?(p)
    result = true
    logger.warn("Installing plugin #{p}")
    pm.install_plugin(p)
  end
  if result
    logger.warn('Re-run vagrant up now that plugins are installed')
    exit
  end
end
ensure_plugins(["vagrant-vbguest", "vagrant-triggers"])

# Provisions-Skript inline definieren

$provisionScript = <<SCRIPT

export JAVA_VERSION_MAJOR=8
export JAVA_VERSION_MINOR=91
export JAVA_VERSION_BUILD=14
export WILDFLY_VERSION=10.0.0.Final
export MYSQL_USER=demouser
export MYSQL_PASSWORD=demopassword
export MYSQL_DB=demo

export JAVA_HOME=/opt/jdk
export JBOSS_HOME=/vagrant/wildfly
export M2_HOME=/opt/maven
export PATH=$PATH:"$JAVA_HOME"/bin:"$M2_HOME"/bin

cat <<EOF >> /etc/profile.d/env.sh
export JAVA_HOME=/opt/jdk
export JBOSS_HOME=/vagrant/wildfly
export M2_HOME=/opt/maven
export PATH=$PATH:"$JAVA_HOME"/bin:"$M2_HOME"/bin
EOF

apt-get update
apt-get upgrade

echo "Installing MySql-Database (Rootuser: root:root)"

debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'

apt-get install -y mysql-server

mysql -u root -p'root'
echo "Creating MySql Datebase $MYSQL_DB and User $MYSQL_USER"
mysql -uroot -p'root' -e "create user '$MYSQL_USER'@'localhost' identified by '$MYSQL_PASSWORD'"
mysql -uroot -p'root' -e "CREATE DATABASE $MYSQL_DB"
mysql -uroot -p'root' -e "grant all privileges on $MYSQL_DB.* to '$MYSQL_USER'@'localhost' identified by '$MYSQL_PASSWORD'"


echo "Installing Java..."
cd $HOME
curl -s -S -O -jksSLH "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/"$JAVA_VERSION_MAJOR"u"$JAVA_VERSION_MINOR"-b"$JAVA_VERSION_BUILD"/jdk-"$JAVA_VERSION_MAJOR"u"$JAVA_VERSION_MINOR"-linux-x64.tar.gz
 tar xf jdk-"$JAVA_VERSION_MAJOR"u"$JAVA_VERSION_MINOR"-linux-x64.tar.gz
 mv $HOME/jdk1."$JAVA_VERSION_MAJOR".0_"$JAVA_VERSION_MINOR" "$JAVA_HOME"
 rm jdk-"$JAVA_VERSION_MAJOR"u"$JAVA_VERSION_MINOR"-linux-x64.tar.gz
 java -version

echo "Installing Wildfly..."
rm -rf "$JBOSS_HOME"
curl  -s -S -O https://download.jboss.org/wildfly/"$WILDFLY_VERSION"/wildfly-"$WILDFLY_VERSION".tar.gz
tar xf wildfly-"$WILDFLY_VERSION".tar.gz
mv $HOME/wildfly-"$WILDFLY_VERSION" "$JBOSS_HOME"
rm wildfly-"$WILDFLY_VERSION".tar.gz
echo "Wildfly erfolgreich entpackt"

echo "Wildfly Konfigurieren..."
cp -f /vagrant/standalone-full.xml $JBOSS_HOME/standalone/configuration/standalone-full.xml
$JBOSS_HOME/bin/add-user.sh admin admin
echo "Wildfly Management User admin:admin hinzugefügt"

curl  -s -S -O -L https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.39.tar.gz
tar xf mysql-connector-java-5.1.39.tar.gz
mv mysql-connector-java-5.1.39/mysql-connector-java-5.1.39-bin.jar "$JBOSS_HOME"/standalone/deployments/mysql-connector.jar
rm mysql-connector-java-5.1.39.tar.gz
rm -rf mysql-connector-java-5.1.39

echo "Mysql Connector wurde eingerichetet"

cat <<EOF >> "$JBOSS_HOME"/standalone/deployments/mysql-ds.xml
<?xml version="1.0" encoding="UTF-8"?>
<datasources xmlns="http://www.jboss.org/ironjacamar/schema">
  <datasource jndi-name="java:/jboss/datasources/demo" pool-name="demo" enabled="true" use-java-context="true">
    <connection-url>jdbc:mysql://172.28.128.4:3306/$MYSQL_DB</connection-url>
    <driver>mysql-connector.jar_com.mysql.jdbc.Driver_5_1</driver>
    <security>
      <user-name>$MYSQL_USER</user-name>
      <password>$MYSQL_PASSWORD</password>
    </security>
    <statement>
      <prepared-statement-cache-size>100</prepared-statement-cache-size>
      <share-prepared-statements>true</share-prepared-statements>
    </statement>
  </datasource>
</datasources>
EOF

echo "DataSource wurde eingerichet"

adduser --system --group --disabled-login wildfly
chown -R wildfly:wildfly $JBOSS_HOME

cat <<EOF >> /etc/systemd/system/wildfly.service
[Unit]
Description=The WildFly Application Server
After=syslog.target network.target
Before=httpd.service

[Service]
Environment=LAUNCH_JBOSS_IN_BACKGROUND=1
Environment=JBOSS_HOME=$JBOSS_HOME
Environment=JAVA_HOME=$JAVA_HOME
User=wildfly
LimitNOFILE=102642
PIDFile=/var/run/wildfly/wildfly.pid
ExecStart=$JBOSS_HOME/bin/standalone.sh -c standalone-full.xml -b 0.0.0.0 --debug 8787
StandardOutput=null

[Install]
WantedBy=multi-user.target
EOF


systemctl daemon-reload
systemctl start wildfly.service
systemctl enable wildfly.service

echo "Wildfly konfiguriert"

echo "Installing Maven..."
curl  -s -S -O http://ftp.halifax.rwth-aachen.de/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar xf apache-maven-3.3.9-bin.tar.gz
mv $HOME/apache-maven-3.3.9 "$M2_HOME"
rm apache-maven-3.3.9-bin.tar.gz
echo "Maven erfolgreich entpackt"

echo ""
echo ""
echo "--------------------------------------"
echo "                Fertig !              "
echo "--------------------------------------"
SCRIPT


Vagrant.configure("2") do |config|

  config.vm.box = "debian/jessie64"
  config.vm.box_version = "= 8.4.0"

  config.vm.network "private_network", ip: "172.28.128.4"

  config.vm.hostname = "vagrant-demo.local"

  config.trigger.before :destroy do
    run_remote "sudo systemctl kill wildfly.service"
    run_remote "rm -rf /vagrant/wildfly"
  end

  config.vm.provider "virtualbox" do |v|
    v.memory =  4000
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
  config.vm.provision "shell", inline: $provisionScript, keep_color: true

end
