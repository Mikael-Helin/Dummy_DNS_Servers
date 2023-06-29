# Dummy DNS Servers

Here we create dummy DNS servers which are not serving DNS. We will make 2 dummy DNS servers, the idea to have these, is to reserve their IPv6 addresses and have them available for other services that require them running before the other services themselves can start.

It is assumed you created the production user and its network

    su -
    useradd production
    passwd production
    loginctl enable-linger $(id -u production)
    su - production
    podman network create --subnet=fc01::/16 production-ipv6-network
    podman network create --subnet=192.168.1.0/24 production-ipv4-network

With buildah we create the temporary DNS image

    git clone git@github.com:Mikael-Helin/Dummy_DNS_Servers.git
    mv Dummy_DNS_Servers/production-dns/ .
    rm -rf Dummy_DNS_Servers
    cd ~/production-dns
    buildah bud -t localhost:5000/dns/bind__debian-bullseye:latest .

Prepare the run script in the same directory and create networks

    chmod +x run_production-dns

Become root to install the systemd script

    su -
    mv /home/production/production-dns/production-dns.service /etc/systemd/system/production-dns.service
    chown root:root /etc/systemd/system/production-dns.service
    systemctl daemon-reload
    systemctl enable production-dns.service
    reboot
