---
title: Linux创建新的用户
date: 2024-06-02 22:39:08
tags: [Linux, 运维]
---

添加用户

```bash
adduser lyx
```

设置密码

```bash
echo "Dgut1234" | passwd --stdin lyx
```

添加sudo权限

```bash
# 添加sudoers文件写权限
chmod u+w /etc/sudoers
# 在sudoers文件中找到root ALL=(ALL) ALL，在下面加入下行代码，保存
lyx ALL=(ALL) ALL
# 移除sudoers文件写权限
chmod u-w /etc/sudoers
```

设置用户下次登录需要更改密码
```bash
chage -d0lyx
```