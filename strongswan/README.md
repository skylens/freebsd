# strongswan


## 安装

```sh
pkg install strongswan
```

## iKEv2 EAP-mschapv2 Remote Access VPN 配置

### stroke

修改 /usr/local/etc/rc.d/strongswan rc.d 启动文件，将 strongswan_interface 参数改为 stroke

: ${strongswan_interface:="stroke"}

开机自启

```sh
sysrc strongswan_enable=YES
```

准备证书 (Let's Encrypt 证书为例)

```sh
cp vpn.domain.com/chain.pem /usr/local/etc/ipsec.d/cacerts/chain.pem
cp vpn.domain.com/fullchain.pem /usr/local/etc/ipsec.d/certs/fullchain.pem
cp vpn.domain.com/cert.pem /usr/local/etc/ipsec.d/certs/cert.pem
cp vpn.domain.com/privkey.pem /usr/local/etc/ipsec.d/private/privkey.pem
```

修改配置文件 /usr/local/etc/ipsec.conf

```
config setup
    uniqueids=never

conn %default
    left=%any
    leftsubnet=0.0.0.0/0
    right=%any
    rightsourceip=172.16.100.0/24
    dpdaction=clear

conn iKEv2-EAP-mschapv2
    keyexchange=ikev2
    ike=aes256-sha256-modp2048,3des-sha1-modp2048,aes256-sha1-modp2048!
    esp=aes256-sha256,3des-sha1,aes256-sha1!  
    rekey=no
    leftcert=fullchain.pem
    leftsendcert=always
    leftid=@vpn.domain.com
    rightid=%any
    rightauth=eap-mschapv2
    rightsendcert=never
    rightdns=8.8.8.8,8.8.8.4
    eap_identity=%identity
    fragmentation=yes
    auto=add
```

修改密码配置文件 /usr/local/etc/ipsec.secrets

```
: RSA "privkey.pem"
domain : EAP "domain.com"
```

启动服务

```
ipsec status
ipsec restart
```

故障排除

```
swanctl --log
```

### vici

修改 /usr/local/etc/rc.d/strongswan rc.d 启动文件，将 strongswan_interface 参数改为 vici

: ${strongswan_interface:="vici"}

开机自启

```sh
sysrc strongswan_enable=YES
```

准备证书 (Let's Encrypt 证书为例)

```sh
cp vpn.domain.com/chain.pem /usr/local/etc/swanctl/x509ca/chain.pem
cp vpn.domain.com/fullchain.pem /usr/local/etc/swanctl/x509/fullchain.pem
cp vpn.domain.com/cert.pem /usr/local/etc/swanctl/x509/cert.pem
cp vpn.domain.com/privkey.pem /usr/local/etc/swanctl/private/privkey.pem
```

修改配置文件 /usr/local/etc/swanctl/swanctl.conf

```
connections {
        iKEv2-EAP-mschapv2 {
                unique=never
                version=2
                dpd_delay=30s
                send_cert=always
                pools=pool-ipv4
                proposals=aes256-sha256-modp2048,3des-sha1-modp2048
                mobike=yes
                local {
                        id=vpn.domain.com
                        certs=fullchain.pem
                }
                remote {
                        auth=eap-mschapv2
                        eap_id=%any
                }
                children {
                        iKEv2-EAP-mschapv2 {
                                esp_proposals=aes256-sha256,3des-sha1
                                local_ts=0.0.0.0/0
                        }
                }
        }
}
pools {
        pool-ipv4 {
                addrs=172.16.100.0/24
                dns=8.8.8.8,8.8.4.4
        }
}
secrets {
        eap-domain {
                id=domain
                secret="domain.com"
        }
}
```

启动服务

```
service strongswan start
service strongswan status
```

故障排除

```
swanctl --log
```
