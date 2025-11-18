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
```


## 2. 配置 OneDrive remote（示例）

在 VPS 上执行 rclone 向导：

```bash
rclone config
```

下面以 OneDrive 为例，按提示大致是：

n          # New remote
onedrive   # name：自定义，例如 onedrive
...        # Storage：在列表中选择 OneDrive（按实际编号）
...        # 账号类型选 personal / 家用账号
...        # client_id / client_secret 如果不清楚可以先留空
y          # Use auto config?（本地有浏览器就选 y，SSH 远程没浏览器就选 n）

完成后可以用：

```bash
rclone ls onedrive:
```

## 3.	使用 systemd 自动挂载 OneDrive（示例）

注意：请先确认路径是否需要改成你自己的：
	•	remote 名：onedrive
	•	挂载点：/mnt/onedrive
	•	配置文件：/etc/rclone/rclone.conf

## 3.1 准备挂载点并允许 allow-other

```bash
sudo mkdir -p /mnt/onedrive
sudo grep -q "user_allow_other" /etc/fuse.conf || echo "user_allow_other" | sudo tee -a /etc/fuse.conf >/dev/null
```

## 3.2 写入 systemd 模板（rclone@.service）

```bash
sudo tee /etc/systemd/system/rclone@.service >/dev/null << 'EOF'
[Unit]
Description=Rclone Mount (%i)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
# 这里可以按需修改为你的 rclone.conf 路径 & 挂载路径
Environment=RCLONE_CONFIG=/etc/rclone/rclone.conf
Environment=MNT=/mnt/onedrive
ExecStart=/usr/bin/rclone mount %i: ${MNT} \
  --allow-other \
  --dir-cache-time=12h \
  --poll-interval=5m \
  --vfs-cache-mode=full \
  --vfs-cache-max-age=12h \
  --log-file=/var/log/rclone-%i.log \
  --log-level=INFO
ExecStop=/bin/fusermount -uz ${MNT}
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## 3.3 载入并设置开机自动挂载

```bash
REMOTE=onedrive

sudo systemctl daemon-reload
sudo systemctl enable --now rclone@"${REMOTE}"
sudo systemctl status --no-pager rclone@"${REMOTE}"
```

看到 Active: active (running) 就说明挂载成功

