# waydroid-server-manual
How to access a server, hosted on waydroid from host linux.

# Motivation
If you try to host some server in waydroid (for example, everyproxy, other proxies, samba, etc) you will be unable to access it from your host machine due to firewall settings.

# Manual
1. Disable NFT to use iptables.
   ```bash
   # disablig nft to use iptables
   # NFT="$(command -v nft)"
   NFT=""
   ```
   Note: it also fixes internet connection in waydroid if you have no internet in waydroid. See https://github.com/waydroid/waydroid/issues/143#issuecomment-1520857943
2. Allow inoming connections from host to the desired port and replies from this port to the host.
   ```bash
   start_iptables() {
    start_ipv6
    if [ -n "$LXC_IPV6_ARG" ] && [ "$LXC_IPV6_NAT" = "true" ]; then
        $IP6TABLES_BIN $use_iptables_lock -t nat -A POSTROUTING -s ${LXC_IPV6_NETWORK} ! -d ${LXC_IPV6_NETWORK} -j MASQUERADE
    fi
    $IPTABLES_BIN $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p udp --dport 67 -j ACCEPT
    $IPTABLES_BIN $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p tcp --dport 67 -j ACCEPT
    $IPTABLES_BIN $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p udp --dport 53 -j ACCEPT
    $IPTABLES_BIN $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p tcp --dport 53 -j ACCEPT
    # allow input traffic to proxy port 8080
    $IPTABLES_BIN $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p tcp --dport <desired_port> -j ACCEPT
    # allow output reply traffic from port 8080
    $IPTABLES_BIN $use_iptables_lock -I OUTPUT -o ${LXC_BRIDGE} -p tcp --sport <desired_port> -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

   ```
3. If something is interfering with routing on your waydroid, i.e. some vpn, you have to tell waydroid to route packets to host ip addr directly:

   3.1. Use waydroid image with root access to use `ip rule add`.

   3.2. Find out host address:
   ```
   > ifconfig | grep -C1 waydroid
   waydroid0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.240.1  netmask 255.255.255.0  broadcast 192.168.240.255
   ```
   The host ip is *192.168.240.1* for me.

   3.3. In waydroid (use adb or terminal emulator like tmux) add ip rule to route outgoing to host traffic through *main* table, which will result in sending through right interface.
   ```
   ip rule add from all to <host_ip_addr> table main
   ```
   Note: main table is shown by `ip route show table main` or just `ip route`.
