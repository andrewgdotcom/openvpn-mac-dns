# openvpn-mac-dns
Manage DNS settings pushed from OpenVPN

This script is the Mac equivalent of Debian's /etc/openvpn/update-resolv-conf .
It is based on the same code, but uses `networksetup` instead of `resolvconf` to perform the work.
The name is therefore slightly misleading (it does not touch /etc/resolv.conf),
but using the same name lets us write OS-agnostic "up" and "down" directives in the client configuration.

First, enable DNS push on your OpenVPN or OpenVPN-AS server.
Many modern clients will understand this natively, and if so you do not need this script.
To do so, add the following custom configuration to the server (edit to taste):

```
push "dhcp-option DNS 192.168.1.1"
push "dhcp-option DOMAIN my.domain"
```

If the above does not work, you must also copy update-resolv-conf to /etc/openvpn/update-resolv-conf on the client Mac.
If /etc/openvpn does not exist on the client, create it first using `sudo mkdir -p /etc/openvpn` .
Make sure that /etc/openvpn/update-resolv-conf is executable.

You should then add "up" and "down" directives in each openvpn client configuration file as follows:

```
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

Note that you will need to add these directives to every openvpn config file that you use.
There is no way to set a global default; this is a limitation of openvpn.

See https://github.com/andrewgdotcom/openvpn-mac-dns/issues/6 for troubleshooting help.

### OpenVPN-AS auto-generated client configuration

In OpenVPN Access Server, set the following option in the server configuration to automatically add the above snippet to auto-generated client files:

```
"vpn.client.config_text": "up /etc/openvpn/update-resolv-conf\ndown /etc/openvpn/update-resolv-conf", 
```

Or equivalently, incant the following on the server command line:

```
/usr/local/openvpn_as/scripts/confdba -mk "vpn.client.config_text" -v "up /etc/openvpn/update-resolv-conf\ndown /etc/openvpn/update-resolv-conf"
```

# Credits

Adapted from the original script here: https://github.com/masterkorp/openvpn-update-resolv-conf

networksetup invocation borrowed from https://blog.netnerds.net/2011/10/openvpn-update-client-dns-on-mac-os-x-using-from-the-command-line/

