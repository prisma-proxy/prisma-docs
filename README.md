# Prisma Docs

Documentation site for the [Prisma](https://github.com/prisma-proxy/prisma) encrypted proxy system. Built with [Docusaurus 3](https://docusaurus.io/) with English and Chinese (Simplified) locales.

**Live site**: [yamimega.github.io/prisma](https://yamimega.github.io/prisma/)

## Content

- **Getting Started** — first proxy session walkthrough, installation guide
- **Configuration** — server and client TOML reference, environment variables
- **Features** — routing rules, PrismaTLS, TUN mode, traffic shaping, management API, console, GUI clients
- **Deployment** — Docker, systemd, Cloudflare CDN, config examples
- **Security** — PrismaVeil v5 protocol spec, cryptography, anti-replay
- **Developer Guide** — contributing, benchmarking, crate architecture

## Local Development

```bash
npm install
npm start
# → http://localhost:3000
```

Most changes are reflected live without restarting the server.

## Build

```bash
npm run build
```

Static output goes to `build/`. Serve with any static hosting service.

## Deployment

GitHub Pages deployment is automated via `.github/workflows/deploy.yml` on push to `main`.

## Related Repositories

| Repository | Description |
|------------|-------------|
| [prisma](https://github.com/prisma-proxy/prisma) | Core Rust workspace (server, client, CLI, FFI, management API) |
| [prisma-gui](https://github.com/prisma-proxy/prisma-gui) | Desktop + mobile client (Tauri 2 + React) |
| [prisma-console](https://github.com/prisma-proxy/prisma-console) | Web management dashboard (Next.js) |

## License

MIT
