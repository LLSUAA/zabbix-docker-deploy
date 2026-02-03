# Zabbix 6.0 LTS Monitoring Stack (Perfect Docker Deploy)

基于 Docker Compose 的企业级监控系统一键部署方案。
采用 **Zabbix 6.0 LTS + MySQL 5.7** 黄金组合，已针对生产环境进行参数调优。

## 🛠️ 架构说明
- **Database**: MySQL 5.7 (配置 max_allowed_packet=512M 解决导入崩溃问题)
- **Server**: Zabbix 6.0 LTS (Ubuntu base)
- **Web**: Nginx + PHP
- **Addons**: Java Gateway + Grafana

## 🚀 部署步骤

### 1. 目录准备
```bash
# 赋予 Grafana 容器写权限
mkdir -p mysql_data grafana_data
sudo chown -R 472:472 grafana_data
2. 启动服务
Bash
sudo docker compose up -d
3. 访问系统
Zabbix: http://<IP>:80 (User: Admin / Pass: zabbix)

Grafana: http://<IP>:3000 (User: admin / Pass: admin) 
