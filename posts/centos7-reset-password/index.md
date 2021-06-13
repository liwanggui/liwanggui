# CentOS 7 重置 root 密码


1. 重启开机 
2. 按 e 编辑启动选项

![](/images/40867598.png)

3. 编辑修改两处：`ro` 改为 `rw`, 在 `LANG=en_US.UFT-8` 后面添加 `init=/bin/bash`

![](/images/40906684.png)

4. 按 `Ctrl+X` 重启，并修改密码, 输入 `passwd root` 命令重置 `root` 密码
5. 由于 `selinux` 开启着的需要执行以下命令更新系统信息, 否则重启之后密码未生效

```bash
touch /.autorelabel
```

6. 重启系统 

```bash
exec /sbin/init
```
