# Zabbix 6.0 LTS Monitoring Stack (Perfect Docker Deploy)

基于 Docker Compose 的企业级监控系统一键部署方案。
采用 **Zabbix 6.0 LTS + MySQL 5.7** 黄金组合，已针对生产环境进行参数调优。

## 🛠️ 架构说明
- **Database**: MySQL 5.7 (配置 max_allowed_packet=512M 解决导入崩溃问题)
- **Server**: Zabbix 6.0 LTS (Ubuntu base)
- **Web**: Nginx + PHP
- **Addons**: Java Gateway + Grafana

## 🚀 部署步骤

### 1. 更换系统源为阿里云
```bash
# 备份旧源
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 一键替换为阿里云源 (针对 Ubuntu 22.04)
sudo sed -i 's/http:\/\/.*archive.ubuntu.com/http:\/\/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i 's/http:\/\/.*security.ubuntu.com/http:\/\/mirrors.aliyun.com/g' /etc/apt/sources.list

# 更新缓存并安装必备工具
sudo apt-get update
sudo apt-get install -y git vim curl net-tools ca-certificates gnupg lsb-release tree
```

### 2. 安装 Docker (使用阿里云 APT 源，避开官网)

```bash
# 1. 创建密钥目录
sudo mkdir -p /etc/apt/keyrings

# 2. 下载并添加阿里云 Docker GPG 密钥
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 3. 写入阿里云 Docker 仓库地址 (注意：这里已去掉可能导致报错的反斜杠)
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. 更新源并安装 Docker 全家桶
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 5. 启动 Docker
sudo systemctl start docker
sudo systemctl enable docker
```
### 3. 配置 Docker 镜像加速 (防止拉取镜像卡死)
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ]
}
EOF

# 重载配置并重启服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### 4. 创建目录与修复权限
```bash
# 赋予 Grafana 容器写权限
mkdir -p mysql_data grafana_data
sudo chown -R 472:472 grafana_data

# 创建 gitignore
cat > .gitignore <<EOF
mysql_data/
grafana_data/
*.log
.DS_Store
EOF
```

### 5.配置核心docker-compose.yml文件（详情见目录列表）

### 6. 启动与验证
```bash
# 1. 启动 (第一次拉取镜像稍微等一下)
sudo docker compose up -d

# 2. 实时查看日志 (此时数据库正在导入数据，不要慌，让它跑一会儿)
sudo docker logs -f zabbix-server
```
✅ 成功标志：

日志不会报错 Packet bigger than...

日志不会报错 Access denied

你会看到类似 [45%] ... [100%] database schema initialized 的进度

最后定格在 server #0 started [main process]

### 7. 访问系统
```bash
Zabbix: http://<IP>:80 (User: Admin / Pass: zabbix)
Grafana: http://<IP>:3000 (User: admin / Pass: admin)
```

