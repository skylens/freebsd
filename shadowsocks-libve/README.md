# shadowsocks-libve

## 安装

```sh
pkg install -y shadowsocks-live
```
## 插件

### cloak

rc.d

```
cat > /usr/local/etc/rc.d/cloak < EOF
#!/bin/sh

# PROVIDE: cloak
# REQUIRE: DAEMON NETWORKING
# KEYWORD: shutdown

# Add the following lines to /etc/rc.conf to enable cloak:
# cloak_enable : set to "YES" to enable the daemon, default is "NO"

. /etc/rc.subr

name=cloak
rcvar=cloak_enable

load_rc_config $name

cloak_enable=${cloak_enable:-"NO"}

logfile="/var/log/${name}.log"

procname=/usr/local/bin/ck-server
configfile=/usr/local/etc/cloak/ck-server.json
command="/usr/sbin/daemon"
command_args="-u nobody -o ${logfile} -t ${name} ${procname} -c ${configfile}"

run_rc_command "$1"
EOF

chmod 0755 /usr/local/etc/rc.d/cloak
```


ck-server.json

```
mkdir /usr/local/etc/cloak/
cat > /usr/local/etc/cloak/ck-server.json < EOF
{
    "ProxyBook":{
    "shadowsocks":["tcp","127.0.0.1:12802"]
    },
    "BindAddr":[":443",":80"],
    "BypassUID":[],
    "RedirAddr":"www.bing.com",
    "PrivateKey":"QMII9Z1ZA/Iye+pM8qbP/R6/2VgcQFADZBY1hYvAAXo=",
    "AdminUID":"1U2UndGrJiviGvH4gHJL8w==",
    "DatabasePath":" /usr/local/etc/cloak/userinfo.db"
}
EOF
```
