>突然觉得服务器ssh密码登录总是浪费一定量的时间，就想试试用sshKey进行登录。

1. 生成服务器sshkey和本地sshkey
```
$ ssh-keygen
```
2. 在服务器上生成一个authorized_keys文件，然后将本地public Key 存到生成的文件中。
3. 设置 /etc/ssh/sshd.config
```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile	.ssh/authorized_keys
```
4. 重启ssh服务
```
service sshd restart
```
5. 将~/.ssh 文件权限设置为其他用户不可读写。也就是700、600、400权限

以上这样全部设置完之后，发现还是无法ssh免密登录，name是为什么呢，我打开了openssh的官网开始寻找答案。

1. 我开始寻找ssh日志，查找问题所在
我们先看ssh客户端的日志
```
ssh root@xxxx -v
```
执行之后输出以下日志
```
OpenSSH_7.5p1, LibreSSL 2.5.4
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 52: Applying options for *
debug1: Connecting to 121.42.42.155 [121.42.42.155] port 22.
debug1: Connection established.
debug1: identity file /Users/yard/.ssh/id_rsa type 1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_rsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_dsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_dsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_ecdsa type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_ecdsa-cert type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_ed25519 type -1
debug1: key_load_public: No such file or directory
debug1: identity file /Users/yard/.ssh/id_ed25519-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_7.5
debug1: Remote protocol version 1.99, remote software version OpenSSH_5.3
debug1: match: OpenSSH_5.3 pat OpenSSH_5* compat 0x0c000000
debug1: Authenticating to 121.42.42.155:22 as 'root'
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: algorithm: diffie-hellman-group-exchange-sha256
debug1: kex: host key algorithm: ssh-rsa
debug1: kex: server->client cipher: aes128-ctr MAC: umac-64@openssh.com compression: none
debug1: kex: client->server cipher: aes128-ctr MAC: umac-64@openssh.com compression: none
debug1: SSH2_MSG_KEX_DH_GEX_REQUEST(2048<3072<8192) sent
debug1: got SSH2_MSG_KEX_DH_GEX_GROUP
debug1: SSH2_MSG_KEX_DH_GEX_INIT sent
debug1: got SSH2_MSG_KEX_DH_GEX_REPLY
debug1: Server host key: ssh-rsa SHA256:tp4/4hwf0Q39NLrFmXNC438rpEPBEqy4C+CXCDy91xE
debug1: Host '121.42.42.155' is known and matches the RSA host key.
debug1: Found key in /Users/yard/.ssh/known_hosts:1
debug1: rekey after 4294967296 blocks
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: rekey after 4294967296 blocks
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /Users/yard/.ssh/id_rsa
debug1: Authentications that can continue: publickey,password
debug1: Trying private key: /Users/yard/.ssh/id_dsa
debug1: Trying private key: /Users/yard/.ssh/id_ecdsa
debug1: Trying private key: /Users/yard/.ssh/id_ed25519
debug1: Next authentication method: password
```

我发现本机电脑提供了id_rsa文件去服务器认证，但是认证没有通过。

2. 我们再来看服务端的日志
我们重新开一个ssh的服务器端口
```
/usr/sbin/sshd -d -p 2222
```
然后再通过本机ssh去连接服务器
```
ssh root@xxxx -p 2222 -v
```
以下是服务器sshd日志
```
debug1: Server will not fork when running in debugging mode.
debug1: rexec start in 4 out 4 newsock 4 pipe -1 sock 7
debug1: inetd sockets after dupping: 3, 3
Connection from 122.234.57.180 port 50213
debug1: Client protocol version 2.0; client software version OpenSSH_7.5
debug1: match: OpenSSH_7.5 pat OpenSSH*
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-1.99-OpenSSH_5.3
debug1: permanently_set_uid: 74/74
debug1: list_hostkey_types: ssh-rsa,ssh-dss
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: client->server aes128-ctr umac-64@openssh.com none
debug1: kex: server->client aes128-ctr umac-64@openssh.com none
debug1: SSH2_MSG_KEX_DH_GEX_REQUEST received
debug1: SSH2_MSG_KEX_DH_GEX_GROUP sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_INIT
debug1: SSH2_MSG_KEX_DH_GEX_REPLY sent
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: KEX done
debug1: userauth-request for user root service ssh-connection method none
debug1: attempt 0 failures 0
debug1: PAM: initializing for "root"
debug1: PAM: setting PAM_RHOST to "122.234.57.180"
debug1: PAM: setting PAM_TTY to "ssh"
debug1: userauth-request for user root service ssh-connection method publickey
debug1: attempt 1 failures 0
debug1: test whether pkalg/pkblob are acceptable
debug1: temporarily_use_uid: 0/0 (e=0/0)
debug1: trying public key file /root/.ssh/authorized_keys
debug1: fd 4 clearing O_NONBLOCK
Authentication refused: bad ownership or modes for directory /root
debug1: restore_uid: 0/0
debug1: temporarily_use_uid: 0/0 (e=0/0)
debug1: trying public key file /root/.ssh/authorized_keys
debug1: fd 4 clearing O_NONBLOCK
Authentication refused: bad ownership or modes for directory /root
debug1: restore_uid: 0/0
Failed publickey for root from 122.234.57.180 port 50213 ssh2
```

我发现在认证的时候出现了这句话
```
 Authentication refused: bad ownership or modes for directory /root
```
这个错误提示的意识是/root文件夹的文件权限有问题
在google上查找这个错误提示后知道是因为~/.ssh文件需要只供root读写，其他用户都不可以用写的权限，然后导致/root/.ssh/authorized_keys无法读取，也就导致了认证不通过。
解决方法也很简单，给/root文件加上700权限就好了。

虽然写的很简单，但整个debug时间超过2小时，主要是因为openssh软件比较古老，资料相对难找。。。开始又懒得去看文档，导致没有找到正确的debug方式，一直在查关于sshconfig的问题。 找到正确debug方式之后，解决起来就非常快。
>PS：无论什么工具，debug一定要好好看文档。。。。
