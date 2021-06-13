# Linux 文本三剑客：sed


## 示例

判断文件中指定字符串行是否存在

```bash
sed -n '/^auth/p' /etc/pam.d/su | sed -n  '/sufficient/p' | sed -n '/pam_rootok.so/p
```

> 判断 `/etc/pam.d/su` 文件中是否有 `auth    required    pam_securetty.so` 配置行

修改行

```bash
sed -i.bak 's/PermitRoot*/PermitRoot no/g' /etc/ssh/sshd_config 
```

> -i.bak 在修改文件时会先备份，本例备份文件名为 sshd_config.bak

删除行

```bash
sed -i '/^PATH/d' /etc/profile
```

