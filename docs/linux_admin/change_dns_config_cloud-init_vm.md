# How to change DNS servers on Ubuntu server
1. sudo -i
2. resolvectl status
3. sudo apt install openvswitch-switch-dpdk
4. sudo bash -c 'echo "network: {config: disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
5. sudo chmod 600 /etc/netplan/50-cloud-init.yaml  
6.  nano /etc/netplan/50-cloud-init.yaml  ## add name servers  
7. sudo netplan apply
8. systemctl restart kubelet
9. sudo reboot -f