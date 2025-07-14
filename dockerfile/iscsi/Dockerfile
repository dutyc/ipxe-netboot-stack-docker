FROM ubuntu:24.04

# 设置环境变量避免交互式安装
ENV DEBIAN_FRONTEND=noninteractive

# 安装必要的软件包
RUN apt-get update && \
    apt-get install -y \
        targetcli-fb \
        iproute2 \
        lvm2 \
        kmod  && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    # 启用需要的内核模块
    modprobe iscsi_target_mod || true

# 创建持久化配置目录
RUN mkdir -p /etc/target

# 暴露 iSCSI 默认端口
EXPOSE 3260

# 设置启动命令（使用完整路径）
CMD ["/usr/bin/targetcli", "shell", "in", "/etc/target"]