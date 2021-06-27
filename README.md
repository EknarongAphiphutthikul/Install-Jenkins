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
----

<br/>

## Install Jenkins