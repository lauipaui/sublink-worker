# Sublink Worker

**中文** | [English](README.en.md)

轻量代理订阅转换与管理服务，Fork 自 [7Sageer/sublink-worker](https://github.com/7Sageer/sublink-worker)。支持 Cloudflare Workers、Vercel、Node.js 和 Docker。本 Fork 的文档只描述当前快照；外部官网可能对应较新版本。[英文版](README.en.md) 完整保留上游英文说明、作者链接、部署按钮和 Star History。

- [上游中文文档](https://sublink.works/)
- [上游英文文档](https://sublink.works/en/)
- [快速开始](https://sublink.works/guide/quick-start/) / [API 参考](https://sublink.works/api/) / [FAQ](https://sublink.works/guide/faq/)
- [上游演示](https://app.sublink.works)：第三方服务，**不要输入私有订阅进行验收**。

## 功能

- 协议：Shadowsocks、VMess、VLESS、Hysteria2、Trojan、TUIC。
- 输出客户端：Sing-Box、Clash、Xray/V2Ray、Surge；具体字段兼容性以目标客户端和代码为准。
- 输入：Base64 订阅、HTTP/HTTPS 订阅、Sing-Box JSON、Clash YAML、Surge INI。
- 合并多来源、KV 短链接、Web UI、规则集与分组设置、浅色/深色主题，以及中文/英文/波斯文/俄文界面。

## 从本 Fork 本地启动

需要仍受支持的 Node.js LTS 与 npm。`package.json` 标注 Node >=18，`.nvmrc` 为 22.22.3；开发/测试工具可能有比打包后运行时更高的版本要求。

```sh
git clone https://github.com/lauipaui/sublink-worker.git
cd sublink-worker
npm ci
npm run build:node
node dist/node-server.cjs
```

默认端口 `8787`，可用 `PORT` 覆盖。Node 监听不是仅回环，私有部署请用防火墙和 HTTPS 反向代理限制访问。无持久存储配置时使用内存 KV，重启后数据丢失。

## 存储配置（Node）

| 环境变量 | 用途 |
| --- | --- |
| `REDIS_URL`，或 `REDIS_HOST` + `REDIS_PORT` | 优先选择 Redis |
| `REDIS_USERNAME` / `REDIS_PASSWORD` / `REDIS_TLS=true` | Redis 认证与 TLS |
| `REDIS_KEY_PREFIX` | 可选键前缀 |
| `KV_REST_API_URL` + `KV_REST_API_TOKEN` | 未选择 Redis 时使用 Upstash REST |
| `DISABLE_MEMORY_KV=true` | 禁止最后的内存 KV 回退 |
| `CONFIG_TTL_SECONDS` / `SHORT_LINK_TTL_SECONDS` | 配置与短链接的保存期限 |
| `STATIC_DIR` | 静态资源目录，默认 `public` |

凭据只放本地或平台 Secret，不写进 README 或 Git。

## Docker 与 Compose

```sh
# 构建此 Fork；与上游发布镜像不是同一来源
docker build -t sublink-worker-local .
# 仅本机访问的示例；未配置持久存储时仍是内存 KV
docker run -d --name sublink-worker \
  -p 127.0.0.1:8787:8787 sublink-worker-local
```

[`docker-compose.yml`](docker-compose.yml) 默认使用 `ghcr.io/7sageer/sublink-worker:latest`（**上游镜像**），配套 Redis，并将 `8787:8787` 发布到宿主机接口。Redis 数据位于 `redis-data` 卷，示例配置 TTL 为 2592000 秒（30 天）。

使用此 Fork 的本地镜像时，要明确选择镜像并审查/移除 `pull_policy: always`，否则可能继续拉取上游或对本地镜像拉取失败。升级前备份 Redis；`docker compose down -v` 会删除命名卷，不能当作无损回滚。

## Cloudflare Workers / Vercel

- Cloudflare：先登录 Wrangler，在自己的账号创建 KV 并绑定为 `SUBLINK_KV`；不要直接沿用 `wrangler.toml` 中未经核实归属的 namespace ID。阅读 [`scripts/setup-kv.cjs`](scripts/setup-kv.cjs) 后再部署，`npm run deploy` 会先运行 KV setup，再执行 `wrangler deploy`，可能改动远端资源。
- Vercel：明确导入 `lauipaui/sublink-worker`，配置 `KV_REST_API_URL`、`KV_REST_API_TOKEN`，使用 [`api/index.js`](api/index.js)、[`vercel.json`](vercel.json) 与构建脚本，实际验证持久化。
- 上游一键按钮默认导入 7Sageer 的仓库，不自动部署本 Fork。
- `npm run dev` 是 Wrangler 开发入口；`npm test` 运行 Vitest。先检查测试、工具要求，不把命令存在当作已通过验收。

## 目录

| 路径 | 内容 |
| --- | --- |
| [`src/`](src/) | 应用、平台运行时、协议解析和配置构建 |
| [`public/`](public/) | Web 界面及静态资源 |
| [`scripts/`](scripts/) | KV、Vercel 构建与发布工具 |
| [`test/`](test/) | 回归测试 |
| [`Dockerfile`](Dockerfile)、[`docker-compose.yml`](docker-compose.yml) | 容器运行入口 |
| [`wrangler.toml`](wrangler.toml) | Workers 与 KV 配置 |

## 隐私、排查与回滚

订阅、转换结果和短链接可能包含 UUID、密码、服务器地址与提供商 Token。短链接不是加密；优先使用自己的可信实例，按敏感数据保护持久存储。私有服务需审查应用访问控制并限制来源，Redis 不对公网开放。测试使用虚拟节点，不向演示站上传真实订阅。

转换失败先检查输入格式与目标客户端版本；短链接丢失先检查是否用了内存 KV、TTL 或存储故障。固定提交/镜像并备份配置和存储后再升级；回归时恢复上一版本及兼容的受保护备份。本次只补文档，未部署、转换真实订阅或重跑全部测试。

## 贡献、许可与声明

改进欢迎提交 Issue/PR，注意区分 Fork 问题和上游问题。沿用上游 [`MIT License`](LICENSE)，保留作者和第三方署名。上游声明项目仅供学习交流，不得用于违法用途，使用后果由使用者承担。
