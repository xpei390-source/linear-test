---
title: Journalctl
tags:
  - 日志分析
dateCreated: 2026-10-12 19:30:00
typecho_postid: "22"
---

## Journalctl
### 一、是什么
- **核心地位：**`systemd-journald`是现代Linux系统的中央日志管理中心，`journalctl`是调取与查看日志的统一查询终端
- **交互技巧：**默认调用`less`分页器，按`q`即可退出浏览
- 比传统grep /var/log/messages 更结构化，支持按服务、时间、优先级、启动周期过滤
### 二、常用参数

| **参数**               | **说明**                                              | **示例**                         |
| ---------------------- | ----------------------------------------------------- | -------------------------------- |
| **`-x`**               | 附带一行注释                                          | `journalctl -xe`                 |
| **`-u`**               | 按服务过滤                                            | `journalctl -u nginx`            |
| **`-n` / `--lines`**   | 显示最新指定行数的日志（默认 10 行）                  | `journalctl -n 50`               |
| **`-f` / `--follow`**  | 实时追踪并滚动输出最新日志（类似于 `tail -f`）        | `journalctl -f`                  |
| **`-r` / `--reverse`** | 反序排列，从最新日志开始倒序显示                      | `journalctl -r`                  |
| **`-b`**               | 本次启动日志                                          | `journalctl -b`                  |
| **`-k`**               | 只看内核日志                                          | `journalctl -k`                  |
| **`-p`**               | 按优先级过滤                                          | `journalctl -p err`              |
| **`--since/--until`**  | 按时间过滤                                            | `journalctl --since "1 hour ago"` |
| **`--no-pager`**       | 直接输出到终端，不调用 `less` 分页器（适合脚本/管道）**慎用** | `journalctl --no-pager`          |
| **`-o json-pretty`**   | JSON格式输出                                          | `journalctl -o json-pretty`      |
| **`--disk-usage`**     | 查看日志占用空间                                      | `journalctl --disk-usage`        |
| **`--vacuum-time`**    | 按时间清理日志                                        | `journalctl --vacuum-time=30d`   |
| **`--vacuum-size`**    | 按大小清理日志                                        | `journalctl --vacuum-size=500M`  |
| **`--no-tail`**        | 显示所有日志行（即便被截断）                          | `journalctl -no-tail`            |

#### 1.基础输出与显示控制

| **参数**               | **说明**                                              | **示例**                |
| ---------------------- | ----------------------------------------------------- | ----------------------- |
| **`-x`**               | 附带一行注释 查看所有日志（通过 `less`）              | `journalctl -xe`        |
| **`-n` / `--lines`**   | 显示最新指定行数的日志（默认 10 行）                  | `journalctl -n 50`      |
| **`-f` / `--follow`**  | 实时追踪并滚动输出最新日志（类似于 `tail -f`）        | `journalctl -f`         |
| **`-r` / `--reverse`** | 反序排列，从最新日志开始倒序显示                      | `journalctl -r`         |
| **`--no-pager`**       | 直接输出到终端，不调用 `less` 分页器（适合脚本/管道） | `journalctl --no-pager` |
| **`--no-tail`**        | 显示所有日志行（即便被截断）                          | `journalctl -a`         |

#### 2.按时间范围筛选

| **参数**             | **说明**                                                     | **示例**                                                     |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`-b` / `--boot`**  | 查看指定启动周期的日志（`0` 当前启动，`-1` 上一次，依此类推） | `journalctl -b` 或 `journalctl -b -1`                        |
| **`--list-boots`**   | 列出所有记录的历史启动记录及偏移编号                         | `journalctl --list-boots`                                    |
| **`-S` / `--since`** | 过滤起始时间（支持相对时间与具体时间格式）                   | `journalctl --since "2026-10-10 08:00:00"` `journalctl --since "1 hour ago"` `journalctl --since "yesterday"` |
| **`-U` / `--until`** | 过滤截止时间                                                 | `journalctl --until "2026-10-10 09:00:00"` `journalctl --until "10 min ago"` |

> 组合技巧：
>
> ```bash
> journalctl --since "2026-10-10" --until "2026-10-10 12:00"
> ```

#### 3.按服务、进程与组件筛选

| **参数 / 语法**       | **说明**                               | **示例**                      |
| --------------------- | -------------------------------------- | ----------------------------- |
| **`-u` / `--unit`**   | 过滤特定 systemd 服务的日志            | `journalctl -u nginx.service` |
| **`_PID=`**           | 按进程 ID 过滤                         | `journalctl _PID=1234`        |
| **`_UID=`**           | 按系统用户 ID 过滤                     | `journalctl _UID=1000`        |
| **`-k` / `--dmesg`**  | 仅显示内核日志（等同于增强版 `dmesg`） | `journalctl -k`               |
| **`/path/to/binary`** | 按可执行程序绝对路径过滤               | `journalctl /usr/sbin/sshd`   |

#### 4.按日志级别筛选 -p

| **级别** | **标识名称** | **含义**                 |
| -------- | ------------ | ------------------------ |
| **0**    | `emerg`      | 系统不可用               |
| **1**    | `alert`      | 必须立即处理             |
| **2**    | `crit`       | 关键故障                 |
| **3**    | `err`        | 运行时错误（排错最常用） |
| **4**    | `warning`    | 警告信息                 |
| **5**    | `notice`     | 正常但值得注意的信息     |
| **6**    | `info`       | 普通信息（默认常见级别） |
| **7**    | `debug`      | 调试日志                 |

- 示例：
- - 查看错误及更严重的日志：`journalctl -p err`(会自动包括0-3级)
  - 查看特定级别区间（如warning到err）：`journalctl -p 3..4`
#### 5.日志容量与维护清理

| **命令 / 参数**                   | **说明**                                 |
| --------------------------------- | ---------------------------------------- |
| `journalctl --disk-usage`         | 查看当前日志占用的磁盘空间大小           |
| `journalctl --vacuum-size=500M`   | 清理日志，保留最近总计 500MB 的文件      |
| `journalctl --vacuum-time=2weeks` | 清理保留 2 周以内的日志，删除更早的数据  |
| `journalctl --verify`             | 校验日志文件的完整性（检查是否存在损坏） |

#### 6.输出格式控制

| 参数                 | 说明       | 实例                    |
| -------------------- | ---------- | ----------------------- |
| **`-o short`**       | 默认格式   | `journalctl -o short`   |
| **`-o json`**        | JSON格式   | `journalctl -o json`    |
| **`-o json-pretty`** | 美化JSON   | `journalctl -o json`    |
| **`-o verbose`**     | 详细字段   | `journalctl -o verbose` |
| **`-o cat`**         | 仅消息内容 | `journalctl -o cat`     |

示例输出： **-o verbose**
**作用**：输出每条日志的**所有可用字段（元数据）**，每行一个字段，结构化展示。
```
Mon 2024-01-01 12:00:00.123456 UTC [s=1234;i=5678;b=abcd...]
    _UID=0
    _GID=0
    _PID=1234
    _COMM=sshd
    _EXE=/usr/sbin/sshd
    _SYSTEMD_UNIT=sshd.service
    MESSAGE=Accepted password for root from 192.168.1.1 port 22 ssh2
    PRIORITY=6
    _HOSTNAME=myserver
    ...
```
**-o cat** 
**作用**：**只输出日志的消息本体（MESSAGE）**，去掉所有元数据（时间戳、主机名、进程名、PID等）。
**特点**：
- 输出极其干净，每行就是一条日志的实际内容。
- 不会显示优先级、时间、来源等任何前缀。
- 相当于 `journalctl -o short` 去掉所有头部的简化版。

```
Accepted password for root from 192.168.1.1 port 22 ssh2
Failed password for invalid user admin from 192.168.1.2 port 12345 ssh2
Started Session 1 of user root.
```
### 三、常用速查表

| **场景分类**  | **常用命令**                      | **作用与说明**                                   |
| ------------- | --------------------------------- | ------------------------------------------------ |
| **基础浏览**  | `journalctl`                      | 正序查看所有系统历史日志                         |
|               | `journalctl -r`                   | **倒序查看**（最新日志排在最顶端）               |
|               | `journalctl -f`                   | **实时追踪日志**（类似tail -f）                  |
|               | `journalctl -n 20`                | **限制行数**（仅显示最新的 20 条日志）           |
| **精准排错**  | `journalctl -u <service>`         | **按服务单元过滤**（如 `-u nginx`）              |
|               | `journalctl -p err`               | **按级别过滤**（仅显示 `err` 及以上严重报错）    |
|               | `journalctl --since "1 hour ago"` | **时间范围过滤**（支持自然语言或标准时间戳）     |
| **实时追踪**  | `journalctl -u <service> -f`      | **动态追踪**（类似 `tail -f`，实时输出最新日志） |
| **启动周期**  | `journalctl -b`                   | 查看**本次开机**以来的所有日志                   |
|               | `journalctl -b -1`                | 查看**上一次引导周期**（关机/崩溃前）的日志      |
| **空间维护**  | `journalctl --disk-usage`         | 查看日志占用的磁盘空间                           |
|               | `journalctl --vacuum-size=500M`   | 清理旧日志，仅保留指定容量（如 500MB）           |
|               | `journalctl --vacuum-time=2weeks` | 清理旧日志，仅保留指定时间范围（如 2 周）        |
| 进程/用户过滤 | `journalctl _PID=1234`            | 按进程PID号过滤                                  |
|               | `journalctl -UID=1000`            | 按进程UID号过滤                                  |

### 三、经典组合实战
- 排查某个服务今天崩溃的原因：

```bash
journalctl -u nginx.service --since today -p err --no-pager
```
- 监控服务实时输出：

```bash
journalctl -u docker.service -f -o short-iso
```
- 查看上一次死机/重启前的最后日志：

```bash
journalctl -b -1 -e
```
- 定位服务最近异常：

```bash
journalctl -u mysql -p err --since "30 minutes ago"
```
- 服务重启验证与联调：

```bash
journalctl -u payment.service -f
```
