# Samba-AD-DC
#samba AD DC server operates at a domain functional level of 2008R2 which is sufficient for enterprise level operation
Preparing server installation
=>sudo apt update && sudo apt upgrade
#
set server identity
=>/etc/hostname | hostnamectl set-hostname <hostname>
#
set static ip in the hosts file
=> e.g 10.10.0.5 dc.test.lab dc
#
Disable dhcp configurations and set static ip
=> in the file /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    ens33<ethernet interface>:
      dhcp4: no
      addresses: [10.10.0.5/24]
      gateway4: 10.10.0.1
      nameservers:
        addresses: [DNS-ip, <fallback e.g 8.8.8.8>]
  version: 2
 #
  #apply the network changes
  =>sudo netplan apply
 #
  #Install samba AD services
  become a root user
  => apt install -y samba winbind krb5-config smbclient dnsutils net-tools
 #
  #backup the original samba configuration file
  => mv /etc/samba/smb.conf /etc/samba/smb.conf.original
 #
  #Provision samba domain to setup samba domain information using the interactive mode
  =>samba-tool domain provision --use-rfc2307 --interactive
 #
  #set up samba DNS backend, this can be skipped if you provisioned DC using SAMBA_INTERNAL dns backend
  #set up the BIND DNS server and the BIND9_DLZ module. For details, see https://wiki.samba.org/index.php/Setting_up_a_BIND_DNS_Server
  #Start the BIND DNS server. For example:
   =>systemctl start named
  #
   #Configuring DNS Resolver
   #Domain members in an AD use DNS to locate services, such as LDAP and Kerberos. For that, they need to use a DNS server that is able to resolve the AD DNS zone.
   #On your DC, set the AD DNS domain in the search and the IP of your DC in the nameserver parameter of the /etc/resolv.conf file. For example:
     =>search test.lab
     =>nameserver 10.10.0.5
  #
     #Also, adjust dns settings in /etc/systemd/resolved.conf
      =>DNS=10.10.0.5
      =>FallbackDNS=8.8.8.8
      =>Domains=test.lab
  #
   #Configuring Kerberos
   #In an AD, Kerberos is used to authenticate users, machines, and services.
   #During the provisioning, Samba created a Kerberos configuration file for your DC. Copy this file to your operating system's Kerberos configuration
   =>cp /usr/local/samba/private/krb5.conf /etc/krb5.conf
  #
  #test samba
  =>samba  | netstat -antp | grep 'smbd|samba'
  
  
  
     
     
  
  
  
