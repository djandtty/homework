
```
cat Vagrantfile
MACHINES = {
  :"kernel-update" => {
              :box_name => "generic/centos8s",
              :box_version => "4.3.4",
              :cpus => 4,
              :memory => 4096,
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
    end
  end
end
```

Текущее ядро:  
```
uname -r
   
   4.18.0-516.el8.x86_64
```
Подключаем репозиторий:  
`sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm`  
Устанавливаю последнюю kernel-ml версию ядра:  
`sudo yum --enablerepo elrepo-kernel install kernel-ml -y`  
Перезагружаю и проверяю:  
```
uname -r
   
   6.7.8-1.el8.elrepo.x86_64
```
