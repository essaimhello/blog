---
title: FreeSWITCH logrotate 日志切割完整配置+测试步骤
tags: [logrotate]
categories: logs
author: Judy
cover: /images/default-cover.jpg
published: true
date: 2026-02-06 12:03:05
---

# FreeSWITCH logrotate 日志切割完整配置+测试步骤
## 一、配置说明
适配文件：`/usr/local/freeswitch/log/freeswitch.log`  
功能：**按天切割、昨天日期命名、保留30天、copytruncate不中断业务、自动清理过期日志**

---

## 二、正式配置步骤
### 步骤1：创建 logrotate 配置文件
```bash
vim /etc/logrotate.d/freeswitch
```

### 步骤2：写入完整配置
```nginx
# FreeSWITCH 主日志切割配置
/usr/local/freeswitch/log/freeswitch.log {
    daily
    rotate 30
    dateext
    dateyesterday
    dateformat .%Y%m%d
    copytruncate
    missingok
    notifempty
   # create 0644 root root
}
```

### 步骤3：权限赋权（logrotate 要求配置文件权限严格）
```bash
chmod 644 /etc/logrotate.d/freeswitch
chown root:root /etc/logrotate.d/freeswitch
```

---

## 三、关键参数极简说明
| 参数 | 作用 |
| --- | --- |
| daily | 每天执行一次切割 |
| rotate 30 | 只保留最近30天日志，自动删超期 |
| dateext + dateformat | 日志后缀 `20260514` 年月日格式 |
| dateyesterday | 切割后用**昨天日期**命名，行业标准 |
| copytruncate | 复制后清空原日志，**不用重启FS、不用发HUP信号** |
| missingok | 日志不存在不报错 |
| notifempty | 空日志不轮转 |
| create | 生成新日志权限0644，属主OPS_admin |


---

## 四、测试步骤（必做，避免上线翻车）
### 测试1：语法校验（检查配置有没有写错）
```bash
logrotate -d /etc/logrotate.d/freeswitch
```

+ `-d` 模拟执行、只打印过程**不实际切割**
+ 无报错、输出正常规则就是配置没问题

### 测试2：手动强制执行一次切割
```bash
logrotate -f /etc/logrotate.d/freeswitch
```

+ `-f` 强制立刻触发轮转

### 测试3：查看日志目录验证效果
```bash
ls -lh /usr/local/freeswitch/log/
```

正常会出现：

```plain
freeswitch.log          # 当前正在写入
freeswitch.log.20260513 # 带日期的切割日志
```

### 测试4：观察业务是否正常写日志
```bash
tail -f /usr/local/freeswitch/log/freeswitch.log
```

看是否持续输出日志，**无中断、不空白** 就正常。

---

## 五、logrotate 自动执行机制
系统默认每天定时执行：

+ CentOS：`/etc/cron.daily/logrotate`
+ 每天凌晨自动跑一遍，无需自己加 crontab

---

## 六、后续维护
1. 日志只会保留最近30天，超期自动删除
2. 不用手动清磁盘、不用写shell定时清理脚本
3. 如需改成保留7天/15天，只需改 `rotate 数字` 即可

需要我帮你把 `cti.log、da2.log` 也加进同一个 logrotate 配置里统一切割吗？

