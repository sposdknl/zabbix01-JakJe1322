Vagrant.configure("2") do |config|
  # Definice virtuálního stroje pro Zabbix Server
  config.vm.define "zabbix_server" do |server|
    server.vm.box = "debian/bookworm64"  # Debian 12 (Bookworm)
    server.vm.hostname = "zabbix-server"

    # Nastavení síťového mostu a přesměrování portů
    server.vm.network "public_network"  # Připojí server přímo do lokální sítě
    server.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

    # Zdroje: Nastavení paměti a CPU
    server.vm.provider "virtualbox" do |vb|
      vb.memory = 2048  # 2GB RAM
      vb.cpus = 2       # 2 CPU jádra
    end

    # Provisioning: Instalace Zabbix Serveru
    server.vm.provision "shell", inline: <<-SHELL
      # Aktualizace systému a instalace nutných balíčků
      apt update
      apt install -y wget gnupg2 software-properties-common mariadb-server apache2

      # Přidání Zabbix repozitáře
      wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
      dpkg -i zabbix-release_latest_7.0+debian12_all.deb
      apt update

      # Instalace Zabbix serveru, frontendu a agenta
      apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

      # Instalace chybějící lokalizace
      apt install -y locales
      echo "en_US.UTF-8 UTF-8" > /etc/locale.gen
      locale-gen en_US.UTF-8
      update-locale LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8

      # Nastavení MariaDB pro Zabbix
      mysql -uroot -e "CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;"
      mysql -uroot -e "CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'zabbix_password';"
      mysql -uroot -e "GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';"
      mysql -uroot -e "SET GLOBAL log_bin_trust_function_creators = 1;"
      
      # Import databázového schématu
      zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p'zabbix_password' zabbix
      mysql -uroot -e "SET GLOBAL log_bin_trust_function_creators = 0;"

      # Konfigurace Zabbix Serveru
      sed -i 's/^# DBPassword=/DBPassword=zabbix_password/' /etc/zabbix/zabbix_server.conf

      # Restartování a povolení služeb
      systemctl restart zabbix-server zabbix-agent apache2
      systemctl enable zabbix-server zabbix-agent apache2
    SHELL
  end

  # Definice virtuálního stroje pro Zabbix Agent
  config.vm.define "zabbix_agent" do |agent|
    agent.vm.box = "debian/bookworm64"  # Debian 12 (Bookworm)
    agent.vm.hostname = "zabbix-agent"

    # Nastavení síťového mostu
    agent.vm.network "public_network"  # Připojí agenta přímo do lokální sítě

    # Zdroje: Nastavení paměti a CPU
    agent.vm.provider "virtualbox" do |vb|
      vb.memory = 512  # 512MB RAM
      vb.cpus = 1      # 1 CPU jádro
    end

    # Provisioning: Instalace Zabbix Agenta
    agent.vm.provision "shell", inline: <<-SHELL
      # Aktualizace systému a instalace Zabbix agenta
      apt update
      apt install -y wget gnupg2 software-properties-common

      # Přidání Zabbix repozitáře
      wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
      dpkg -i zabbix-release_latest_7.0+debian12_all.deb
      apt update
      apt install -y zabbix-agent

      # Instalace chybějící lokalizace
      apt install -y locales
      echo "en_US.UTF-8 UTF-8" > /etc/locale.gen
      locale-gen en_US.UTF-8
      update-locale LANG=en_US.UTF-8 LANGUAGE=en_US.UTF-8

      # Konfigurace Zabbix Agenta
      sed -i 's/^# Server=127.0.0.1/Server=zabbix_server_ip/' /etc/zabbix/zabbix_agentd.conf
      sed -i 's/^# ServerActive=127.0.0.1/ServerActive=zabbix_server_ip/' /etc/zabbix/zabbix_agentd.conf

      # Restartování a povolení Zabbix Agenta
      systemctl restart zabbix-agent
      systemctl enable zabbix-agent
    SHELL
  end
end
