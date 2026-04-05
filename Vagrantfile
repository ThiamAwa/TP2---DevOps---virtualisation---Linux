Vagrant.configure("2") do |config|
  # Box de base : Ubuntu 20.04 (Focal)
  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false   

  # ---------------------------
  # Machine srv-app (application)
  # ---------------------------
  config.vm.define "srv-app" do |app|
    app.vm.hostname = "srv-app"
    # Réseau privé avec IP fixe
    app.vm.network "private_network", ip: "192.168.56.10"
    # Redirection du port 8080 de la VM vers le port 8080 de l'hôte (pour Tomcat)
    app.vm.network "forwarded_port", guest: 8080, host: 8080
    # Configuration VirtualBox
    app.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.name = "srv-app"
    end
  end

  # ---------------------------
  # Machine srv-db (base de données)
  # ---------------------------
  config.vm.define "srv-db" do |db|
    db.vm.hostname = "srv-db"
    db.vm.network "private_network", ip: "192.168.56.11"
    db.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
      vb.name = "srv-db"
    end
   
  end
end