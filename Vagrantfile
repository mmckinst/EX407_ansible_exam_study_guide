Vagrant.configure("2") do |config|

  config.vm.define "web01" do |web|
    web.vm.box = "centos/7"
  end

  config.vm.define "db01" do |web|
    web.vm.box = "centos/7"
  end

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "install.yaml"
  end
end
