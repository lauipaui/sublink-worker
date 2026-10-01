[中文](README.md) | **English**

<div align="center">
  <img src="public/favicon.png" alt="Sublink Worker" width="120" height="120"/>

  <h1><b>Sublink Worker</b></h1>
  <h5><i>One Worker, All Subscriptions</i></h5>

  <p><b>A lightweight subscription converter and manager for proxy protocols, deployable on Cloudflare Workers, Vercel, Node.js, or Docker.</b></p>

  <a href="https://trendshift.io/repositories/12291" target="_blank">
    <img src="https://trendshift.io/api/badge/repositories/12291" alt="7Sageer%2Fsublink-worker | Trendshift" width="250" height="55"/>
  </a>

  <br>

<p style="display: flex; align-items: center; gap: 10px;">
  <a href="https://deploy.workers.cloudflare.com/?url=https://github.com/7Sageer/sublink-worker">
    <img src="https://deploy.workers.cloudflare.com/button" alt="Deploy to Cloudflare Workers" style="height: 32px;"/>
  </a>
  <a href="https://vercel.com/new/clone?repository-url=https://github.com/7Sageer/sublink-worker&env=KV_REST_API_URL,KV_REST_API_TOKEN&envDescription=Vercel%20KV%20credentials%20for%20data%20storage&envLink=https://vercel.com/docs/storage/vercel-kv">
    <img src="https://vercel.com/button" alt="Deploy to Vercel" style="height: 32px;"/>
  </a>
</p>

  <h3>📚 Documentation</h3>
  <p>
    <a href="https://app.sublink.works"><b>⚡ Live Demo</b></a> ·
    <a href="https://sublink.works/en/"><b>Documentation</b></a>
    <a href="https://sublink.works"><b>中文文档</b></a>·
  </p>
  <p>
    <a href="https://sublink.works/guide/quick-start/">Quick Start</a> ·
    <a href="https://sublink.works/api/">API Reference</a> ·
    <a href="https://sublink.works/guide/faq/">FAQ</a>
  </p>
</div>

## 🚀 Quick Start

### One-Click Deployment
- Choose a "deploy" button above to click
- That's it! See the [Document](https://sublink.works/guide/quick-start/) for more information.

### Alternative Runtimes
- **Node.js**: `npm run build:node && node dist/node-server.cjs`
- **Vercel**: `vercel deploy` (configure KV in project settings)
- **Docker**: `docker pull ghcr.io/7sageer/sublink-worker:latest`
- **Docker Compose**: `docker compose up -d` (includes Redis)

## ✨ Features

### Supported Protocols
ShadowSocks • VMess • VLESS • Hysteria2 • Trojan • TUIC

### Client Support
Sing-Box • Clash • Xray/V2Ray • Surge

### Input Support
- Base64 subscriptions
- HTTP/HTTPS subscriptions
- Full configs (Sing-Box JSON, Clash YAML, Surge INI)

### Core Capabilities
- Import subscriptions from multiple sources
- Generate fixed/random short links (KV-based)
- Light/Dark theme toggle
- Flexible API for script automation
- Multi-language support (Chinese, English, Persian, Russian)
- Web interface with predefined rule sets and customizable policy groups

## 🤝 Contributing

Issues and Pull Requests are welcome to improve this project.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ⚠️ Disclaimer

This project is for learning and exchange purposes only. Please do not use it for illegal purposes. All consequences resulting from the use of this project are solely the responsibility of the user and are not related to the developer.

## ⭐ Star History

Thanks to everyone who has starred this project! 🌟

<a href="https://star-history.com/#7Sageer/sublink-worker&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=7Sageer/sublink-worker&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=7Sageer/sublink-worker&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=7Sageer/sublink-worker&type=Date" />
 </picture>
</a>


## Fork-specific deployment and security notes

This checkout belongs to `lauipaui/sublink-worker`, forked from [7Sageer/sublink-worker](https://github.com/7Sageer/sublink-worker). The upstream one-click buttons, demo and published container image above are retained for attribution; they do **not** automatically deploy this fork. External documentation can describe a newer release.

### Build the fork locally

Use a supported Node.js LTS and npm. `package.json` declares Node >=18, while this snapshot's `.nvmrc` pins 22.22.3; development/test tooling may impose higher requirements than the bundled runtime.

```sh
git clone https://github.com/lauipaui/sublink-worker.git
cd sublink-worker
npm ci
npm run build:node
node dist/node-server.cjs
```

Default port is `8787`, configurable through `PORT`. The Node listener is not loopback-only; restrict access with a firewall/HTTPS reverse proxy. Memory KV is the fallback and disappears on process restart.

### Storage selection for Node

| Variable(s) | Behavior |
| --- | --- |
| `REDIS_URL` or `REDIS_HOST` + `REDIS_PORT` | Select Redis first |
| `REDIS_USERNAME`, `REDIS_PASSWORD`, `REDIS_TLS=true` | Optional Redis authentication/TLS |
| `REDIS_KEY_PREFIX` | Optional key prefix |
| `KV_REST_API_URL`, `KV_REST_API_TOKEN` | Upstash REST storage when Redis is not selected |
| `DISABLE_MEMORY_KV=true` | Disable the final in-memory fallback |
| `CONFIG_TTL_SECONDS`, `SHORT_LINK_TTL_SECONDS` | Saved-config / short-link TTL settings |
| `STATIC_DIR` | Static asset directory; default `public` |

Keep backend credentials in local/platform secrets, not committed files. The included Compose file starts Redis and a worker, persists Redis in `redis-data`, sets config TTL to 2592000 seconds (30 days), and publishes `8787:8787` on host interfaces. By default it pulls the **upstream image** on startup. To run your own build, select it explicitly and review/remove the `pull_policy: always` setting for a local-only image. Back up Redis data before upgrades; `docker compose down -v` deletes named volumes and is not a safe rollback step.

### Cloudflare / Vercel

- Cloudflare: authenticate Wrangler, create a KV namespace in **your own** account and bind it as `SUBLINK_KV`. Do not reuse the namespace ID committed in `wrangler.toml` without verifying ownership. Review `scripts/setup-kv.cjs`; `npm run deploy` runs this setup step before `wrangler deploy` and can change remote resources.
- Vercel: import this fork explicitly, configure `KV_REST_API_URL`/`KV_REST_API_TOKEN` in project secrets, and use the included `api/index.js`, `vercel.json` and build script. Validate the deployment and persistence rather than assuming the upstream button selected your fork.
- `npm run dev` uses Wrangler for local development. `npm test` runs Vitest; read tests and configure the required Node/tooling version first.

### Privacy and rollback

Subscription URLs and generated configurations may contain passwords, UUIDs, server addresses and private provider tokens. Prefer your own trusted converter to public demos; treat short links and stored configs as private access-bearing data, not encryption. Review application access controls and use network/reverse-proxy restrictions for private deployments. Do not publicly expose Redis or publish real subscriptions in tests/issues.

Pin a revision/image, back up persistent storage and private settings, then test conversion with synthetic nodes. On regression restore the previous application revision/image and compatible protected storage backup. This documentation update did not deploy an instance, convert real subscriptions or rerun the full test suite.
