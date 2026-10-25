# Network tuning

The selected configuration remains CUBIC with fq_codel, as recorded in the September 16 correction.

## TCP Congestion Control

### /etc/sysctl.d/40-network.conf

    #Select TCP CUBIC congestion control algorithm for LAN
	net.ipv4.tcp_congestion_control = cubic
	net.core.default_qdisc = fq_codel

	# Ubisoft fix from Universal Blue
     net.ipv4.tcp_mtu_probing = 1

	# Increase netdev receive queue
	# May help prevent losing packets
	net.core.netdev_max_backlog = 4096
[Link](https://www.reddit.com/r/linux_gaming/comments/10oc0dq/psa_for_people_having_trouble_connecting_to/)
[Link](https://blog.cloudflare.com/http-2-prioritization-with-nginx/)
[Link](https://github.com/CachyOS/CachyOS-Settings/blob/master/usr/lib/sysctl.d/70-cachyos-settings.conf)

### Alternative to evaluate

BBR/BBRv3 was listed as an alternative for lossy or high-latency connections, with CachyOS as a reference. The notes contain no controlled comparison establishing that it would improve Benchtop’s workloads. Keep it separate from the selected CUBIC setting.

The earlier BBR experiment used this pair; it is not an instruction to replace the default:

    net.ipv4.tcp_congestion_control = bbr
    net.core.default_qdisc = fq

## DNS

### /etc/NetworkManager/conf.d/dns.conf

	[main]
	dns=systemd-resolved

CachyOS-Settings ships the same setting (`usr/lib/NetworkManager/conf.d/dns.conf`).
