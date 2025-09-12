# 🏢 后端部署指南

本指南将引导您完成 WHartTest 后端服务的生产环境部署。系统已改为使用API方式调用嵌入模型，无需本地下载模型文件。


## 🚀 部署方案

我们提供多种部署方案以适应不同环境的需求。

### 🐳 方案一：使用 Docker 部署 (推荐)

Docker 提供了环境一致性，是生产环境部署的首选方案。

#### 1. 构建 Docker 镜像
```bash
# 在项目根目录 (WHartTest_Django/) 下执行
docker build -t wharttest-django .
```

#### 2. 运行 Docker 容器

您可以使用 `.env` 文件来管理环境变量，这是最推荐的方式。

```bash
# 确保 .env 文件在项目根目录中
# 运行容器，并将 .env 文件传递给容器
docker run -d \
  --restart always \
  -p 8000:8000 \
  --env-file .env \
  -v ./whart_data:/app/data \
  wharttest-django
```
*   `-v ./whart_data:/app/data` 将容器内的数据目录挂载到宿主机，用于持久化存储，例如 SQLite 数据库、上传的文件等。

#### 3. 使用 Docker Compose
为了更方便地管理服务，您可以使用 `docker-compose.yml`。

```yaml
version: '3.8'
services:
  web:
    build:
      context: ./WHartTest_Django
    container_name: wharttest_backend
    restart: always
    ports:
      - "8000:8000"
    env_file:
      - .env
    volumes:
      - ./whart_data:/app/data
      - ./.cache:/app/.cache  # 挂载模型缓存目录
```
*   **注意**: 确保您的 `.env` 文件中包含了所有必要的环境变量。

### 🛠️ 方案二：手动部署 (以 Ubuntu 为例)

此方案适用于您希望对部署环境有完全控制权的场景。

#### 1. 系统准备
```bash
sudo apt update
sudo apt install python3-pip python3-venv git nginx
```

#### 2. 克隆项目
```bash
git clone <your-repo-url>
cd WHartTest_Django
```

#### 3. 创建并激活虚拟环境
```bash
python3 -m venv venv
source venv/bin/activate
```

#### 4. 安装依赖
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 5. 数据库配置
```bash
# 执行迁移
python manage.py migrate

# 创建超级管理员用户
python manage.py createsuperuser
```

#### 6. 收集静态文件
在生产环境中，静态文件（如 CSS, JavaScript, 图片）应由 Nginx 等 Web 服务器直接提供，以获得更好的性能。`collectstatic` 命令会将项目所有应用中的静态文件收集到 `STATIC_ROOT` 指定的单个目录中，以便于部署。
```bash
python manage.py collectstatic --noinput
```

#### 7. 使用 Gunicorn 启动服务
```bash
# 安装 gunicorn
pip install gunicorn

# 启动服务
gunicorn wharttest_django.wsgi:application \
  --bind 0.0.0.0:8000 \
  --workers 4 \
  --timeout 120 \
  --preload
```
*   `--preload` 会在启动时预加载模型，减少首次请求的延迟。


## 🔍 部署验证

### 1. 验证 API 连接
启动服务后，检查日志输出，确认嵌入模型 API 连接正常。
```log
🚀 正在初始化嵌入模型API...
✅ 嵌入模型API连接成功
🧪 API测试成功，服务正常
🤖 向量存储管理器初始化完成:
   ✅ 实际使用的嵌入模型: API嵌入服务
```

### 2. API 健康检查
```bash
# 检查项目 API 是否正常 (需要有效的 JWT Token)
curl -X GET http://your-domain.com/api/projects/ \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### 3. 知识库功能测试
通过 API 创建一个知识库，上传文档并进行搜索，验证整个流程是否正常。

## ✅ 生产环境检查清单

- [ ] `DEBUG` 设置为 `False`
- [ ] `SECRET_KEY` 已更换为强密钥
- [ ] 使用 `Gunicorn` 或其他 WSGI 服务器
- [ ] 配置 `Nginx` 作为反向代理
- [ ] 数据库已从 SQLite 切换到 `PostgreSQL`
- [ ] 嵌入模型 API 已配置并连接正常
- [ ] 静态文件已通过 `collectstatic` 收集并由 Nginx 服务
- [ ] `SSL/TLS` 证书已配置，强制 HTTPS
- [ ] 防火墙已启用，只开放必要端口
- [ ] 备份策略已制定（数据库和用户上传文件）
- [ ] 日志记录和监控已配置
