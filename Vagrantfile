Vagrant.configure("2") do |config|
    # all of your forgerock install binaries should be on your host in ../data
    config.vm.synced_folder "../data", "/vagrant_data"
    config.vm.provision "shell", inline: <<-SHELL
      add-apt-repository ppa:openjdk-r/ppa -y
      apt-get update
      echo "\n------ Installing Reequired Packages --------\n"
      # avahi-daemon and libnss-mdns for fqdn based networking between the servers
      apt-get -y install zip net-tools openjdk-11-jdk avahi-daemon libnss-mdns
      update-alternatives --config java
    SHELL

    ## build DS config store
    config.vm.define "vm1" do |vm1|
        vm1.vm.box = "ubuntu/focal64"
        vm1.vm.hostname = "ds1.local"
        vm1.vm.network "private_network", type: "dhcp"
        vm1.vm.network "forwarded_port", guest: 1636, host: 1636
        vm1.vm.provision "shell", inline: <<-SHELL
            echo "\n--- Running Provision Script -----\n"
            sudo groupadd ds_user
            sudo useradd -s /bin/false -g ds_user -d /opt/forgerock/cfg ds_user
            echo "\n---- Setting up Staging area ---\n"
            sudo mkdir -p /opt/forgerock/staging 
            sudo cp /vagrant_data/staging/DS* /opt/forgerock/staging/.
            sudo unzip /opt/forgerock/staging/DS* -d /opt/forgerock/staging/.
            sudo mv /opt/forgerock/staging/opendj /opt/forgerock/opendj
            sudo chgrp -R ds_user /opt/forgerock/opendj
            sudo chmod g+rwx /opt/forgerock/opendj
            sudo chmod g+r /opt/forgerock
            sudo chown -R ds_user /opt/forgerock/opendj
            echo "\n---- Creating Deployment Key ---\n"
            DEPLOYMENT_KEY=$(/opt/forgerock/opendj/bin/dskeymgr create-deployment-key --deploymentKeyPassword password --validity "10 years")
            export $DEPLOYMENT_KEY
            echo "Here's the deployment key: "
            echo "${DEPLOYMENT_KEY}"
            echo "\n---- Starting setup ... ---\n"
            /opt/forgerock/opendj/setup --deploymentKey $DEPLOYMENT_KEY \
                --deploymentKeyPassword password \
                --rootUserDN uid=admin \
                --rootUserPassword str0ngAdm1nPa55word \
                --monitorUserPassword str0ngMon1torPa55word \
                --hostname ds.example.com \
                --adminConnectorPort 4444 \
                --ldapPort 1389 \
                --enableStartTls \
                --ldapsPort 1636 \
                --httpsPort 8443 \
                --profile am-config \
                --set am-config/amConfigAdminPassword:5up35tr0ng \
                --acceptLicense
            # startup DS 
            echo "\n--- Starting DS ---\n"
            /opt/forgerock/opendj/bin/start-ds
            # Export the keys for AM to use to connect
            echo "\n--- Exporting Keys ---\n"
            keytool -exportcert \
            -keystore /opt/forgerock/opendj/config/keystore \
            -storepass $(cat /opt/forgerock/opendj/config/keystore.pin) \
            -alias ssl-key-pair \
            -rfc \
            -file ds-cert.pem
            # move the cert to the shared mount 
            echo "\n--- Moving the keys to the shared mount ---\n"
            cp -R ds-cert.pem /vagrant_data/ds-cfg-cert.pem
        SHELL
      end

      ## build DS cts store 
      config.vm.define "vm2", autostart:false do |vm2|
        vm2.vm.box = "ubuntu/focal64"
        vm2.vm.hostname = "ds2.local"
        vm2.vm.network "private_network", type:"dhcp"
        vm2.vm.network "forwarded_port", guest: 1636, host: 2636
        vm2.vm.provision "shell", inline: <<-SHELL
            echo "\n--- Running Provision Script -----\n"
            groupadd ds_user
            useradd -s /bin/false -g ds_user -d /opt/forgerock/cfg ds_user
            echo "\n---- Setting up Staging area ---\n"
            sudo mkdir -p /opt/forgerock/staging 
            sudo cp /vagrant_data/DS* /opt/forgerock/staging/.
            sudo unzip /opt/forgerock/staging/DS* -d /opt/forgerock/staging/.
            sudo mv /opt/forgerock/staging/opendj /opt/forgerock/opendj
            chgrp -R ds_user /opt/forgerock/opendj
            chmod g+rwx /opt/forgerock/opendj
            chmod g+r /opt/forgerock
            chown -R ds_user /opt/forgerock/opendj
            echo "\n---- Creating Deployment Key ---\n"
            DEPLOYMENT_KEY=$(/opt/forgerock/opendj/bin/dskeymgr create-deployment-key --deploymentKeyPassword password --validity "10 years")
            export $DEPLOYMENT_KEY
            echo "Here's the deployment key: "
            echo "${DEPLOYMENT_KEY}"
            echo "\n---- Starting setup ... ---\n"
            /opt/forgerock/opendj/setup \
                --deploymentKey $DEPLOYMENT_KEY \
                --deploymentKeyPassword password \
                --rootUserDN uid=admin \
                --rootUserPassword str0ngAdm1nPa55word \
                --monitorUserPassword str0ngMon1torPa55word \
                --hostname ds.example.com \
                --adminConnectorPort 4444 \
                --ldapPort 1389 \
                --enableStartTls \
                --ldapsPort 1636 \
                --httpsPort 8443 \
                --profile am-cts \
                --set am-cts/amCtsAdminPassword:5up35tr0ng \
                --acceptLicense
            echo "\n--- Starting DS ---\n"
            /opt/forgerock/opendj/bin/start-ds
            # Export the keys for AM to use to connect
            echo "\n--- Exporting Keys ---\n"
            keytool -exportcert \
            -keystore /opt/forgerock/opendj/config/keystore \
            -storepass $(cat /opt/forgerock/opendj/config/keystore.pin) \
            -alias ssl-key-pair \
            -rfc \
            -file ds-cert.pem
            # move the cert to the shared mount 
            echo "\n--- Moving the keys to the shared mount ---\n"
            cp -R ds-cert.pem /vagrant_data/ds-cts-cert.pem
        SHELL
      end

      ## build DS user store 
      config.vm.define "vm3", autostart:false do |vm3|
        vm3.vm.box = "ubuntu/focal64"
        vm3.vm.hostname = "ds3.local"
        vm3.vm.network "private_network", type:"dhcp"
        vm3.vm.network "forwarded_port", guest: 1636, host: 3636
        vm3.vm.provision "shell", inline: <<-SHELL
            echo "\n--- Running Provision Script -----\n"
            groupadd ds_user
            useradd -s /bin/false -g ds_user -d /opt/forgerock/cfg ds_user
            echo "\n---- Setting up Staging area ---\n"
            sudo mkdir -p /opt/forgerock/staging 
            sudo cp /vagrant_data/DS* /opt/forgerock/staging/.
            sudo unzip /opt/forgerock/staging/DS* -d /opt/forgerock/staging/.
            sudo mv /opt/forgerock/staging/opendj /opt/forgerock/opendj
            chgrp -R ds_user /opt/forgerock/opendj
            chmod g+rwx /opt/forgerock/opendj
            chmod g+r /opt/forgerock
            chown -R ds_user /opt/forgerock/opendj
            echo "\n---- Creating Deployment Key ---\n"
            DEPLOYMENT_KEY=$(/opt/forgerock/opendj/bin/dskeymgr create-deployment-key --deploymentKeyPassword password --validity "10 years")
            export $DEPLOYMENT_KEY
            echo "Here's the deployment key: "
            echo "${DEPLOYMENT_KEY}"
            echo "\n---- Starting setup ... ---\n"
            /opt/forgerock/opendj/setup \
                --deploymentKey $DEPLOYMENT_KEY \
                --deploymentKeyPassword password \
                --rootUserDN uid=admin \
                --rootUserPassword str0ngAdm1nPa55word \
                --monitorUserPassword str0ngMon1torPa55word \
                --hostname ds.example.com \
                --adminConnectorPort 4444 \
                --ldapPort 1389 \
                --enableStartTls \
                --ldapsPort 1636 \
                --httpsPort 8443 \
                --profile am-identity-store \
                --set am-identity-store/amIdentityStoreAdminPassword:5up35tr0ng \
                --acceptLicense
            echo "\n--- Starting DS ---\n"
            /opt/forgerock/opendj/bin/start-ds
            # Export the keys for AM to use to connect
            echo "\n--- Exporting Keys ---\n"
            keytool -exportcert \
            -keystore /opt/forgerock/opendj/config/keystore \
            -storepass $(cat /opt/forgerock/opendj/config/keystore.pin) \
            -alias ssl-key-pair \
            -rfc \
            -file ds-cert.pem
            # move the cert to the shared mount 
            echo "\n--- Moving the keys to the shared mount ---\n"
            cp -R ds-cert.pem /vagrant_data/ds-user-cert.pem
        SHELL
      end

      ## build AM instance
      config.vm.define "vm4" do |vm4|
        vm4.vm.box = "ubuntu/focal64"
        vm4.vm.hostname = "am1.local"
        vm4.vm.network "private_network", type:"dhcp"
        vm4.vm.network "forwarded_port", guest: 8080, host: 8080
        vm4.vm.provision "shell", inline: <<-SHELL
            echo "\n----- Installing Tomcat ------\n"
            groupadd tomcat
            useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
            wget -q https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz
            mkdir /opt/tomcat
            tar xvf apache-tomcat-*tar.gz -C /opt/tomcat --strip-components=1
            chgrp -R tomcat /opt/tomcat
            chmod g+rwx /opt/tomcat
            chmod g+r /opt/tomcat/*
            chown -R tomcat /opt/tomcat /opt/tomcat /opt/tomcat
            wget -c https://gist.githubusercontent.com/bradtumy/b0a528b948cb6b3a7a120966f77ffff1/raw/da579001a4ad3eb42cbb8d374101c1635e3a4e70/tomcat.service
            sudo mv tomcat.service /etc/systemd/system/tomcat.service
            sed -i "s#</tomcat-users>#  <user username=\\"admin\\" password=\\"secret\\" roles=\\"manager-gui,admin-gui\\"/>\\n</tomcat-users>#" /opt/tomcat/conf/tomcat-users.xml
            sudo systemctl daemon-reload
            sudo systemctl enable --now tomcat
            sudo systemctl status tomcat
            sudo mkdir -p /opt/forgerock/staging
            sudo cp /vagrant_data/AM* /opt/forgerock/staging/.
            sudo unzip /opt/forgerock/staging/AM-7* -d /opt/forgerock/staging/.
            sudo cp -R /opt/forgerock/staging/AM-7*.war /opt/forgerock/staging/sso.war
            sudo mv /opt/forgerock/staging/sso.war /opt/tomcat/webapps/.
            echo "\n----- Importing Keys into AM's Keystore ------\n"
            echo "\n----- Setup AM's Keystore ----\n"
            sudo mkdir -p /opt/forgerock/keys
            sudo cp /usr/lib/jvm/java-11-openjdk-amd64/lib/security/cacerts /opt/forgerock/keys/.
            sudo chown -R tomcat /opt/forgerock/keys
            sudo keytool -importcert -file /vagrant_data/ds-cfg-cert.pem -alias cfg-cert -keystore /opt/forgerock/keys/cacerts -storepass changeit -noprompt
            sudo keytool -importcert -file /vagrant_data/ds-cts-cert.pem -alias cts-cert -keystore /opt/forgerock/keys/cacerts -storepass changeit -noprompt
            sudo keytool -importcert -file /vagrant_data/ds-user-cert.pem -alias user-cert -keystore /opt/forgerock/keys/cacerts -storepass changeit -noprompt
            echo "\n----- Export CATALINA OPTS ----\n"
            export CATALINA_OPTS="$CATALINA_OPTS -Djavax.net.ssl.trustStore=/opt/forgerock/keys/cacerts -Djavax.net.ssl.trustStorePassword=changeit -Djavax.net.ssl.trustStoreType=jks"
            sudo systemctl stop tomcat
            sudo systemctl start tomcat
        SHELL
      end

      ## build IG instance
      config.vm.define "vm5" , autostart:false do |vm5|
        vm5.vm.box = "ubuntu/focal64"
        vm5.vm.hostname = "ig1.local"
        vm5.vm.network "private_network", type:"dhcp"
        vm5.vm.network "forwarded_port", guest: 8080, host: 9080
        vm5.vm.provision "shell", inline: <<-SHELL
            echo "\n----- Installing Tomcat ------\n"
            groupadd tomcat
            useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
            wget -q https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz
            mkdir /opt/tomcat
            tar xvf apache-tomcat-*tar.gz -C /opt/tomcat --strip-components=1
            chgrp -R tomcat /opt/tomcat
            chmod g+rwx /opt/tomcat
            chmod g+r /opt/tomcat/*
            chown -R tomcat /opt/tomcat /opt/tomcat /opt/tomcat
            wget -c https://gist.githubusercontent.com/bradtumy/b0a528b948cb6b3a7a120966f77ffff1/raw/da579001a4ad3eb42cbb8d374101c1635e3a4e70/tomcat.service
            sudo mv tomcat.service /etc/systemd/system/tomcat.service
            sed -i "s#</tomcat-users>#  <user username=\\"admin\\" password=\\"secret\\" roles=\\"manager-gui,admin-gui\\"/>\\n</tomcat-users>#" /opt/tomcat/conf/tomcat-users.xml
            sudo systemctl daemon-reload
            sudo systemctl enable --now tomcat
            sudo systemctl status tomcat
            sudo mkdir -p /opt/forgerock/staging 
            sudo cp /vagrant_data/IG* /opt/forgerock/staging/.
            sudo unzip /opt/forgerock/staging/IG* -d /opt/forgerock/staging/.
            sudo cp -R /opt/forgerock/staging/*.war /opt/forgerock/staging/ROOT.war
            sudo rm -R /opt/tomcat/webapps/ROOT*
            sudo mv /opt/forgerock/staging/ROOT.war /opt/tomcat/webapps/.
            sudo systemctl stop tomcat
            sudo systemctl start tomcat
        SHELL
      end
    end
