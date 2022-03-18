# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'

opts = GetoptLong.new(
  [ '--build-commit', GetoptLong::REQUIRED_ARGUMENT ],
  [ '--build-tag',    GetoptLong::REQUIRED_ARGUMENT ],
  [ '--git-repo',     GetoptLong::REQUIRED_ARGUMENT ],
  [ '--base-box',     GetoptLong::REQUIRED_ARGUMENT ]
)

# set default build commit
buildCommit='200cbd5717c15a338977dfe652c0411cca0d3583'
buildTag=''
# set default git repo
gitRepo='https://github.com/nasa/HDTN.git'
# set default base box type
baseBox='focal64'

opts.each do |opt, arg|
  case opt
    when '--build-commit'
      buildCommit=arg
    when '--build-tag'
      buildTag=arg
    when '--git-repo'
      gitRepo=arg
    when '--base-box'
      baseBox=arg
  end
end

# test with Ubuntu built focal64 and Bento built bento/ubunto-20.04
machines = [
  { selector: 'focal64', hostname: 'hdtn-ubuntu2004-lts',   ram: 1536, family: 'debian', box: 'ubuntu/focal64',     box_version: '20220302.0.0', cpus: 1 },
  { selector: 'bento',   hostname: 'hdtn-bentoubuntu-2004', ram: 1536, family: 'debian', box: 'bento/ubuntu-20.04', box_version: '202112.19.0',  cpus: 1 }
]

machine = machines.find {|x| x[:selector] == baseBox}

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  #config.vm.synced_folder '.', '/vagrant'

  # VirtualBox.
  config.vm.define "virtualbox" do |virtualbox|
    virtualbox.vm.hostname = machine[:hostname]
    virtualbox.vm.box = machine[:box]
    virtualbox.vm.box_version = machine[:box_version]

    config.vm.provider :virtualbox do |vbox|
      vbox.gui = false
      vbox.memory = machine[:ram]
      vbox.cpus = machine[:cpus]

      pwd = ENV["PWD"]
      home = ENV["HOME"]
      workingdir = pwd.sub(/#{home}/, '~')
      # Customize the Description section of the VM in VirtualBox:
      vbox.customize ["modifyvm", :id, "--description", "Vagrant running from #{workingdir}" ]
    end

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get install -y \
      cmake \
      build-essential \
      libzmq3-dev python3-zmq \
      libboost-dev libboost-all-dev \
      openssl libssl-dev
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    /bin/su -c "mkdir -p ~/working/github.com/nasa && \
                  cd ~/working/github.com/nasa && \
                  git clone #{gitRepo}" \
                    vagrant
  SHELL

  if buildTag.empty? 
    config.vm.provision "shell", inline: <<-SHELL
      /bin/su -c "cd ~/working/github.com/nasa/HDTN && \
                    git -c advice.detachedHead=false checkout #{buildCommit}" \
                      vagrant
    SHELL
  else
    config.vm.provision "shell", inline: <<-SHELL
      /bin/su -c "cd ~/working/github.com/nasa/HDTN && \
                    echo git -c advice.detachedHead=false checkout #{buildTag}" \
                      vagrant
    SHELL
  end

  config.vm.provision "shell", inline: <<-'SHELL'
    /bin/su -c "mkdir -p ~/working/build && cd working/build && \
                  cmake -DCMAKE_BUILD_TYPE=Release ../github.com/nasa/HDTN && \
                  echo 'Update CMAKE_INSTALL_PREFIX:PATH in CMakeCache.txt' && \
                  grep CMAKE_INSTALL_PREFIX:PATH CMakeCache.txt && \
                  sed -i'' 's#^\(CMAKE_INSTALL_PREFIX:PATH=\).*#\1/home/vagrant/HDTN#' CMakeCache.txt && \
                  echo '  with ' && \
                  grep CMAKE_INSTALL_PREFIX:PATH CMakeCache.txt && \
                  cmake -DCMAKE_BUILD_TYPE=Release ../github.com/nasa/HDTN" \
                    vagrant
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    /bin/su -c "echo 'cd working/build' > build-instructions && \
                  echo 'export HDTN_SOURCE_ROOT=~/working/github.com/nasa/HDTN' >> build-instructions && \
                  echo 'make' >> build-instructions && \
                  echo './tests/unit_tests/unit-tests' >> build-instructions && \
                  echo './tests/integrated_tests/integrated-tests' >> build-instructions" \
                    vagrant
  SHELL

  end
end
