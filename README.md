# Jetbrains Server 懒猫应用

包名：`czyt.app.jetbrains-server`，要求 lzcos 1.5.0+。
使用 `docker.io/czyt/jetbra` 的稳定版本，默认通过 `docker.1ms.run` 镜像加速交付，目标架构为 amd64。

## 安装与登录

管理员入口为 `/login`。安装时账号默认 `admin`，密码默认随机生成 20 位；自定义密码至少 12 位。
部署参数同时提供给应用和 `simple-inject-password`，已通过懒猫认证的用户可自动填充并登录。
请只将应用访问权限授予允许管理此服务的用户；匿名访问不注入管理员凭据，继续由上游访问码/会话机制鉴权。
若在上游管理界面修改账号或密码，请同步更新部署参数，否则自动登录会使用旧凭据。

数据目录 `/data` 持久化到 `/lzcapp/var/data`。
`PORT=10768`、`JETBRA_DATA_DIR=/data` 固定；标题、管理员账号/密码、安全 Cookie 和可信代理通过部署参数配置。
懒猫 HTTPS 入口默认启用 `JETBRA_COOKIE_SECURE`；直接 HTTP 访问时关闭。
`JETBRA_TRUSTED_PROXIES` 默认留空，仅在明确代理 IP 时设置逗号分隔列表。
文件上传/下载接入懒猫文件选择器。

## 自动更新与发布

`.github/workflows/lazycat.yml` 每天 UTC 03:17（北京时间 11:17）运行，也支持手动触发。
使用 `ca-x/lazycat-github-action/.github/workflows/lazycat.yml@v1`，发现新的稳定 SemVer 镜像后校验 amd64 镜像摘要，更新包版本和镜像、提交、打标签、生成 GitHub Release，仅发布到喵喵商店。
不发布官方商店，也不允许版本回退。相同或更高商店版本跳过；缺失的商店版本通过经过 SHA256 校验的对应 Release 资产补发。

需要仓库或授权此仓库使用的组织 Secrets：

- `APPSTORE_URL`：喵喵商店地址。
- `APPSTORE_TOKEN`：喵喵商店发布凭据。

仓库同名 Secret 优先于组织 Secret。上游 Docker Hub 镜像必须可公开读取，镜像加速服务也必须提供相同的 amd64 内容；验证失败时不会发布。
可通过 GitHub Variable `LAZYCAT_DOCKER_MIRROR` 指定其他镜像加速前缀。

## 本地验证

```sh
actionlint
mkdir -p dist
lzc-cli project release -o dist/application.lpk
lzc-cli lpk info dist/application.lpk
```

LPK 输出到 `dist/`，与打包内容 `content/` 分离。
GitHub Release 资产命名为 `czyt.app.jetbrains-server-v<version>.lpk`，不再将 LPK 提交到仓库；旧 LPK 仍可从 Git 历史获取。
镜像模式可能产生官方商店镜像地址/图标大小 lint 提示；本项目只发布喵喵商店。
