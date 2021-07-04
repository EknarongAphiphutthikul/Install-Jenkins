# Jenkins

## Prepare System
- Multiple VM Ubuntu version 20.04 on Hyper-V   [set up hyper-v on windows10](https://github.com/EknarongAphiphutthikul/Install-Hyper-V)
- DNS Server  [set up](https://github.com/EknarongAphiphutthikul/Install-Dns-bind9)
- Update Package On Ubuntu 20.04
  ```sh
  sudo apt-get update
  ```
- Show hostname
  ```sh
  hostnamectl
  ```
- Set hostname
  ```sh
  sudo hostnamectl set-hostname jenkins.ake.com
  ```
- Show ip
  ```sh
  ifconfig
  ```
- Set ipv4 internal network (vEthernet-Internal-ME)
  - On cloud : you'll need to disable.
    ```sh
    sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```
    ```console
    network: {config: disabled}
    ```
  - Show file config in netplan
    ```sh
    ls /etc/netplan/
    ```
    ```console
    00-installer-config.yaml
    ```
  - Edit file config in netplan
    ```sh
    sudo nano /etc/netplan/00-installer-config.yaml
    ```
    ```console
    network:
      ethernets:
        eth0:
          dhcp4: false
          addresses:
            -  169.254.19.103/16
          nameservers:
            search: [ ake.com ]
            addresses:
              - 169.254.19.105
        eth1:
          dhcp4: true
      version: 2
    ```
  - Apply Config
    ```sh
    sudo netplan apply
    ```

- Set DNS (change default 127.0.0.53 to 169.254.19.105)  
  > **Important** : Workaround for  [Bug #1624320](https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1624320)
  ```sh
  sudo rm -f /etc/resolv.conf
  sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
  sudo reboot
  ```
- Install Docker  
  https://github.com/EknarongAphiphutthikul/Install-Docker

- Install Docker Compose
  ```sh 
  sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

  sudo chmod +x /usr/local/bin/docker-compose

  sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

  docker-compose --version
  ```
----

<br/>

## Install Jenkins
- Install Local Persist Volume Plugin  
  - https://unix.stackexchange.com/questions/439106/docker-create-a-persistent-volume-in-a-specific-directory  
  - https://github.com/MatchbookLab/local-persist  
  - https://stackoverflow.com/questions/63227362/docker-volumes-create-options-driver
  ```sh
  curl -fsSL https://raw.githubusercontent.com/MatchbookLab/local-persist/master/scripts/install.sh | sudo bash
  ```
- Create Volume
  ```sh
  docker volume create -d local-persist --opt mountpoint=/home/akeadm/jenkins/data --name jenkins-data-volume

  docker volume create -d local-persist --opt mountpoint=/home/akeadm/jenkins/cert --name jenkins-cert-volume
  ```
- Create certificate  
  [How to create certificates](https://github.com/EknarongAphiphutthikul/OpenSSL-Certificate-Authority)  
  ```sh
  mkdir /home/akeadm/jenkins/cert
  ```
  copy jenkins.key.pem jenkins.cert.pem to /home/akeadm/temp
  ```sh
  cd /home/akeadm/temp

  openssl rsa -in jenkins.key.pem -out /home/akeadm/jenkins/cert/key.pem -passin pass:changeit
  
  mv jenkins.cert.pem /home/akeadm/jenkins/cert/cert.pem

  rm /home/akeadm/temp/*
  ```
- Copy config  
  copy log.properties to /home/akeadm/jenkins/data/log.properties

- Install Jenkins  
  ```sh
  docker container run --name jenkins \
   --detach --restart unless-stopped \
   --env JENKINS_OPTS="-Djava.util.logging.config.file=/var/jenkins_home/log.properties --httpPort=-1 --httpsPort=2376 --httpsCertificate=/certs/client/cert.pem --httpsPrivateKey=/certs/client/key.pem" \
   --volume jenkins-cert-volume:/certs/client:ro \
   --volume jenkins-data-volume:/var/jenkins_home:rw \
   --publish 2376:2376 --publish 50000:50000 \
   jenkinsci/blueocean:1.25.0-alpha-1-bcc31d32159f

  docker ps
  ```
  ```console
  CONTAINER ID   IMAGE                                             COMMAND                  CREATED              STATUS              PORTS                                                                                                NAMES
  a997bb301f30   jenkinsci/blueocean:1.25.0-alpha-1-bcc31d32159f   "/sbin/tini -- /usr/…"   About a minute ago   Up About a minute   0.0.0.0:2376->2376/tcp, :::2376->2376/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp, 8080/tcp   jenkins
  ```
- Enable firewall
  ```sh
  sudo ufw allow 2376/tcp
  sudo ufw allow 50000/tcp
  sudo ufw reload
  sudo ufw status
  ```
- Administrator password
  ```sh
   docker exec jenkins  more /var/jenkins_home/secrets/initialAdminPassword
  ```