# 🚀 ipxe-netboot-stack-docker  

**iPXE 网络启动 + iSCSI 无盘部署 + Docker Compose 编排**  
通过 Docker 一键部署完整的 PXE 网络引导环境，支持从 iSCSI 加载 ESXi、Linux、Windows 等操作系统。  

---

### 🧩 功能亮点  

- **一键部署**：使用 Docker Compose 编排 DHCP、TFTP、iSCSI、NGINX 服务，无需手动配置复杂依赖。  
- **iPXE 支持 HTTPS 和 COMBOOT**：预编译固件支持现代协议（HTTPS）和 Windows 安装引导。  
- **iSCSI 持久化存储**：通过容器挂载镜像文件实现无盘启动，兼容群晖、物理机、虚拟机。  
- **多系统引导菜单**：支持 ESXi、Debian、CentOS 等操作系统的网络安装。  
- **详细排错指南**：整合常见问题解决方案（如 TFTP 超时、iSCSI 连接失败等）。  

---

### 📦 组件架构  

| 组件           | 镜像                 | 作用                               |
| -------------- | -------------------- | ---------------------------------- |
| **DHCP 服务**  | `wastrachan/dhcpd`   | 分配 IP 地址并指定 TFTP 服务器     |
| **TFTP 服务**  | `手动构建`           | 提供 iPXE 固件和启动脚本           |
| **iSCSI 目标** | `mcmoe/iscsi-target` | 提供远程磁盘作为系统根文件系统     |
| **Web 服务**   | `nginx:latest`       | 提供 HTTP 引导脚本、内核镜像等资源 |

---

### 📁 目录结构  

```bash
├── dhcp
│   ├── dhcpd.conf
├── docker-compose.yml
├── dockerfile
│   ├── iscsi
│   │   └── Dockerfile
│   └── tftp
│       └── Dockerfile
├── iscsi
├── tftp
└── www
```

---

### 🔧 快速开始  

#### 1. 克隆项目  

```bash
git clone https://github.com/dutyc/ipxe-netboot-stack-docker.git
cd ipxe-netboot-stack-docker
```

#### 2. 初始化 iSCSI 镜像  

```bash
cd iscsi
dd if=/dev/zero of=storage.img bs=1G count=4
```

#### 3. 启动服务  

```bash
docker compose up -d
```

#### 5. 测试客户端启动  

- 确保客户端在同一网络，通过网卡 PXE 引导进入菜单。  
- 选择 ESXi 或 Linux 系统进行无盘安装。  

---

### ⚠️ 常见问题与解决方案  

#### 1. **TFTP 连接超时**  

- **原因**：TFTP 文件路径错误或权限不足。  

- **解决**：  

  ```bash
  chmod -R 755 tftpboot/
  sudo ufw allow 69/udp comment 'TFTP'
  ```

#### 2. **iSCSI 登录失败**  

- **原因**：LUN 编号错误或 ACL 权限未设置。  

- **解决**：  

  - 群晖 918+ 默认 LUN 编号为 `1`，脚本中需显式指定：  

    ```ipxe
    sanboot iscsi:192.168.3.2::::iqn.2025-07.com.example:storage1.2 lun=1
    ```

  - 在群晖 iSCSI 管理界面中显式允许客户端 IQN（如 `iqn.2025-07.com.example:client1`）。  

#### 3. **DHCP 服务未响应**  

- **原因**：网络模式未使用 `host` 或防火墙阻止 UDP 67/68 端口。  

- **解决**：  

  - 确保 `docker-compose.yml` 中 DHCP 服务使用 `network_mode: host`。  

  - 开放防火墙：  

    ```bash
    sudo ufw allow 67/udp comment 'DHCP'
    sudo ufw allow 68/udp comment 'DHCP'
    ```

#### 4. **iPXE 菜单不显示**  

- **原因**：`boot.ipxe` 或 `bootmenu.ipxe` 脚本路径错误。  

- **解决**：  

  - 确保 Web 服务根目录（`www/`）包含 `bootmenu.ipxe`。  

  - 修改 iPXE 脚本：  

    ```ipxe
    chain http://192.168.3.2/bootmenu.ipxe
    ```

---

### 📌 注意事项  

1. **网络模式**：  

   - 所有服务使用 `network_mode: host`，确保监听物理网络接口。  
   - 虚拟机需使用 **桥接模式**（非 NAT）。  

2. **防火墙放行**：  

   - Ubuntu 防火墙需放行以下端口：  

     ```bash
     sudo ufw allow 67/udp comment 'DHCP'
     sudo ufw allow 69/udp comment 'TFTP'
     sudo ufw allow 3260/tcp comment 'iSCSI'
     sudo ufw allow 80/tcp comment 'Web'
     ```

3. **客户端 IQN 设置**：  

   - 在 iPXE 脚本中显式指定客户端 IQN：  

     ```ipxe
     set initiator-iqn iqn.2025-07.com.example:client1
     sanboot iscsi:192.168.3.2::::iqn.2025-07.com.example:storage1.2 lun=1
     ```

---

### 📚 参考文档  

- **iPXE 官方文档**：[https://ipxe.org](https://ipxe.org)  

---

### 📬 提问与反馈  

欢迎提交 Issues 或 Pull Requests！  
如果你遇到以下问题，请提供以下信息：  

1. `docker-compose logs` 输出  
2. `tcpdump` 抓包结果（如 `sudo tcpdump -i eth0 port 69`）  
3. 客户端启动日志截图或文本  

---

### 📌 适用场景  

- **企业级无盘工作站**：通过 iSCSI 提供集中式存储。  
- **自动化部署环境**：结合 iPXE 菜单快速安装操作系统。  
- **虚拟化实验平台**：在虚拟机中测试 PXE 引导流程。  

---

### ✅ 项目贡献者  

- **dutyc**（当前维护者）  
- **VedioTalk**（原始群辉教程作者）  
- **iPXE 社区**（固件支持）  