# rclone 基础配置（HorizonTech 通用模块）

本文件整理 HorizonTech 各篇「上云」教程中复用的 rclone 步骤，例如：

- Immich 上云
- Plex 上云
- Transmission / qBittorrent 上云
- 其它需要把网盘挂载到 VPS 的场景

## 1. 安装 rclone

> 适用于大部分 Debian / Ubuntu 系统。

```bash
curl -fsSL https://rclone.org/install.sh | sudo bash
rclone version
