# 部署与初始化

[返回首页](../README.md)

## 1. 准备目录与镜像

下面以安装了 Docker Compose 的 Linux / NAS 为例。准备：

- 应用配置目录：长期保存配置、数据库和运行数据。
- 媒体目录：保存本地媒体、下载文件或生成的 STRM。
- 可拉取的 CineCircuit 镜像，项目使用的镜像名为 `misthanl/cinecircuit:latest`。

> 本文未验证该镜像对匿名用户的拉取权限。若镜像尚未公开，应先由维护者提供可访问的镜像标签或授权。创建公开文档仓库不会自动公开源码或镜像。

无需为了阅读或使用此文档去克隆私有源码仓库。在部署目录中保存下方 Compose 文件：

```bash
mkdir -p cinecircuit/config cinecircuit/media
cd cinecircuit
```

## 2. 保存 docker-compose.yml

下面对应项目当前提供的 Linux / NAS 配置：

```yaml
services:
  cinecircuit:
    container_name: cinecircuit
    hostname: cinecircuit
    image: misthanl/cinecircuit:latest
    stdin_open: true
    tty: true
    network_mode: host
    ports:
      - "8000:8000"
    volumes:
      - ./config:/config:rw
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./media:/media:rw
    environment:
      PORT: "8000"
      PUID: "0"
      PGID: "0"
      UMASK: "000"
      TZ: Asia/Shanghai
    privileged: true
    restart: always
```

这是项目现有配置，使用 host 网络、root 身份、privileged 模式并挂载 Docker socket；适合由管理员管理的部署环境。host 网络下，`ports` 不承担端口映射作用，应用直接使用宿主机端口。

| 配置 | 含义 | 首次部署要检查什么 |
| --- | --- | --- |
| `image` | 要运行的镜像 | 标签可访问，架构匹配设备 |
| `./config:/config` | 持久化配置与数据库 | 目录长期保留，容器能写入 |
| `./media:/media` | 让容器访问媒体文件 | 换成真实 NAS 媒体目录时，保持应用内路径使用 `/media` |
| `PORT` | 应用监听端口 | 默认 8000，不与现有服务冲突 |
| `TZ` | 容器时区 | 默认 `Asia/Shanghai` |

若采用 bridge 网络或其他系统，请按实际部署方式调整端口和路径，尤其是后续媒体服务器反代端口。

## 3. 启动

```bash
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100 cinecircuit
```

浏览器访问 `http://服务器地址:8000`。首次启动时，在容器日志中查找一次性初始化令牌。

## 4. 创建管理员

![首次初始化界面](../assets/01-setup.png)

1. 将启动日志中的一次性令牌填入“**一次性令牌**”。
2. 输入管理员用户名、密码，并再次输入密码。
3. 点击“**创建管理员账号**”。
4. 初始化完成后，用该账号进入系统。令牌在管理员创建成功后失效。

初始化令牌不是日常登录密码，也不是系统 API 令牌。不要把含令牌的启动日志原样贴进公开反馈。

## 5. 确认部署成功

- 可以完成登录并打开“设置”。
- 容器没有反复重启。
- `config` 目录开始产生运行数据。
- 准备配置的本地目录在容器内实际存在，并有需要的读写权限。

### 环境变量补充

`CONFIG_DIR` 默认使用 `/config`；`INITIAL_ADMIN_TOKEN` 可在新安装时显式指定，也可以留空让应用生成。若指定，需使用私有的 16–256 字符令牌。

仅在主机创建 `.env` 不会让其中所有变量自动进入容器。新增变量应通过 Compose 的 `environment` 或 `env_file` 显式传入。

**下一步：[基础配置](03-configuration.md)。**
