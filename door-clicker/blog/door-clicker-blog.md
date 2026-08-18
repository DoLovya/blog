# 基于 ESP8266 + MQTT 的智能门禁系统

> 一个完整的物联网门禁项目，实现远程开门、状态监控、设备管理等功能

## 一、项目简介

Door Clicker 是一个基于 ESP8266 的智能门禁控制系统，通过 MQTT 协议实现远程开门。用户可以通过手机浏览器一键开门，支持设备在线状态监控、舵机角度可调、管理员权限管理等功能。

### 核心特性

- 📱 **极简开门界面**：手机端一键开门，操作简单
- 🎛️ **Web 管理后台**：MQTT 配置、舵机参数调整、实时日志查看
- 📡 **MQTT 协议通信**：天然支持"内网穿透"，公网可控制内网设备
- 💓 **心跳机制**：设备定期上报状态，90 秒超时判定在线/离线
- 🔌 **自动重连**：MQTT 断线自动恢复，开门指令失败自动重试
- 🎚️ **舵机参数可调**：开门角度、延时均可配置
- 📝 **实时日志系统**：操作日志、错误日志分类查看
- 🔐 **配置页认证**：配置页面需登录访问

---

## 二、系统架构

![系统架构图](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/images/architecture-diagram.jpg)

### 数据流说明

1. **Web 浏览器 → Web Server**：通过 HTTP API 发送开门请求
2. **Web Server → MQTT Broker**：将命令发布到指定 Topic
3. **MQTT Broker → ESP8266**：推送到订阅该 Topic 的设备
4. **ESP8266 → 舵机**：控制 GPIO 信号执行开门动作

### 为什么用 MQTT？

MQTT 协议实现了"内网穿透"的效果：
- ESP8266 主动连接 MQTT Broker（出站连接，无需公网 IP）
- Web Server 作为 MQTT 客户端，通过 Broker 转发命令
- 设备可以在任何网络环境下工作

---

## 三、硬件准备

### 所需组件

| 组件 | 型号 | 说明 |
|------|------|------|
| 主控 | ESP8266 (Huzzah) | WiFi 通信 + 舵机控制 |
| 舵机 | SG90/MG996R | 门锁驱动 |
| MQTT Broker | EMQX/Mosquitto | 消息中转 |

### 硬件安装

舵机固定在门锁上后，只需将三根线（VCC、GND、SIGNAL）插入 ESP8266 的排针即可。

![硬件安装实物图](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/images/hardware-installation.jpg)

### 接线说明

| ESP8266 引脚 | 连接 |
|-------------|------|
| 3V3 | 舵机 VCC（建议外部 5V 供电） |
| G | 舵机 GND |
| D4 (GPIO2) | 舵机 SIGNAL |

> 💡 SG90 舵机的三根线（红/棕/橙）对应 VCC/GND/SIGNAL。在 ESP8266 Huzzah 开发板上，3V3 / G / D4 三个引脚连续排列，正好对应舵机 3Pin 公头的针距，可直接将舵机线插到排针上，**无需杜邦线分线**，接线非常方便。
>
> ⚠️ 舵机启动电流较大（可达 500mA+），建议使用外部 5V 供电以避免 ESP8266 重启。

---

## 四、快速开始

### 4.1 搭建 MQTT Broker

推荐使用 [EMQX](https://emqx.com/) 或 [Mosquitto](https://mosquitto.org/)：

```bash
# Docker 快速部署 EMQX
docker run -d --name emqx \
  -p 1883:1883 \
  -p 8083:8083 \
  -p 8883:8883 \
  emqx/emqx:latest
```

### 4.2 烧录固件

```bash
cd door-clicker-firmware

# 安装 PlatformIO (首次使用)
# 参考 https://platformio.org/install

# 编译
pio run

# 烧录
pio run --target upload

# 查看串口日志
pio device monitor
```

### 4.3 启动 Web 服务

```bash
cd door-clicker-web

# 创建虚拟环境（首次使用）
python3 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt

# 启动服务
python run.py
```

### 4.4 访问系统

| 页面 | URL | 说明 |
|------|-----|------|
| 开门页 | `http://<host>:5000/` | 极简开门按钮 |
| 配置页 | `http://<host>:5000/config` | MQTT 配置、舵机参数、日志 |
| 登录页 | `http://<host>:5000/login` | 管理员登录 |

### 4.5 默认账号

- 用户名：`admin`
- 密码：`admin`

> ⚠️ 首次登录后请立即修改密码

### Web 界面效果

**桌面版：**

![Web 界面效果图](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/images/web-ui-screenshot.jpg)

**手机版：**

![手机版界面](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/images/web-ui-mobile.jpg)

---

## 五、功能详解

Web 管理后台配置页总览（MQTT 连接、舵机参数、密码管理、通信日志集中管理）：

![配置页总览](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/images/web-config.jpg)

### 5.1 固件配网（ESP8266 端）

固件烧录后第一次启动，ESP8266 会进入 **AP + STA 双模式**：一边开放 WiFi 热点供你设置参数，一边尝试连接上次配置的路由器（首次为空则跳过）。

#### 步骤 1：连接设备热点

| 参数 | 值 |
|------|-----|
| WiFi 名称（SSID） | `DoorClicker` |
| WiFi 密码 | `door1234` |

> 💡 SSID 与密码在 `door-clicker-firmware/src/main.cpp` 中定义，可自行修改。

#### 步骤 2：访问 ESP8266 配置页

手机/电脑连上 `DoorClicker` 热点后，浏览器访问：

```
http://192.168.4.1/
```

自动重定向到 `http://192.168.4.1/config` 配置页。页面包含 WiFi 配置、MQTT 配置、舵机参数和舵机测试四大分区：

![ESP8266 配置页](https://cdn.jsdelivr.net/gh/DoLovya/blog@main/door-clicker/blog/images/firmware-config.jpg)

| 分区 | 字段 | 说明 |
|------|------|------|
| **WiFi 配置** | SSID / 密码 | 家中路由器的 WiFi 凭据 |
| **MQTT 配置** | Server / Port / 用户 / 密码 / Topic | MQTT Broker 地址、端口和开门主题（默认：`door/<芯片ID_HEX8>`） |
| **舵机参数** | GPIO 引脚 | 默认 `2`（即 D4） |
| **舵机测试** | GPIO 引脚 / 测试按钮 | 填入引脚后点「▶ 测试舵机」验证 0°→90°→0° 旋转是否正常 |

> ✅ 填写后点「保存配置」，ESP8266 会重启并尝试连接路由器。截图中的测试按钮（▶ 测试舵机）**不会**写入配置，可放心点击验证接线。

#### 步骤 3：恢复正常联网

保存配置并重启后，ESP8266 会连接到你设置的路由器 WiFi，此时：
- `DoorClicker` 热点仍保留（AP 模式常驻），但 STA 模式已连接公网
- 你可以把手机/电脑重新连回家用路由器，即可正常上网
- ESP8266 通过 MQTT 定期上报心跳到 Broker，Web 服务端通过心跳判定设备在线/离线

#### 步骤 4：部署 Web 服务并测试开门

```bash
# 1. 部署 door-clicker-web（参考 4.3 节与第八章）
cd door-clicker-web
pip install -r requirements.txt
python run.py

# 2. 在 Web 配置页（/config）填入相同的 MQTT Broker 与开门 Topic
#    若连接成功，页面会显示「MQTT 已连接 · 设备在线」

# 3. 回到开门页（/），点击中间的「开门」按钮
#    舵机应按配置的角度执行开门 → 延时 → 关门动作
```

至此完成「烧录 → 配网 → 联网 → 开门」全链路，后续管理全部通过 Web 端进行。

### 5.2 开门协议

Web 端通过 MQTT 发送 JSON 指令到 `door/{chip_id}`：

```json
[
  {"angle": 90, "duration": 200},
  {"angle": 0, "duration": 200}
]
```

| 字段 | 类型 | 说明 |
|------|------|------|
| angle | int | 目标角度（0-180） |
| duration | int | 动作持续时间（毫秒） |

舵机控制器支持多步命令队列，开门→关门依次执行。

### 5.3 心跳机制

设备每 30 秒发布心跳消息到 `door/{chip_id}/status`：

```json
{
  "event": "heartbeat",
  "clientId": "door_3FF12345",
  "uptime": 3600
}
```

**在线判定规则**：
- Web 端 90 秒内收到心跳 → 设备在线
- Web 端超过 90 秒未收到 → 设备离线

### 5.4 舵机配置

在配置页可以调整舵机参数：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| 开门角度 | 开门时舵机转到的角度 | 90° |
| 开门延时 | 开门动作持续时间 | 200ms |
| 关门角度 | 关门时舵机转到的角度 | 0° |
| 关门延时 | 关门动作持续时间 | 200ms |

### 5.5 MQTT 连接管理

系统提供完整的 MQTT 连接管理能力：

- **自动重连**：MQTT 断线后自动恢复连接
- **开门重试**：发送开门指令时若检测到未连接，自动重连后重试
- **连接测试**：配置页一键测试 MQTT Broker 连通性（5 秒超时）
- **手动重连**：配置页可手动触发 MQTT 重连

### 5.6 主题订阅管理

除了默认的设备状态主题，还支持手动订阅/取消订阅额外主题：

```
# 默认订阅
door/{chip_id}/status    # 设备心跳状态

# 可手动添加
door/{chip_id}/custom    # 自定义主题
```

### 5.7 实时日志系统

系统内置日志管理，支持分类查看：

| 日志类型 | 说明 |
|----------|------|
| info | 操作记录（开门、配置变更等） |
| error | 错误日志（MQTT 连接失败等） |
| send | MQTT 发送记录 |
| receive | MQTT 接收记录 |

日志存储在 `data/logs/` 目录，可通过配置页查看和清空。

### 5.8 配置页认证

配置页面需登录后才能访问，密码使用 SHA-256 加密存储：

```python
import hashlib
password_hash = hashlib.sha256(password.encode("utf-8")).hexdigest()
```

配置文件中只存储 hash 值，即使泄露也无法还原明文密码。

---

## 六、API 接口

| 方法 | 路径 | 认证 | 说明 |
|------|------|------|------|
| GET | `/` | - | 开门页面 |
| POST | `/api/open/door` | - | 发送开门指令 |
| GET | `/api/mqtt/status` | - | 获取 MQTT 连接和设备状态 |
| GET | `/api/device/status` | - | 获取设备在线状态 |
| GET | `/config` | 登录 | 配置页面 |
| GET | `/api/config` | 登录 | 获取当前配置 |
| PUT | `/api/config` | 登录 | 更新配置（支持密码修改） |
| POST | `/api/mqtt/test` | 登录 | 测试 MQTT 连接 |
| POST | `/api/mqtt/reconnect` | 登录 | 手动重连 MQTT |
| GET | `/api/topics` | 登录 | 获取已订阅主题列表 |
| POST | `/api/topics` | 登录 | 订阅新主题 |
| DELETE | `/api/topics/<topic>` | 登录 | 取消订阅 |
| GET | `/api/logs` | 登录 | 获取日志 |
| DELETE | `/api/logs` | 登录 | 清空日志 |
| GET | `/api/health` | - | 健康检查 |

---

## 七、项目结构

```
door-clicker/
├── door-clicker-firmware/       # ESP8266 固件
│   ├── include/                 # 头文件
│   │   ├── config_store.h       # 配置存储（WiFi/MQTT）
│   │   ├── door_clicker_app.h   # 主应用类
│   │   ├── door_command.h       # 舵机命令结构
│   │   ├── servo_controller.h   # 舵机控制
│   │   ├── http_config_service.h # HTTP 配网服务
│   │   └── logger.h             # 串口日志
│   ├── src/                     # 源文件
│   ├── docs/                    # 文档
│   │   └── mqtt-protocol.md     # MQTT 协议说明
│   ├── data/                    # 默认配置
│   ├── test/                    # 测试
│   └── platformio.ini           # PlatformIO 配置
│
└── door-clicker-web/            # Web 管理服务
    ├── src/                     # 源码目录
    │   ├── templates/           # HTML 模板
    │   │   ├── door.html        # 开门页
    │   │   ├── index.html       # 配置页
    │   │   └── login.html       # 登录页
    │   ├── static/              # 静态资源
    │   │   ├── css/             # 样式表
    │   │   └── favicon.svg      # 图标
    │   ├── app.py               # Flask 主应用 + 路由
    │   ├── auth.py              # 认证（登录/登出/会话）
    │   ├── config_manager.py    # 配置管理（读写/校验）
    │   ├── mqtt_client_manager.py # MQTT 客户端（单例）
    │   ├── mqtt_command_publisher.py # 命令发布
    │   ├── mqtt_command_subscriber.py # 命令订阅
    │   └── log_manager.py       # 日志管理
    ├── tests/                   # 单元测试
    ├── data/                    # 运行时数据（配置/日志）
    ├── run.py                   # 启动入口
    ├── config.json              # 运行时配置
    └── requirements.txt         # Python 依赖
```

---

## 八、部署上线

### 8.1 服务器要求

- Linux 服务器（推荐 CentOS/Ubuntu）
- Python 3.9+
- Nginx
- MQTT Broker（EMQX 推荐）

### 8.2 自动化部署

项目使用 GitHub Actions 实现 CI/CD，包含三个工作流：

| 工作流 | 触发条件 | 说明 |
|--------|----------|------|
| `web-ci.yml` | push to main | Web 服务单元测试 |
| `firmware-ci.yml` | push to main | 固件编译检查 |
| `deploy.yaml` | push to main | 自动部署到服务器 |

**部署流程**：
1. 代码推送到 main 分支
2. GitHub Actions 运行测试
3. 通过 SSH 连接服务器
4. 拉取最新代码并安装依赖
5. 重启 systemd 服务

### 8.3 Nginx 配置

```nginx
server {
    listen 80;
    server_name <your-domain>;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 8.4 systemd 服务

```ini
[Unit]
Description=Door Clicker Web Service
After=network.target

[Service]
Type=simple
User=door-clicker
WorkingDirectory=/opt/door-clicker/door-clicker-web
ExecStart=/opt/door-clicker/door-clicker-web/venv/bin/python run.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

## 九、技术栈

### 固件

| 技术 | 版本 | 用途 |
|------|------|------|
| PlatformIO | 6.0+ | 开发环境 |
| Arduino Framework | - | ESP8266 SDK |
| PubSubClient | 2.8+ | MQTT 客户端 |
| ArduinoJson | 7.3+ | JSON 解析 |

### Web 服务

| 技术 | 版本 | 用途 |
|------|------|------|
| Python | 3.9+ | 编程语言 |
| Flask | 3.0+ | Web 框架 |
| Flask-SocketIO | 5.3+ | WebSocket 支持 |
| Paho-MQTT | 2.1+ | MQTT 客户端 |
| websockets | 13.0+ | WebSocket 库 |
| SHA-256 | - | 密码加密 |

---

## 十、常见问题

### Q1: 舵机不动作？
- 确认 GPIO2 (D4) 接线正确
- 检查舵机是否已初始化
- 使用配置页面的测试功能验证

### Q2: MQTT 连接失败？
- 确认 MQTT Broker 地址和端口
- 检查 WiFi 连接是否正常
- 查看配置页的连接测试功能
- 查看日志中的错误信息

### Q3: ESP8266 不断重启？
- 检查供电是否充足（舵机启动电流可达 500mA+）
- 舵机建议使用外部 5V 供电

### Q4: 设备显示离线？
- 检查 MQTT Broker 是否正常运行
- 确认设备 WiFi 连接状态
- 查看日志中的心跳发送情况
- 设备状态以 90 秒超时判定

### Q5: 如何修改密码？
- 登录后进入配置页
- 在密码区域输入新密码保存
- 密码使用 SHA-256 加密存储

---

## 十一、总结

Door Clicker 项目展示了如何使用 ESP8266 + MQTT 构建一个完整的物联网门禁系统。通过 MQTT 协议实现了"内网穿透"，使得公网可以控制内网设备。项目包含了固件开发、Web 服务、自动化部署等完整的技术栈，适合作为物联网入门学习项目。

### 项目链接

- GitHub: https://github.com/DoLovya/door-clicker
