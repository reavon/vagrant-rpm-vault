# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = '2'

required_plugins = %w(vagrant-vbguest)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

@script = <<SCRIPT
sudo yum install -y rpmdevtools rpm-devel rpm-build mock

cat > /tmp/build.sh <<EOF
rpmdev-setuptree
ln -s /opt/rpm/SPECS/vault.spec /home/vagrant/rpmbuild/SPECS/
find /opt/rpm/SOURCES -type f -exec ln -s {} rpmbuild/SOURCES \\\;
spectool -g -R /home/vagrant/rpmbuild/SPECS/vault.spec
rpmbuild -ba /home/vagrant/rpmbuild/SPECS/vault.spec
cp /home/vagrant/rpmbuild/RPMS/x86_64/vault*.rpm /opt/rpm
EOF
chmod 0755 /tmp/build.sh

su - vagrant /tmp/build.sh
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.synced_folder '.', '/opt/rpm'

  config.vm.provider "virtualbox"
  config.vm.provider "vmware_fusion"

  config.vm.box = "centos/7"
  # config.vm.box = "centos-7.0-base"

  config.vm.provider :virtualbox do |vb|
    vb.customize ['modifyvm', :id, '--cpus', 2]
    vb.customize ['modifyvm', :id, '--memory', 2048]
  end

  config.vm.provision 'shell', inline: @script
end
