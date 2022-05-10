# wireguard-openwrt

* Connect to the admin webpage for openwrt

* Go to the `System > Software`

* Choose `Update lists...`

* Install `wireguard-tools` `luci-app-wireguard`

* Create a text file with the following parameters:

```
WG_IF="vpn"
WG_SERV="SERVER_ADDRESS"
WG_PORT=""
WG_ADDR=""
WG_ADDR6="" # optional
WG_KEY=""
WG_PSK=""
WG_PUB=""
```

* Copy and paste the parameters in the OpenWRT shell, to keep them in memory
* Run the following commands to **configure the firewall**:
```
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci del_list firewall.wan.network="${WG_IF}"
uci add_list firewall.wan.network="${WG_IF}"
uci commit firewall
/etc/init.d/firewall restart
```
* **Configure the network** :
```
uci -q delete network.${WG_IF}
uci set network.${WG_IF}="interface"
uci set network.${WG_IF}.proto="wireguard"
uci set network.${WG_IF}.private_key="${WG_KEY}"
uci add_list network.${WG_IF}.addresses="${WG_ADDR}"
uci add_list network.${WG_IF}.addresses="${WG_ADDR6}"
```
* **Add VPN peers**:
```
uci -q delete network.wgserver
uci set network.wgserver="wireguard_${WG_IF}"
uci set network.wgserver.public_key="${WG_PUB}"
uci set network.wgserver.preshared_key="${WG_PSK}"
uci set network.wgserver.endpoint_host="${WG_SERV}"
uci set network.wgserver.endpoint_port="${WG_PORT}"
uci set network.wgserver.route_allowed_ips="1"
uci set network.wgserver.persistent_keepalive="25"
uci add_list network.wgserver.allowed_ips="0.0.0.0/0"
uci add_list network.wgserver.allowed_ips="::/0"
uci commit network
/etc/init.d/network restart
```
