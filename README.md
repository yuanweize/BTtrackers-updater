# Aria2 BT Tracker 自动更新工具

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.6+](https://img.shields.io/badge/python-3.6+-blue.svg)](https://www.python.org/downloads/)

一个用于自动更新 Aria2 BT Tracker 列表的 Python 脚本，帮助保持 Aria2 的 BT Tracker 始终最新，提升下载的连通性和速度。

## ✨ 功能特性

- 🔄 **多源获取**: 从多个公开的 Tracker 列表源获取最新地址
- 🔗 **智能合并**: 自动合并现有配置和新获取的 Tracker，智能去重
- 💾 **安全备份**: 修改配置前自动备份原文件，保障安全
- ⭐ **RPC动态更新**: 支持通过Aria2 RPC接口动态更新tracker，无需重启程序
- ⚙️ **多种更新模式**: 支持配置文件更新、RPC更新、混合更新三种模式
- ⏰ **定时更新**: 使用 cron 定时任务，每天自动更新
- 📝 **完善日志**: 详细的日志记录和异常处理
- ⚙️ **灵活配置**: 支持配置文件自定义设置
- 🔁 **重试机制**: 支持重试机制，提高获取成功率
- ✅ **格式验证**: Tracker 格式验证，确保有效性

## 快速开始

### 一键安装（推荐）

```bash
# 使用 curl 一键安装
curl -fsSL https://raw.githubusercontent.com/yuanweize/BTtrackers-updater/main/install.sh | sudo bash
```

或者使用 wget：

```bash
# 使用 wget 一键安装
wget -qO- https://raw.githubusercontent.com/yuanweize/BTtrackers-updater/main/install.sh | sudo bash
```

## 安装和配置

### 环境要求

- Python 3.6+
- requests 库

### 其他安装方式

#### 方法1: Git 克隆

```bash
# 克隆项目
git clone https://github.com/yuanweize/BTtrackers-updater.git
cd BTtrackers-updater

# 运行安装脚本
sudo ./install.sh
```

#### 方法2: 下载压缩包

```bash
# 下载最新版本
wget https://github.com/yuanweize/BTtrackers-updater/archive/main.zip
unzip main.zip
cd BTtrackers-updater-main

# 运行安装脚本
sudo ./install.sh
```

#### 方法3: 手动安装

```bash
# 安装Python依赖
pip3 install requests

# 下载脚本文件
wget https://raw.githubusercontent.com/yuanweize/BTtrackers-updater/main/update_bt_trackers.py
wget https://raw.githubusercontent.com/yuanweize/BTtrackers-updater/main/config.json

# 给脚本添加执行权限
chmod +x update_bt_trackers.py
```

### 配置文件说明

编辑 `config.json` 文件来自定义设置：

```json
{
  "aria2_conf_path": "/opt/aria2/aria2.conf",     // Aria2 配置文件路径
  "backup_enabled": true,                          // 是否启用备份
  "backup_suffix": ".bak",                         // 备份文件后缀
  "tracker_sources": [                             // Tracker 源列表
    "https://trackerslist.com/all.txt",
    "https://ngosang.github.io/trackerslist/trackers_all.txt",
    "https://raw.githubusercontent.com/XIU2/TrackersListCollection/master/all.txt",
    "https://raw.githubusercontent.com/DeSireFire/animeTrackerList/master/AT_all.txt"
  ],
  "request_timeout": 10,                           // 请求超时时间（秒）
  "max_retries": 3,                               // 最大重试次数
  "log_level": "INFO",                            // 日志级别
  "log_file": "bt_tracker_update.log",            // 日志文件路径
  "rpc": {                                         // RPC 配置
    "enabled": false,                              // 是否启用 RPC 功能
    "url": "http://localhost:6800/jsonrpc",       // RPC 地址
    "secret": "",                                  // RPC 访问密钥（可选）
    "timeout": 10,                                // RPC 请求超时时间
    "verify_ssl": true                             // 是否验证 SSL 证书
  },
  "update_mode": "config",                         // 更新模式：config/rpc/hybrid
  "fallback_to_config": true                      // RPC失败时是否回退到配置文件更新
}
```

#### 更新模式说明

- **config**: 仅更新配置文件（默认模式，需要重启aria2生效）
- **rpc**: 仅通过RPC动态更新（实时生效，无需重启）
- **hybrid**: 同时更新配置文件和RPC（推荐）

#### RPC 配置步骤

1. **启用 Aria2 RPC 服务**

   在 aria2.conf 中添加或确认以下配置：
   ```
   # 启用RPC
   enable-rpc=true
   rpc-listen-all=true
   rpc-listen-port=6800
   # 设置RPC访问密钥（强烈推荐）
   rpc-secret=your_secret_here
   ```

2. **在本工具中启用RPC**

   修改 config.json：
   ```json
   {
     "rpc": {
       "enabled": true,
       "url": "http://localhost:6800/jsonrpc",
       "secret": "your_secret_here"
     },
     "update_mode": "hybrid"
   }
   ```

## RPC 动态更新功能详解

### 功能优势

- **实时生效**: 无需重启aria2程序，tracker更新立即生效
- **不中断下载**: 现有下载任务不会受到影响
- **远程管理**: 可以远程更新aria2的tracker配置
- **安全可靠**: 支持密钥认证，确保操作安全

### 使用步骤

#### 1. 测试RPC连接

在使用RPC功能前，建议先测试连接：

```bash
python3 update_bt_trackers.py --test-rpc
```

成功的话会显示类似输出：
```
2025-09-22 04:03:44 - INFO - 成功连接到Aria2 RPC，版本: 1.36.0
2025-09-22 04:03:44 - INFO - === Aria2 RPC连接信息 ===
2025-09-22 04:03:44 - INFO - 版本: 1.36.0
2025-09-22 04:03:44 - INFO - 功能: Async DNS, BitTorrent, Firefox3 Cookie, GZip, HTTPS, Message Digest, Metalink, XML-RPC, SFTP
2025-09-22 04:03:44 - INFO - RPC URL: http://localhost:6800/jsonrpc
2025-09-22 04:03:44 - INFO - 当前bt-tracker数量: 15
2025-09-22 04:03:44 - INFO - === 连接测试成功 ===
```

#### 2. 使用RPC模式更新

```bash
# 使用配置文件设置（推荐）
python3 update_bt_trackers.py

# 仅使用RPC更新
python3 update_bt_trackers.py --update-mode rpc

# 指定RPC参数
python3 update_bt_trackers.py --rpc-url http://localhost:6800/jsonrpc --rpc-secret mysecret

# 混合模式（同时更新配置文件和RPC）
python3 update_bt_trackers.py --update-mode hybrid
```

### 更新模式详解

- **config**: 仅更新配置文件（默认模式，需要重启aria2生效）
- **rpc**: 仅通过RPC更新（实时生效，无需重启）
- **hybrid**: 同时更新配置文件和RPC（推荐，既保证实时生效，又确保重启后配置不丢失）

### RPC 故障排除

#### 1. RPC连接失败

**错误信息：** `无法连接到Aria2 RPC服务`

**解决方法：**
1. 检查aria2是否正在运行：`ps aux | grep aria2`
2. 检查RPC配置：`grep -E "(enable-rpc|rpc-listen|rpc-secret)" /opt/aria2/aria2.conf`
3. 检查端口是否开放：`netstat -tlnp | grep 6800`

#### 2. 认证失败

**错误信息：** `RPC Error 1: Unauthorized`

**解决方法：**
1. 检查配置文件中的secret是否与aria2.conf中的rpc-secret一致
2. 确保RPC密钥不包含特殊字符或空格

#### 3. RPC功能未启用

**错误信息：** `RPC功能未启用，请检查配置文件`

**解决方法：**
1. 在config.json中设置 `"rpc": {"enabled": true}`
2. 或使用命令行参数 `--rpc` 启用

## 使用方法

### 验证安装

安装完成后，可以验证是否正常工作：

```bash
# 测试运行（预览模式，不会修改文件）
python3 update_bt_trackers.py --dry-run

# 查看帮助信息
python3 update_bt_trackers.py --help

# 列出所有tracker源
python3 update_bt_trackers.py --list-sources
```

### 手动运行

```bash
# 基本用法
python3 update_bt_trackers.py

# 使用自定义配置文件
python3 update_bt_trackers.py -c /path/to/custom.json

# 指定 aria2 配置文件路径
python3 update_bt_trackers.py --aria2-conf /path/to/aria2.conf

# 详细输出模式
python3 update_bt_trackers.py -v
```

### 命令行选项

```bash
# 查看帮助
python3 update_bt_trackers.py --help

# 使用自定义配置文件
python3 update_bt_trackers.py -c /path/to/custom.json

# 预览模式（不实际修改文件）
python3 update_bt_trackers.py --dry-run

# 指定 aria2 配置文件路径
python3 update_bt_trackers.py --aria2-conf /path/to/aria2.conf

# 详细输出模式
python3 update_bt_trackers.py -v

# 列出所有 tracker 源
python3 update_bt_trackers.py --list-sources

# RPC 相关选项
python3 update_bt_trackers.py --test-rpc                     # 测试RPC连接
python3 update_bt_trackers.py --rpc                          # 启用RPC模式
python3 update_bt_trackers.py --rpc-url http://localhost:6800/jsonrpc  # 指定RPC地址
python3 update_bt_trackers.py --rpc-secret mysecret          # 指定RPC密钥
python3 update_bt_trackers.py --update-mode hybrid           # 指定更新模式
```

### 设置定时任务

如果使用了install.sh安装脚本，会自动设置cron定时任务。

#### 查看定时任务

```bash
# 查看已设置的定时任务
sudo crontab -u aria2 -l

# 查看定时任务日志
tail -f /opt/bt-tracker-updater/bt_tracker_update.log
```

#### 手动设置 cron

如果需要自定义定时任务：

```bash
# 编辑 crontab
crontab -e

# 添加以下行（每天凌晨2点执行）
0 2 * * * /usr/bin/python3 /path/to/update_bt_trackers.py

# 或者每6小时执行一次
0 */6 * * * /usr/bin/python3 /path/to/update_bt_trackers.py
```

#### 常用 cron 时间设置

```bash
# 每天凌晨2点
0 2 * * *

# 每6小时
0 */6 * * *

# 每天上午8点和晚上8点
0 8,20 * * *

# 每周日凌晨3点
0 3 * * 0
```

对于RPC模式，建议增加定时任务的频率，因为更新不会中断下载：

```bash
# 每2小时更新一次（RPC模式）
0 */2 * * * /usr/bin/python3 /path/to/update_bt_trackers.py --update-mode hybrid

# 每30分钟更新一次（仅RPC）
*/30 * * * * /usr/bin/python3 /path/to/update_bt_trackers.py --update-mode rpc
```

## 日志和故障排除

### 日志查看

```bash
# 查看日志文件
tail -f bt_tracker_update.log

# 查看最近的日志
tail -n 50 bt_tracker_update.log

# 实时监控日志
watch -n 5 tail -n 20 bt_tracker_update.log
```

### 常见问题

1. **网络连接问题**
   - 检查服务器网络连接
   - 确认防火墙设置
   - 可以在配置文件中增加 `max_retries` 值

2. **配置文件格式错误**
   - 使用 JSON 验证工具检查 config.json 格式
   - 确保路径使用正确的斜杠

3. **权限问题**
   ```bash
   chmod +x update_bt_trackers.py
   chmod 644 config.json
   ```

### 测试运行

```bash
# 测试配置文件
python3 -c "import json; print(json.load(open('config.json')))"

# 预览模式（不实际修改文件）
python3 update_bt_trackers.py --dry-run
```

## 更新日志

- **v1.0**: 基础功能实现
- **v1.1**: 添加配置文件支持和增强日志系统
- **v1.2**: 添加重试机制和 Tracker 验证
- **v1.3**: 添加RPC动态更新功能，支持多种更新模式

## 许可证

MIT License