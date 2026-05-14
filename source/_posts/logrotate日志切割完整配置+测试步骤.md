---
title: FreeSWITCH logrotate 日志切割完整配置+测试步骤
tags: [logrotate, freeswitch, 日志管理]
categories: logs
author: Judy
cover: /images/default-cover.jpg
published: true
date: 2026-05-14 12:03:05
---

# FreeSWITCH logrotate 日志切割完整配置+测试步骤

---

## 📋 配置说明

**适配文件**：`/usr/local/freeswitch/log/freeswitch.log`  

**核心功能**：

- ✅ 按天自动切割
- ✅ 以昨天日期命名（行业标准）
- ✅ 保留最近30天日志
- ✅ **copytruncate** 不中断业务
- ✅ 自动清理过期日志

---

## 🛠️ 正式配置步骤

### 步骤1：创建 logrotate 配置文件

```bash
vim /etc/logrotate.d/freeswitch
```

### 步骤2：写入完整配置

```nginx
# ============================================
# FreeSWITCH 主日志切割配置
# 功能：按天切割 | 保留30天 | 自动清理过期日志
# ============================================
/usr/local/freeswitch/log/freeswitch.log {
    daily                  # 每天执行一次
    rotate 30              # 保留最近30天
    dateext                # 使用日期作为扩展名
    dateyesterday          # 用昨天日期命名（推荐）
    dateformat .%Y%m%d     # 日期格式：.20260514
    copytruncate           # 复制后清空原文件，不中断服务
    missingok              # 文件不存在时不报错
    notifempty             # 空文件不轮转
    # create 0644 root root  # 可选：创建新文件权限
    compress               # 压缩旧日志（可选）
}
```

### 步骤3：设置正确权限

```bash
chmod 644 /etc/logrotate.d/freeswitch
chown root:root /etc/logrotate.d/freeswitch
```

> **⚠️ 注意**：logrotate 对配置文件权限要求严格，必须设置为 `644` 且属主为 `root`。

---

## 📝 关键参数详解

| 参数组合 | 作用说明 |
| :--- | :--- |
| `daily` | 每天执行一次日志切割 |
| `rotate 30` | 只保留最近30天日志，超期自动删除 |
| `dateext + dateformat` | 日志文件后缀为 `20260514` 格式 |
| `dateyesterday` | 切割后用**昨天日期**命名（行业最佳实践） |
| `copytruncate` | 复制后清空原日志，**无需重启FS、无需发HUP信号** |
| `missingok` | 日志文件不存在时不报错 |
| `notifempty` | 空日志文件不进行轮转 |
| `create 0644 root root` | 轮转后生成新日志文件的权限设置 |
| `compress` | 对旧日志文件进行 gzip 压缩 |

---

## ✅ 测试步骤（必做！避免线上翻车）

### 测试1：语法校验

```bash
logrotate -d /etc/logrotate.d/freeswitch
```

**参数说明**：
- `-d`：**模拟执行**模式，只打印过程不实际切割
- 无报错、输出正常规则即为配置正确

### 测试2：手动强制执行切割

```bash
logrotate -f /etc/logrotate.d/freeswitch
```

**参数说明**：
- `-f`：强制触发轮转（不受时间限制）

### 测试3：验证切割效果

```bash
ls -lh /usr/local/freeswitch/log/
```

**正常输出示例**：

```plain
total 4.0K
-rw-r--r-- 1 freeswitch freeswitch 0 May 14 00:00 freeswitch.log          # 当前正在写入
-rw-r--r-- 1 freeswitch freeswitch 1.2M May 13 23:59 freeswitch.log.20260513  # 已切割日志
```

### 测试4：确认业务日志正常写入

```bash
tail -f /usr/local/freeswitch/log/freeswitch.log
```

**验证标准**：持续输出日志、**无中断、不空白**即为正常。

---

## 🕐 自动执行机制

系统默认每天定时执行 logrotate：

| 系统类型 | 定时任务路径 | 执行时间 |
| :--- | :--- | :--- |
| CentOS/RHEL | `/etc/cron.daily/logrotate` | 每天凌晨（由 cron 控制） |
| Debian/Ubuntu | `/etc/cron.daily/logrotate` | 每天凌晨 |

> **💡 提示**：无需手动添加 crontab，系统已默认配置。

---

## 🔧 后续维护

1. **自动清理**：日志只会保留最近30天，超期自动删除
2. **零运维**：不用手动清理磁盘、无需编写 shell 定时脚本
3. **灵活调整**：如需保留7天/15天，只需修改 `rotate` 后的数字即可

---

## 🔄 扩展配置（可选）

如需将 `cti.log`、`da2.log` 等其他日志也加入统一切割，可修改配置文件：

```nginx
# 多日志文件统一切割配置
/usr/local/freeswitch/log/freeswitch.log
/usr/local/freeswitch/log/cti.log
/usr/local/freeswitch/log/da2.log {
    daily
    rotate 30
    dateext
    dateyesterday
    dateformat .%Y%m%d
    copytruncate
    missingok
    notifempty
}
```

---

> 📌 **总结**：本配置实现了 FreeSWITCH 日志的自动化管理，无需人工干预，确保日志文件可控且不占用过多磁盘空间。

---

*原创文章，转载请注明出处*
