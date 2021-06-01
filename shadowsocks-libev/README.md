# shadowsocks-libve

## 安装

```sh
pkg install -y shadowsocks-libev
```
## 插件

### cloak

编译

```
git clone https://github.com/cbeuw/Cloak.git
cd Cloak
# 取出最新的版本号
git tag | tail -1
go get ./...
# go build -ldflags "-X main.version=$(git tag | tail -1)" ./cmd/ck-server
env CGO_ENABLED=0 GOOS=freebsd GOARCH=amd64 go build -trimpath -ldflags " -s -w" -o ./ck-server ./cmd/ck-server
```

安装

```
cp ck-server /usr/local/bin/
chown 1000:1000 /usr/local/bin/ck-server
```

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

生成必要的id和keys

```
ck-server -u
ck-server -k && echo "PublicKey and PrivateKey"
```

ck-server.json

```
mkdir /usr/local/etc/cloak/
cat > /usr/local/etc/cloak/ck-server.json < EOF
{
    "ProxyBook":{
    "shadowsocks":["tcp","127.0.0.1:8880"]
    },
    "BindAddr":[":8443"],
    "BypassUID":[],
    "RedirAddr":"www.bing.com",
    "PrivateKey":"kPI6r8n7IiRTMhlXRzrGP0+TwmOJLdutLpETFjD7I3A=",
    "AdminUID":"RdKsWXAZX0hT1Ou+PWVWkA==",
    "DatabasePath":"/usr/local/etc/cloak/userinfo.db"
}
EOF
```

bug（需要给生成的 userinfo.db 修改一下所属用户，不然 service cloak start 会失败）

```
ck-server -c /usr/local/etc/cloak/ck-server.json
chown nobody /usr/local/etc/cloak/userinfo.db
```

客户端设置

```
Transport=direct;ProxyMethod=shadowsocks;EncryptionMethod=plain;UID=RdKsWXAZX0hT1Ou+PWVWkA==;PublicKey=y0lK3+F7Hi7Tn7oYjR+Kai1eb06SQaW6T1Gt8spFoQ8=;ServerName=www.bing.com;NumConn=4;BrowserSig=chrome;StreamTimeout=300
```

### gost
