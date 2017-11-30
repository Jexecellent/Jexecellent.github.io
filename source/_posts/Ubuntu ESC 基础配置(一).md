---
title: 'Ubuntu ESC 基础配置(一)'
date: 
tags: ['Linux']
---

1. 修改服务器访问默认端口，防止服务器被恶意访问。
   
    steps:

      ``` 
      sudo  vi  /etc/ssh/sshd_config // 编辑配置文件，记录配置文件地址
      
      ```

     输入密码；这里注意重新打开一个窗口，经验之谈，防止忘记了无法登陆服务器。
     
     下面是配置文件

     ```
    # What ports, IPs and protocols we listen for
    Port 3389
    # Use these options to restrict which interfaces/protocols sshd will bind to
    #ListenAddress ::
    #ListenAddress 0.0.0.0
    Protocol 2
    # HostKeys for protocol version 2
    HostKey /etc/ssh/ssh_host_rsa_key
    HostKey /etc/ssh/ssh_host_dsa_key
    HostKey /etc/ssh/ssh_host_ecdsa_key
    HostKey /etc/ssh/ssh_host_ed25519_key
    #Privilege Separation is turned on for security
    UsePrivilegeSeparation yes
            
    # Lifetime and size of ephemeral version 1 server key
    KeyRegenerationInterval 3600
    ServerKeyBits 1024 
                
    # Logging
    LogLevel INFO
        
    # Authentication:
    LoginGraceTime 120
    StrictModes yes
        
    RSAAuthentication yes
    PubkeyAuthentication yes
    #AuthorizedKeysFile     %h/.ssh/authorized_keys

    # Don't read the user's ~/.rhosts and ~/.shosts files
    IgnoreRhosts yes
    # For this to work you will also need host keys in /etc/ssh_known_hosts
    RhostsRSAAuthentication no
    # similar for protocol version 2
    HostbasedAuthentication no
    # Uncomment if you don't trust ~/.ssh/known_hosts for RhostsRSAAuthentication
    #IgnoreUserKnownHosts yes

    # To enable empty passwords, change to yes (NOT RECOMMENDED)
    PermitEmptyPasswords no

    # Change to yes to enable challenge-response passwords (beware issues with
    # some PAM modules and threads)
    ChallengeResponseAuthentication no

    # Change to no to disable tunnelled clear text passwords

    #KerberosOrLocalPasswd yes
    #KerberosTicketCleanup yes

    # GSSAPI options
    #GSSAPIAuthentication no
    #GSSAPICleanupCredentials yes

    X11Forwarding yes
    X11DisplayOffset 10
    PrintMotd no
    PrintLastLog yes
    TCPKeepAlive yes
    #UseLogin no

    #MaxStartups 10:30:60
    #Banner /etc/issue.net

    # Allow client to pass locale environment variables
    AcceptEnv LANG LC_*

    Subsystem sftp /usr/lib/openssh/sftp-server

    # Set this to 'yes' to enable PAM authentication, account processing,
    # and session processing. If this is enabled, PAM authentication will
    # be allowed through the ChallengeResponseAuthentication and
    # PAM authentication via ChallengeResponseAuthentication may bypass
    # If you just want the PAM account and session checks to run without
    # and ChallengeResponseAuthentication to 'no'.
    UsePAM yes
    Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,blowfish-cbc,aes128-cbc,3des-cbc,cast128-cbc,arcfour,aes192-cbc,aes256-cbc
    UseDNS no
    AddressFamily inet
    PermitRootLogin no
    SyslogFacility AUTHPRIV
    PasswordAuthentication no
    AllowUsers bean

     ```

修改端口之后重启ssh服务，执行以下命令：

```
sudo service  ssh restart
```

重新打开一个终端窗口，登陆服务器;

2. nginx 转发配置

```
    upstream  wechat {
        server 127.0.0.1:8081
    }
    server {
        listen 80;
        server_name  wechat.kreditlab.com;

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forward-For $rpoxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_set_header X-Nginx-Proxy true;

            proxy_pass http://wechat;
            proxy_redirect off
        }
    }
```

防火墙基础配置

```
# 更新服务器，输入命令回车，输入密码
sudo apt-get update && sudo apt-get upgrade   

sudo iptables -F

# 新创建防火墙配置文件
sudo vi /etc/iptables.up.rules

```

进入 iptables.up.rules文件编辑

```
*filter

# allow all connections
-A INPUT -m STATE --state ESTABLISHED,RELATED -j ACCEPT

# allow out traffic 
-A OUTPUT -j ACCEPT

# allow http or https  允许 http 和 https 协议
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT

# allow ssh port login  允许 ssh登陆服务器
-A INPUT -p tcp -m state --state  NEW --dport  3389 -j ACCEPT

# ping 
-A INPUT  -p icmp -m icmp --icmp-type 8 -j ACCEPT 

# log denied calls 日志
-A INPUT -m  --limit 5/min -j LOG --log-prefix "iptables denied:" --log-level 7

# drop incoming sensitive connections 拦截恶意访问，如果60秒内访问80端口超过150次及拦截
-A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --set
-A INPUT -p tcp  --dport 80 -i eth0 -m state --state NEW -m recent --update --seconds 60 --hitcount 150 -j DROP 

# reject all other inbound 
-A INPUT -j REJECT
-A FORWARD -j REJECT

COMMIT

```

防火墙规则编辑完成后，执行命令确认防火墙配置文件位置

```
sudo iptables-restore < /etc/iptables.up.rules

```

建立成功后查看防火墙的状态是否成功启动

```
sudo ufw status   

>  Status:inactive  // 防火墙没有激活
```

激活防火墙

```
sudo ufw enable 

```

创建shell脚本，自动启用防火墙

```
sudo vi /etc/network/if-up.d/iptables   

```

写入脚本


```
#!/bin/sh
iptables-restore /etc/iptables.up.rules

```

给脚本配置执行的权限

```
sudo chmod +X /etc/network/if-up.d/iptables

```

 安装fail2ban安防模块

```
sudo apt-get install fail2ban

```

安装之后打开修改部分配置文件

```
sudo vi /etc/fail2ban/jail.conf

```

查看fail2ban 是否在运行

```
sudo service fail2ban  status

> fail2ban is running ...

```

开启/停止fail2ban 

```
sudo service fail2ban start/stop

```





























