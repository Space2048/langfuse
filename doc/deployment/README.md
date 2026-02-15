# 部署指南

本文档提供了 Langfuse 项目的完整部署指南，涵盖从开发环境到生产环境的各个部署选项。

## 📋 部署选项概览

### 部署方式对比
| 部署方式 | 适用场景 | 复杂度 | 维护成本 | 推荐度 |
|---------|---------|--------|----------|--------|
| **Docker Compose** | 单机部署、开发测试 | 低 | 低 | ⭐⭐⭐⭐⭐ |
| **Kubernetes (Helm)** | 生产环境、高可用 | 中高 | 中高 | ⭐⭐⭐⭐ |
| **云平台 (Terraform)** | 云原生部署 | 中 | 中 | ⭐⭐⭐ |
| **虚拟机部署** | 传统环境 | 中 | 高 | ⭐⭐ |
| **Langfuse Cloud** | 托管服务 | 极低 | 极低 | ⭐⭐⭐⭐⭐ |

### 环境要求
- **CPU**: 最低 2 核，推荐 4 核
- **内存**: 最低 4GB，推荐 8GB+
- **存储**: 最低 20GB，推荐 50GB+
- **网络**: 稳定的互联网连接

## 🐳 Docker Compose 部署

### 快速开始
```bash
# 1. 克隆仓库
git clone https://github.com/langfuse/langfuse.git
cd langfuse

# 2. 复制环境变量文件
cp .env.example .env

# 3. 编辑环境变量（根据需要修改）
nano .env

# 4. 启动服务
docker compose up -d

# 5. 检查服务状态
docker compose ps

# 6. 访问应用
# Web 界面: http://localhost:3000
# API 文档: http://localhost:3000/api/docs
```

### 环境变量配置
```bash
# 数据库配置
POSTGRES_DB=langfuse
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password  # 必须修改
POSTGRES_HOST=postgres
POSTGRES_PORT=5432

# ClickHouse 配置
CLICKHOUSE_HOST=clickhouse
CLICKHOUSE_PORT=9000
CLICKHOUSE_USER=default
CLICKHOUSE_PASSWORD=your_secure_password  # 必须修改

# Redis 配置
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=your_secure_password  # 必须修改

# 应用配置
NEXTAUTH_SECRET=your_nextauth_secret  # 必须修改
NEXTAUTH_URL=http://localhost:3000
ENCRYPTION_KEY=your_encryption_key  # 必须修改

# 邮件配置（可选）
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password
SMTP_FROM=noreply@langfuse.com
```

### 服务说明
```yaml
# docker-compose.yml 中的服务
services:
  postgres:     # PostgreSQL 主数据库
  clickhouse:   # ClickHouse 分析数据库
  redis:        # Redis 缓存和队列
  minio:        # MinIO 对象存储
  langfuse-web: # Next.js Web 应用
  langfuse-worker: # 后台工作进程
```

### 数据持久化
```bash
# Docker 卷位置
/var/lib/docker/volumes/langfuse_postgres_data
/var/lib/docker/volumes/langfuse_clickhouse_data
/var/lib/docker/volumes/langfuse_redis_data
/var/lib/docker/volumes/langfuse_minio_data

# 备份数据
docker compose exec postgres pg_dump -U postgres langfuse > backup.sql
docker compose cp langfuse-clickhouse-1:/var/lib/clickhouse ./clickhouse-backup
```

### 管理命令
```bash
# 启动服务
docker compose up -d

# 停止服务
docker compose down

# 重启服务
docker compose restart

# 查看日志
docker compose logs -f
docker compose logs -f langfuse-web
docker compose logs -f postgres

# 进入容器
docker compose exec postgres psql -U postgres
docker compose exec langfuse-web bash

# 更新服务
docker compose pull
docker compose up -d --force-recreate

# 清理资源
docker compose down -v  # 删除数据卷
docker system prune -a  # 清理未使用的资源
```

## ☸️ Kubernetes 部署 (Helm)

### 前提条件
```bash
# 1. 安装 kubectl
# https://kubernetes.io/docs/tasks/tools/

# 2. 安装 Helm
# https://helm.sh/docs/intro/install/

# 3. 配置 Kubernetes 集群
# 本地: minikube, kind, k3s
# 云平台: EKS, AKS, GKE
```

### Helm Chart 部署
```bash
# 1. 添加 Helm 仓库
helm repo add langfuse https://charts.langfuse.com
helm repo update

# 2. 创建命名空间
kubectl create namespace langfuse

# 3. 创建 values.yaml 配置文件
cat > values.yaml << EOF
global:
  postgresql:
    auth:
      username: "langfuse"
      password: "your_secure_password"
      database: "langfuse"
  clickhouse:
    auth:
      username: "default"
      password: "your_secure_password"
  redis:
    auth:
      password: "your_secure_password"

web:
  replicaCount: 2
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "500m"

worker:
  replicaCount: 2
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "500m"
EOF

# 4. 安装 Langfuse
helm install langfuse langfuse/langfuse \
  --namespace langfuse \
  --values values.yaml \
  --set global.domain=langfuse.yourdomain.com

# 5. 检查部署状态
kubectl get all -n langfuse
kubectl get pods -n langfuse
kubectl get svc -n langfuse
```

### Ingress 配置
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: langfuse-ingress
  namespace: langfuse
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - langfuse.yourdomain.com
    secretName: langfuse-tls
  rules:
  - host: langfuse.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: langfuse-web
            port:
              number: 3000
```

### 持久化存储
```yaml
# storage.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: langfuse
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: standard

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: clickhouse-pvc
  namespace: langfuse
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: standard
```

### 监控和日志
```bash
# 安装 Prometheus Stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace

# 查看指标
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# 访问 http://localhost:3000
# 用户名: admin
# 密码: prom-operator

# 安装 Loki 日志
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack \
  --namespace logging \
  --create-namespace
```

## ☁️ 云平台部署

### AWS (Terraform)
```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

module "langfuse" {
  source = "github.com/langfuse/terraform-aws-langfuse"
  
  # 基础配置
  environment = "production"
  vpc_id      = "vpc-12345678"
  
  # 数据库配置
  postgres_instance_class = "db.t3.medium"
  postgres_storage_gb     = 100
  
  # 计算配置
  web_instance_type    = "t3.medium"
  worker_instance_type = "t3.medium"
  min_instances        = 2
  max_instances        = 10
  
  # 网络配置
  domain_name = "langfuse.yourdomain.com"
  ssl_certificate_arn = "arn:aws:acm:us-east-1:123456789012:certificate/abc123"
  
  # 安全配置
  allowed_cidr_blocks = ["10.0.0.0/16", "192.168.1.0/24"]
}
```

### 部署步骤
```bash
# 1. 初始化 Terraform
terraform init

# 2. 规划部署
terraform plan

# 3. 应用配置
terraform apply

# 4. 获取输出
terraform output web_url
terraform output admin_password

# 5. 销毁资源
terraform destroy
```

### 其他云平台
- **Google Cloud**: 使用 GKE 和 Cloud SQL
- **Azure**: 使用 AKS 和 Azure Database
- **DigitalOcean**: 使用 Kubernetes 和 Managed Databases
- **Vercel**: 仅部署前端，后端使用其他服务

## 🖥️ 虚拟机部署

### 系统要求
- **操作系统**: Ubuntu 22.04 LTS
- **内存**: 8GB+
- **存储**: 100GB+
- **网络**: 静态 IP 地址

### 安装步骤
```bash
# 1. 更新系统
sudo apt update && sudo apt upgrade -y

# 2. 安装 Docker
sudo apt install -y docker.io docker-compose

# 3. 安装 Node.js 24
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt install -y nodejs

# 4. 安装 pnpm
sudo npm install -g pnpm@9.5.0

# 5. 克隆项目
git clone https://github.com/langfuse/langfuse.git
cd langfuse

# 6. 配置环境变量
cp .env.example .env
nano .env  # 修改配置

# 7. 构建应用
pnpm i
pnpm run build

# 8. 启动服务
docker compose up -d

# 9. 配置反向代理（Nginx）
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/langfuse

# 10. 启用站点
sudo ln -s /etc/nginx/sites-available/langfuse /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# 11. 配置 SSL（Let's Encrypt）
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d langfuse.yourdomain.com
```

### Nginx 配置
```nginx
# /etc/nginx/sites-available/langfuse
server {
    listen 80;
    server_name langfuse.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name langfuse.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/langfuse.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/langfuse.yourdomain.com/privkey.pem;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # 代理到 Langfuse
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        proxy_pass http://localhost:3000;
    }
}
```

## 🔒 安全配置

### 基础安全
```bash
# 1. 修改默认密码
# 在 .env 文件中修改所有密码
POSTGRES_PASSWORD=strong_password_here
REDIS_PASSWORD=another_strong_password
CLICKHOUSE_PASSWORD=yet_another_strong_password
NEXTAUTH_SECRET=very_long_random_string_here

# 2. 启用防火墙
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable

# 3. 定期更新
sudo apt update && sudo apt upgrade -y
docker compose pull
docker compose up -d --force-recreate
```

### 数据库安全
```sql
-- 创建只读用户
CREATE USER readonly WITH PASSWORD 'readonly_password';
GRANT CONNECT ON DATABASE langfuse TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

-- 启用 SSL
ALTER SYSTEM SET ssl = 'on';
ALTER SYSTEM SET ssl_cert_file = '/var/lib/postgresql/server.crt';
ALTER SYSTEM SET ssl_key_file = '/var/lib/postgresql/server.key';

-- 定期备份
pg_dump -U postgres -h localhost -d langfuse | gzip > backup_$(date +%Y%m%d).sql.gz
```

### 应用安全
```typescript
// 启用 CSP 头
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-inline' 'unsafe-eval';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.langfuse.com;
    `.replace(/\s+/g, ' '),
  },
];

module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: securityHeaders,
      },
    ];
  },
};
```

## 📊 监控和告警

### 健康检查
```bash
# 应用健康检查
curl -f http://localhost:3000/api/health || exit 1

# 数据库健康检查
docker compose exec postgres pg_isready -U postgres

# Redis 健康检查
docker compose exec redis redis-cli ping

# ClickHouse 健康检查
curl -f http://localhost:8123/ping
```

### 监控指标
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'langfuse-web'
    static_configs:
      - targets: ['langfuse-web:3000']
    metrics_path: '/api/metrics'
    
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
      
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

### 告警规则
```yaml
# alert-rules.yml
groups:
  - name: langfuse
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} per second"
          
      - alert: DatabaseDown
        expr: up{job="postgres"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Database is down"
          description: "PostgreSQL database is not responding"
```

## 🔄 备份和恢复

### 备份策略
```bash
#!/bin/bash
# backup.sh

# 备份 PostgreSQL
docker compose exec postgres pg_dumpall -U postgres | gzip > /backup/postgres_$(date +%Y%m%d).sql.gz

# 备份 ClickHouse
docker compose exec clickhouse clickhouse-client --query="BACKUP DATABASE langfuse TO Disk('backup', 'clickhouse_$(date +%Y%m%d)')"

# 备份 Redis
docker compose exec redis redis-cli --rdb /data/dump.rdb
docker compose cp langfuse-redis-1:/data/dump.rdb /backup/redis_$(date +%Y%m%d).rdb

# 备份 MinIO 数据
mc mirror minio/langfuse /backup/minio_$(date +%Y%m%d)

# 清理旧备份
find /backup -type f -mtime +30 -delete
```

### 恢复数据
```bash
# 恢复 PostgreSQL
gunzip -c /backup/postgres_20240115.sql.gz | docker compose exec -T postgres psql -U postgres

# 恢复 ClickHouse
docker compose exec clickhouse clickhouse-client --query="RESTORE DATABASE langfuse FROM Disk('backup', 'clickhouse_20240115')"

# 恢复 Redis
docker compose cp /backup/redis_20240115.rdb langfuse-redis-1:/data/dump.rdb
docker compose restart redis

# 恢复 MinIO
mc mirror /backup/minio_20240115 minio/langfuse
```

## 🚀 性能优化

### 数据库优化
```sql
-- PostgreSQL 优化
CREATE INDEX idx_traces_created_at ON traces(created_at DESC);
CREATE INDEX idx_traces_project_id ON traces(project_id);
VACUUM ANALYZE;

-- ClickHouse 优化
OPTIMIZE TABLE traces FINAL;
ALTER TABLE traces MODIFY TTL created_at + INTERVAL 90 DAY;
```

### 应用优化
```typescript
// 启用缓存
// next.config.js
module.exports = {
  experimental: {
    staleTimes: {
      dynamic: 30,      // 30秒
      static: 1800,     // 30分钟
    },
  },
};

// 启用压缩
const compression = require('compression');
app.use(compression());
```

### 负载均衡
```nginx
# nginx 负载均衡配置
upstream langfuse_backend {
    least_conn;
    server 10.0.1.10:3000;
    server 10.0.1.11:3000;
    server 10.0.1.12:3000;
    keepalive 32;
}

server {
    location / {
        proxy_pass http://langfuse_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

## 📚 故障排除

### 常见问题

#### 1. 服务启动失败
```bash
# 检查日志
docker compose logs -f

# 检查端口占用
netstat -tulpn | grep :3000

# 检查资源使用
docker stats
df -h
free -h
```

#### 2. 数据库连接问题
```bash
# 测试数据库连接
docker compose exec postgres psql -U postgres -c "SELECT 1;"
docker compose exec redis redis-cli ping
curl http://localhost:8123/ping

# 检查环境变量
docker compose exec langfuse-web printenv | grep DB
```

#### 3. 性能问题
```bash
# 监控资源使用
docker stats
htop

# 分析慢查询
docker compose exec postgres psql -U postgres -c "SELECT * FROM pg_stat_activity WHERE state = 'active';"

# 检查日志
docker compose logs --tail=100 langfuse-web
```

#### 4. 内存泄漏
```bash
# 监控内存使用
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# 分析堆内存
node --inspect=0.0.0.0:9229 server.js
# 然后使用 Chrome DevTools 分析
```

### 获取帮助
- **文档**: https://langfuse.com/docs
- **社区**: https://langfuse.com/discord
- **GitHub**: https://github.com/langfuse/langfuse/issues
- **支持**: support@langfuse.com

## 🎯 部署检查清单

### 部署前
- [ ] 环境要求满足
- [ ] 域名和 SSL 证书准备
- [ ] 备份策略制定
- [ ] 监控告警配置

### 部署中
- [ ] 环境变量正确配置
- [ ] 服务成功启动
- [ ] 数据库初始化完成
- [ ] 健康检查通过

### 部署后
- [ ] 功能测试通过
- [ ] 性能测试完成
- [ ] 备份验证成功
- [ ] 文档更新完成

### 定期维护
- [ ] 安全更新应用
- [ ] 备份验证
- [ ] 性能监控
- [ ] 日志分析

---

**部署状态检查**：
```bash
# 运行部署检查脚本
curl -f http://your-domain.com/api/health || echo "健康检查失败"
docker compose ps | grep -v "Up" && echo "有服务未运行"
df -h | grep -E "(100%|9[0-9]%)" && echo "磁盘空间不足"
free -h | grep Mem | awk '{if ($3/$2 > 0.9) print "内存使用过高"}'
```

如果所有检查通过，您的 Langfuse 部署已准备就绪！🎉