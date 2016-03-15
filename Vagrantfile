# -*- mode: ruby -*-
# vi: set ft=ruby :

$SCRIPT = <<SHELL
# network issue workaround
sudo hostname localhost

# configuring mysql
sudo yum install -y mysql-server wget
sudo service mysqld start

mysql -u root -e "CREATE DATABASE IF NOT EXISTS sonar;"
mysql -u root -e "GRANT USAGE ON *.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';"
mysql -u root -e "GRANT USAGE ON *.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';"
mysql -u root -e "GRANT ALL PRIVILEGES ON sonar.* TO 'sonar'@'%'"
mysql -u root -e "FLUSH PRIVILEGES"

# install java and maven
sudo yum install -y java-1.7.0-openjdk-devel
echo "Downloading maven"
curl -s -LO http://mirror.cc.columbia.edu/pub/software/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz
sudo tar xzf apache-maven-3.0.5-bin.tar.gz -C /usr/local
sudo ln -s /usr/local/apache-maven-3.0.5 /usr/local/maven

echo -e 'export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk.x86_64' >> "${HOME}/.bashrc"
source "${HOME}/.bashrc"
echo -e 'export M2_HOME=/usr/local/maven\nexport PATH=${M2_HOME}/bin:${PATH}' | sudo tee /etc/profile.d/maven.sh > /dev/null

rm apache-maven-3.0.5-bin.tar.gz

# configuring sonarqube
echo "Downloading sonarqube"
curl -s -LO --retry 5 https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-5.4.zip
sudo unzip sonarqube-5.4.zip -d /opt
sudo ln -s /opt/sonarqube-5.4 /usr/local/sonar
rm sonarqube-5.4.zip

sudo sed -i 's/^#sonar.jdbc.username=/sonar.jdbc.username=sonar/' /usr/local/sonar/conf/sonar.properties
sudo sed -i 's/^#sonar.jdbc.password=/sonar.jdbc.password=sonar/' /usr/local/sonar/conf/sonar.properties
sudo sed -i '/3306/s/^#//g' /usr/local/sonar/conf/sonar.properties
sudo sed -i 's/^#sonar.web.host=0.0.0.0/sonar.web.host=0.0.0.0/' /usr/local/sonar/conf/sonar.properties
sudo sed -i 's/^#sonar.web.context=/sonar.web.context=/' /usr/local/sonar/conf/sonar.properties
sudo sed -i 's/^#sonar.web.port=9000/sonar.web.port=9000/' /usr/local/sonar/conf/sonar.properties

echo "Downloading sonarqube plugins"
pushd /usr/local/sonar/extensions/plugins/
rm -f sonar-java-plugin*jar
sudo curl -s --retry 5 -L -o sonar-java-plugin-3.11.jar https://sonarsource.bintray.com/Distribution/sonar-java-plugin/sonar-java-plugin-3.11.jar
rm -f sonar-scm-git-plugin*jar
sudo curl -s --retry 5 -L -o sonar-scm-git-plugin-1.1.jar http://downloads.sonarsource.com/plugins/org/codehaus/sonar-plugins/sonar-scm-git-plugin/1.1/sonar-scm-git-plugin-1.1.jar
sudo curl -s --retry 5 -L -o sonar-pmd-plugin-2.5.jar https://sonarsource.bintray.com/Distribution/sonar-pmd-plugin/sonar-pmd-plugin-2.5.jar
sudo curl -s --retry 5 -L -o sonar-findbugs-plugin-3.3.jar https://sonarsource.bintray.com/Distribution/sonar-findbugs-plugin/sonar-findbugs-plugin-3.3.jar
sudo curl -s --retry 5 -L -o sonar-checkstyle-plugin-2.4.jar http://sonarsource.bintray.com/Distribution/sonar-checkstyle-plugin/sonar-checkstyle-plugin-2.4.jar
popd

# load mysqldump
if [[ -f "${HOME}/Hadoop-Word-Count/dump.sql" ]]; then
   mysql -uroot sonar < "${HOME}/Hadoop-Word-Count/dump.sql"
fi

sudo /usr/local/sonar/bin/linux-x86-64/sonar.sh start

# configuring hadoop
echo "Downloading hadoop"
curl --silent -LO --retry 5 http://www-eu.apache.org/dist/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.tar.gz
sudo tar -zxvf hadoop-2.7.2.tar.gz -C /opt
rm hadoop-2.7.2.tar.gz
sudo ln -s /opt/hadoop-2.7.2 /usr/local/hadoop
echo 'export HADOOP_HOME=/usr/local/hadoop' >> "${HOME}/.bashrc"
echo 'export PATH="${PATH}:${HADOOP_HOME}/bin"' >> "${HOME}/.bashrc"
source "${HOME}/.bashrc"

if [[ -f "${HOME}/Hadoop-Word-Count/message.txt" ]]; then
   echo "cat ${HOME}/Hadoop-Word-Count/message.txt" >> ~/.bashrc 
fi

SHELL

Vagrant.configure(2) do |config|

  config.vm.box = "centos-6.5"
  config.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box"

  config.vm.provider "virtualbox" do |v| 
    v.memory = 1536
    v.cpus = 2
  end

  config.vm.network "forwarded_port", guest: 9000, host: 9000

  config.vm.provision "shell", inline: $SCRIPT, privileged: false

  config.vm.synced_folder "./", "/home/vagrant/Hadoop-Word-Count"
end
