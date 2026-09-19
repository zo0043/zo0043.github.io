# 自定义域名绑定指引（GitHub Pages + 子域名）

本仓库已通过 Hugo + GitHub Actions 自动部署到 GitHub Pages。
下面是把站点绑定到**子域名**（如 `blog.zo0043.com`）的完整步骤。

## ⚠️ 发布前必须替换的两处占位符

当前代码里用的是占位域名 `blog.example.com`，拿到你的真实域名后：

1. **`hugo.yaml`** — 两处：
   - `baseURL: "https://blog.example.com/"`（第 4 行）
   - `copyright` 里的 `https://blog.example.com/`（第 6 行）
2. **`CNAME`** 文件 — 文件内容就是一行域名，改成你的真实子域名（如 `blog.zo0043.com`，不带 `https://`）

改完后提交推送，Actions 会自动重新构建。

## 第一步：在域名服务商处配置 DNS

去你买域名的服务商（阿里云 / 腾讯云 / Cloudflare / Namecheap 等）DNS 管理页，添加一条记录：

| 类型  | 主机记录 | 记录值                 | TTL |
|-------|----------|------------------------|-----|
| CNAME | blog     | zo0043.github.io       | 600 |

> 如果想让 `www.zo0043.com` 也能访问，再加一条 `www → zo0043.github.io` 的 CNAME。

## 第二步：在 GitHub 仓库启用自定义域名

1. 打开仓库 **Settings → Pages**
2. 在 **Custom domain** 输入框填你的子域名（如 `blog.zo0043.com`），点 **Save**
3. 勾选 **Enforce HTTPS**（等 GitHub 自动签发证书，通常几分钟，DNS 生效后才能签发）

> GitHub 会自动在仓库里创建/更新 `CNAME` 文件，内容和代码里的一致即可。

## 第三步：验证

1. 浏览器访问 `https://blog.zo0043.com`，确认能打开且地址栏显示新域名
2. 用 `dig blog.zo0043.com` 或 [dnschecker.org](https://dnschecker.org) 确认 CNAME 生效
3. 若 HTTPS 未自动生效，等 5–30 分钟刷新，或确认 DNS 已全球生效后再 Enforce HTTPS

## 常见问题

- **CNAME 记录冲突**：CNAME 不能和其他记录（如 MX、TXT 根记录）共存于同一主机记录，如有冲突先删除。
- **证书签发失败**：先确认 `dig` 能看到 CNAME，再勾选 Enforce HTTPS。
- **部署失败**：看仓库 **Actions** 标签页的运行日志，通常是 YAML 格式或 submodule 问题。
