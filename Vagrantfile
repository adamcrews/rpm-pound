# -*- mode: ruby -*-
# vi: set ft=ruby :

# This Vagrantfile will build an rpm of the current version of 
# Pound.  Just run 'vagrant up' and wait for the rpm to be 
# written to the cwd.

$buildrpm = <<BUILDRPM

echo "Building all Pound rpms from the available srpms"

for SRPM in /vagrant/srpm/*src.rpm; do
  
  BUILD_REQUIRES=`rpm2cpio ${SRPM} | cpio -i --quiet --to-stdout pound.spec | awk -F: '/^BuildRequires/ {req_line=$2 ; split(req_line,dep_array,","); for (item in dep_array) { split(dep_array[item],final_deps," "); for (dependancy in final_deps) printf "%s ", final_deps[1]}}'`

  echo "SRPM: ${SRPM}"
  echo "BUILD_REQUIRES: ${BUILD_REQUIRES}"

  echo "Installing dependencies via yum"
  set -x
  #sudo yum install -y epel-release
  #sudo yum clean metadata
  sudo yum groupinstall -y 'Development tools'
  sudo yum install -y rpmdevtools ${BUILD_REQUIRES}
  set +x
  echo "Dependencies installed."

  echo "Setting up RPM build env"
  rpmdev-setuptree

  echo "Building..."
  rpmbuild --rebuild $SRPM

  echo "Copying output to the vagrant share."
  cp ~/rpmbuild/RPMS/*/* /vagrant

  echo "Done building pound from $SRPM"
done
BUILDRPM

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Bare bones Centos 6.5 box from a trustworthy community member.
  config.vm.box = "puppetlabs/centos-6.5-64-nocm"

  # This optional plugin caches RPMs for faster rebuilds.
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine
  end

  config.vm.provision "shell", privileged: false, inline: $buildrpm
end
