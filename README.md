# 漫读（WhaleBag）

自建的稍后阅读服务：保存网页 → 抓取正文 → 离线存档 → 阅读 / 标签 / 全文搜索 / 导出。
配套一个 Safari 扩展，点一下就把当前页面收进自己的库里。

本仓库**只放构建产物**，源码在私有仓库，不对外发布。所有数据都留在你自己的机器上，不经过任何第三方服务器。

---

## 下载什么

到 [Releases](../../releases) 页面，按你的用法选一个：

| 文件 | 用途 |
| --- | --- |
| `whalebag-docker-<版本>.tar.gz` | 服务端镜像（Linux / NAS / Docker），最常用 |
| `WhaleBag-macos-<版本>.zip` | macOS 宿主 App，内含 Safari 扩展 |
| `whalebag-safari-extension-<版本>.zip` | 单独的扩展目录，手动加载用 |

只想在电脑上用、不想装 Docker 的话，装 macOS App + 用 Docker 起服务端是最省事的组合（扩展要连服务端）。

---

## 安装服务端（Docker）

```bash
docker load < whalebag-docker-1.1.0.tar.gz
mkdir -p ~/whalebag/data
docker run -d --name whalebag --restart unless-stopped \
  -p 8787:8787 \
  -v ~/whalebag/data:/data \
  whalebag:1.1.0
```

首次启动会自动生成访问 token，取出来：

```bash
docker exec whalebag cat /data/config.json
```

想固定 token 就在 `docker run` 时加 `-e WHALEBAG_TOKEN=你的token`。

浏览器打开 `http://127.0.0.1:8787/web/?token=<token>` 即可进入阅读界面，之后基本不用再手动输。

换成自己的数据目录、端口都可以，容器内固定监听 `8787`，数据全部落在 `/data`
（数据库、正文图片、`config.json`）。

> 容器里 `/media` 图片路径不带鉴权。放到公网请在前面加反向代理并做访问控制，或只在内网 / Tailscale 里用。

## 安装 Safari 扩展（macOS）

1. 解压 `WhaleBag-macos-<版本>.zip`，双击 `WhaleBag.app` 打开一次，让系统注册扩展。
   窗口本身是一个中文设置面板：能看到扩展是否启用，填「服务地址 + token」后可直接打开阅读库。
2. Safari → 设置 → 扩展 → 勾选 **漫读**。
3. Safari → 视图 → 自定义工具栏，把漫读图标拖到工具栏。
4. 点扩展图标，首次填「服务地址 + token」：一般是 `http://127.0.0.1:8787` 加上面取到的 token
   （App 和扩展的配置各自保存一次）。

在网页上右键 → **保存到漫读** 也能用。

> 产物是 ad-hoc 签名的。首次打开若被 Gatekeeper 拦下，到「系统设置 → 隐私与安全性」点「仍要打开」；
> 若 Safari 提示扩展未受信任，可在 Safari 的「开发」菜单里勾选「允许未签名的扩展」。

## 怎么用

- **保存**：扩展点一下，或在阅读界面顶部粘贴链接回车。抓取在后台进行，列表里显示「抓取中…」。
- **浏览**：左侧「未读 / 星标 / 已读」（已读即归档，仍留在「已读」里）；下面是标签筛选。
- **搜索**：顶部搜索框覆盖标题和正文，中文 ≥3 字走全文索引，命中词会高亮。
- **阅读**：点行进入阅读视图，`A−`/`A+` 调字号、可切深浅色，阅读位置自动记住。
  快捷键：列表 `j`/`k` 移动、`Enter` 打开、`/` 搜索；阅读里 `s` 星标、`a` 归档、`e` 导出、`Esc` 返回。
- **导出 / 备份**：阅读页可导出 Markdown / 单文件 HTML / 纯文本；左侧「备份」可打包全部文章与图片，
  「恢复」上传即可导回。
- **导入 wallabag**：侧栏「导入 wallabag」选它的导出 JSON 即可迁移。

## 更新

换新版本：`docker rm -f whalebag` → `docker load` 新镜像 → 用原来的数据目录重新 `docker run`。
数据在 `/data`，不会因为换镜像丢失。macOS App 直接解压覆盖旧版即可。

## 许可

MIT，详见 [LICENSE](LICENSE)。

抓取或导入的文章正文、图片版权归原作者，仅应存放在你自己的实例里。

与 wallabag、Pocket 等项目官方无关，只是借鉴了它们的交互习惯，并兼容 wallabag 的导出格式。
