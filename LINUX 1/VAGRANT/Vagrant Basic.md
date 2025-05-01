
Create a new dir and from in execute 

``` cmd
vagrant init
```

This will create a Vagrant file

#### Basic Vagrant File

Choose a Vagrant box from https://app.vagrantup.com/boxes/search?utf8=%E2%9C%93&sort=updated&provider=&q=centos

``` ruby
Vagrant.configure("2") do |config|
	config.vm.box = "generic/debian12"
	config.vm.network "forwarded_port", guest: 80, host: 8080
	config.vm.synced_folder "./temp", "/home/vagrant/temp"
	config.vm.provider "virtualbox" do |vb|
		vb.gui = true
		vb.memory = "1024"
	end
end
```

Check the status of vagrant virtual machines

``` cmd
vagrant status
```

```
Current machine states:

default                   running (virtualbox)
```

default: the name of the vm

To ssh into a vagrant host

```
vagrant ssh
```

To stop a vagrant vm

```
vagrant halt
```

#### Create a Vagrant Box

