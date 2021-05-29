# FreeBSD 相关

[FreeBSD 与 RISC-V: 开源物联网生态系统的未来](https://feng.si/posts/2019/06/freebsd-and-risc-v-the-future-of-open-source-iot-ecosystem/)

## 开始

### 基本设置

```sh
# DHCP
sysrc ifconfig_vmx0=DHCP
# static
sysrc ifconfig_vmx0="inet 172.16.32.61 netmask 255.255.255.0"
# gateway
sysrc defaultrouter="172.16.32.1"
# disable sendmail
sysrc sendmail_enable=NONE
sysrc sendmail_submit_enable=NO
sysrc sendmail_outbound_enable=NO
sysrc sendmail_msp_queue_enable=NO
# enable sshd
sysrc sshd_enable=YES
```

### csh 配置

~/.tcshrc

```sh
set path = (/sbin /bin /usr/sbin /usr/bin /usr/games /usr/local/sbin /usr/local/bin $HOME/bin $HOME/work /usr/local/go/bin)

set prompt = "[%N@%m:%~]%# "

# history

set history = 100
set savehist = 100

# autocomplete
set autolist
set complete = enhance
set autoexpand
set correct = cmd
set correct = all

alias vim 'nvim'
alias ls 'ls --color=always'
alias l 'ls -l'
alias mv 'mv -i'
alias cp 'cp -i'
alias rm 'rm -i'

setenv GREP_OPTIONS --color=auto
```

### sshd

```sh
# 允许 root ssh 远程
sed -e 's|^#PermitRootLogin no|PermitRootLogin yes|g' -i.bak /etc/ssh/sshd_config
service sshd restart
```

### 配置第三方非官方源（FreeBSD 官方源不支持镜像）

```sh
mkdir -p /usr/local/etc/pkg/repos
cat >> /usr/local/etc/pkg/repos/FreeBSD.conf << EOF
FreeBSD: {
  url: "pkg+http://mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/quarterly",
}
EOF
pkg update -f
pkg upgrade
```

### ports

```
portsnap extract
portsnap fetch
portsnap update
```

## 用户

### 新增

```sh
pw useradd -m -s /bin/sh -n skylens
```

### 删除

```sh
pw userdel -r -n skylens
```

### sudo

```sh
pkg install -y sudo
pw groupmod wheel -m skylens
```

## 软件

### 安装必要软件

**!!! vim 包及相关的依赖太多，不建议安装，可以使用 neovim**

删除 vim

```sh
pkg remove `cat vimdeps.txt`
```

```sh
pkg install -y pkg wget git nano aria2 tmux neovim
```

### neovim 配置

```sh
mkdir -p ~/.config/nvim
cd ~/.config/nvim
cat > init.vim << EOF
set paste
colorscheme delek
EOF
```

### strongswan

[strongswan](strongswan/README.md)

### shadowsocks-libev

[shadowsocks-libev](shadowsocks-libev/README.md)

## 更新

### 小版本更新

```sh
# 查看当前版本
freebsd-version -k
# 更新
/usr/sbin/freebsd-update fetch
/usr/sbin/freebsd-update -r 12.2-RELEASE upgrade
/usr/sbin/freebsd-update install
reboot
/usr/sbin/freebsd-update install
```

### 大版本更新

如果有使用 bash 作为默认 shell，需要删除 bash （ pkg remove bash ），不然会报错

**freebsd 12 升级到 freebsd 13**

```sh
freebsd-version -k
/usr/sbin/freebsd-update -r 13.0-RELEASE upgrade
/usr/sbin/freebsd-update install
reboot
pkg-static install -f pkg
```
