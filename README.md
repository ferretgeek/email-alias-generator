# 邮箱标签地址生成器

中文 · [English](README_EN.md)

在浏览器里批量生成 `名字+标签@邮箱域名` 形式的地址，为不同网站分配不同标签，并导出 TXT。它只生成地址，不创建新邮箱。

适用环境：浏览器，推荐用 Python 3.10+ 启动本地服务；需邮箱支持标签收件，iCloud 与自定义域名仅供兼容性试验。

[本地使用](#三分钟开始) · [邮箱兼容性](#各平台到底支持到什么程度) · [部署说明](docs/部署说明.md)

## 界面

![工作台](docs/images/dashboard.png)

![入口与产品设计语言](docs/images/intro.png)

## 它能做什么

- **自动认平台** — 自动识别 Gmail 与 Microsoft / Exchange Online 地址，按各自规则生成。
- **也认你的自建域名** — iCloud 和任意域名有显式的"兼容性实验模式"，**不把未证实的能力包装成承诺**。
- **认得多种输入格式** — 纯邮箱、`邮箱----附加字段`，以及 Gmail 的双行格式。
- **量能撑住** — 每个邮箱可生成 1–50000 个标签地址，单次上限 200000 个，避免手滑把浏览器拖死。
- **输出可控** — 可选保留原邮箱、保留附加字段、单行或双行格式。
- **附加字段默认剥离** — 即使你选了保留，页面预览里仍然只显示遮罩。
- **四套全局主题** — 天光、青瓷、暮霞与深灰，选择留在当前浏览器。
- **没有外部依赖** — 无外部字体、无统计脚本、无接口请求、无第三方运行依赖。

## 三分钟开始

需要 Python 3.10 或更高版本，运行时只用标准库。

```bash
python scripts/serve.py
```

打开 `http://127.0.0.1:4173`。

也可以直接双击 `index.html`，但本地服务器会附带更完整的安全响应头（并设 10 秒连接 deadline、最多 64 个工作线程，请求日志规范为有界单行）。

Docker：

```bash
docker compose up -d --build
```

然后打开 `http://127.0.0.1:8080`。公网部署、反向代理和升级方式见[部署说明](docs/部署说明.md)。

## 各平台到底支持到什么程度

这一节写得比较啰嗦，因为这件事**不能含糊**——生成地址不等于对方一定收得到。

| 模式 | 工具行为 | 依据与边界 |
| --- | --- | --- |
| 自动识别 | 只处理 Gmail 与 Microsoft | 推荐默认值 |
| Gmail | 生成 `name+tag@gmail.example` | [Google 官方说明](https://support.google.com/a/users/answer/9282734)：标签地址会进入当前收件箱 |
| Microsoft | 生成 Exchange Online Plus Addressing | [Microsoft 官方文档](https://learn.microsoft.com/exchange/recipients-in-exchange-online/plus-addressing-in-exchange-online)：默认开启，但组织管理员可以关闭 |
| iCloud | 仅在你主动选择后生成候选地址 | Apple 官方介绍的是[已创建的 iCloud 别名](https://support.apple.com/guide/icloud/mm6b1a490a/icloud)和 [Hide My Email](https://support.apple.com/guide/icloud/create-and-edit-addresses-mm1a876f7aed/icloud)，**没有承诺动态 `+tag`；必须自己先实测** |
| 任意域名 | 生成候选地址 | 是否投递完全取决于你的邮件服务商 |

它只生成收件地址：**不创建邮箱账号、不登录邮箱、不读取邮件、不绕过网站规则。** 另外要知道，有些网站会直接拒绝含 `+` 的地址。

## 隐私设计

你粘进去的内容可能包含密码、应用专用密码、客户端标识或刷新令牌。所以这个项目按"**输入就是秘密**"来设计：

1. 页面设置 `connect-src 'none'`——**浏览器层面就不存在数据上传通道**，不是"我们承诺不上传"。
2. 用户内容不进 `localStorage`、不进日志、不进 URL；只有主题名称会持久化。
3. 附加字段默认不进入结果，预览里永远遮挡。
4. 下载文件由浏览器内存中的 `Blob` 创建，不经过任何服务器。
5. `.gitignore` 默认忽略 TXT、私有目录、数据目录和导出目录，降低误提交风险。
6. 仓库里的示例全部是从零编写的虚构内容，不含开发者的原始账号数据。

## 技术上值得一提的地方

**输入体积在拆行前就设限。** 粘贴内容在拆行**之前**限制为 5 MiB 字符和 100000 行，诊断信息只保留前 200 条，最终 UTF-8 输出另有 32 MiB 上限。一个纯前端工具最容易死在"用户粘了一个 80MB 的文件"，所以上限放在最前面。

**`connect-src 'none'` 是硬保证。** 这条 CSP 让页面根本无法发起网络请求——即使代码里有 bug 或依赖被投毒，也没有可用的出口。这比任何隐私声明都可靠。

**CI 里的安全门禁是真跑的。** Ruff、Bandit、pip-audit、detect-secrets、CodeQL 和预览资产校验都在 CI 里；公开发布前另对工作树和**完整 Git 历史**跑一遍 Gitleaks。

更详细的威胁模型、CSP 与发布门禁见[技术与安全](docs/技术与安全.md)。

## 验证

```bash
node --test tests/alias-core.test.js
python -m unittest discover -s tests -p "test_*.py" -v
python -m compileall -q scripts tests
```

## 项目结构

```text
assets/             浏览器核心、交互与视觉样式
deploy/             Nginx 安全配置
docs/               部署、技术、安全审计与预览图
scripts/serve.py    零依赖的本地静态服务器
tests/              Node 核心测试与 Python 服务器测试
index.html          应用入口
```

## 参与与许可

欢迎提交问题与改进，先读 [CONTRIBUTING.md](CONTRIBUTING.md)。安全问题请使用 GitHub Private Vulnerability Reporting，说明见 [SECURITY.md](SECURITY.md)。

[MIT License](LICENSE)。这是独立项目，与 Google、Microsoft、Apple 均无隶属或背书关系。
