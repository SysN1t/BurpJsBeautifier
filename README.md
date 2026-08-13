# BurpJsBeautifier

> **Burp Suite extension for API attack-surface mapping, JS beautifying, secret detection & unauthorized-access validation.**
> **面向实战的 Burp Suite 扩展:API 攻击面测绘、JS 美化、硬编码凭证检测、未授权/越权验证。**

![Version](https://img.shields.io/badge/version-1.1.0-blue)
![License](https://img.shields.io/badge/license-Apache--2.0-green)
![JDK](https://img.shields.io/badge/JDK-17%2B-orange)
![Montoya API](https://img.shields.io/badge/Montoya%20API-2026.4-lightgrey)

输入一个 URL,即可自动抓取并美化其 JS/JSON,从压缩/混淆的代码中提取全部 API 端点,检测硬编码凭证,并内置批量验证工作台——一条流水线完成攻击面测绘与未授权/越权验证。**v1.1.0(生产版)** | Author: **SysN3t** | License: **Apache-2.0**

---

## ✨ 功能特性 / Features

### 1. JS Beautifier — JS 抓取与美化
- **主动抓取**:输入 URL/域名(每行一个),HTML 内嵌 `<script>`、外部 `<script src>`(相对/根相对/协议相对解析)同域递归抓取(深度 ≤3、单批上限 500 URL)
- **零依赖美化器**:手写 tokenizer,正确处理字符串/模板字符串(`` `${}` `` 嵌套)/正则字面量/注释,美化不破坏内容
- **Source Map 源码还原**:自动发现 `sourceMappingURL` → 下载 `.map` → 还原 `sourcesContent`
- **Webpack 模式**:自动识别 runtime,解析 chunk 映射并抓取全部分片
- **域名分组折叠树**、关键字搜索(URL/内容)、批量导出 `.js`

### 2. API Routes — API 攻击面测绘
- **深度提取**:字符串字面量、**多级常量折叠**(`BASE_URL + '/api' + version`)、**`new URL(path, base)`(URI.resolve 语义)**、fetch/axios/$.ajax 方法推断、Vue/React/Angular 路由、express/koa/fastify 后端路由、webpack chunk、动态 `import()`
- **高精度去噪**:第三方库文件识别(jstz/moment/jquery 等 30+ 库自动跳过)、IANA 时区枚举、Cookie/CSS/代码片段等噪音过滤
- **证据可溯**:每个端点带**来源 JS、行号、上下文**(前后 80 字符),右侧查看器自动滚动高亮定位
- **风险分级**:静态启发式(敏感词/方法/类型/认证参数)→ 高/中/低色块,高危置顶
- **被动监控**(默认开启):代理流量自动提取 API,白/黑名单过滤,自动拉取页面 JS(webpack 对齐),端点去重可重置
- 导出 CSV、右键复制/发送 Repeater/构造上传包/发送 Requests

### 3. Requests — 未授权 / 越权验证工作台
- 队列(上限 1000,占位参数 `{id}` 自动展开)、**暂停/继续**、危险接口绕过(默认开)
- **四种探测模式**:
  - 批量请求(匿名)—— 疑似未授权初筛
  - 对比检测(Cookie Jar)—— 匿名 vs 带 Cookie 对比
  - **JWT 批量测试** —— 粘贴泄露 JWT,验证能否绕过登录
  - **凭证批量测试** —— 粘贴任意凭证(Cookie / Authorization / X-Api-Key / Basic 等,每行一个请求头)
- 判定:🔴 疑似未授权 / 凭证有效-疑似越权 · 🟠 需认证 / 凭证失效 · 🔵 待人工确认 · 🟡 端点存在
- 原始请求/返回包查看、HTML/JSON 报告导出

### 4. 信息泄露 / Secrets — 硬编码凭证检测
- **20+ 类规则**:云厂商 AK/SK(AWS/阿里云/腾讯云)、Google/GitHub/OpenAI/Slack/Stripe/Twilio/SendGrid/npm/PyPI/Firebase、OAuth(GitHub/Google)、GitLab/HuggingFace/Shopify/Mailgun/Square/Notion/Dropbox/Facebook、私钥、数据库连接串、Basic/Bearer、**JWT(结构验证)**、**硬编码 Cookie**(`document.cookie`/`setCookie`)、API Key 头、高熵字符串(Shannon 熵)
- **三重去伪**:示例值排除(`EXAMPLE`/`your_`/`xxx`)、引号约束、JWT base64url 结构验证
- 右键:复制值 / 复制 Bearer 头 / **用此 Token 测试接口**(一键发 Repeater)

### 5. 其他 / Misc
- **笔记本**:自动持久化、时间戳、导出 txt
- **请求日志**:插件全部请求实时记录(场景/方法/URL/状态/耗时),失败红色高亮,导出 CSV
- **上传数据包构造**:multipart 自动构造(Cookie Jar/Bearer/自定义认证头)→ Repeater

## 🚀 快速开始 / Quick Start

### 加载 / Load

1. Burp → **Extensions** → **Add** → Extension type 选 **Java** → 选择 `BurpJsBeautifier.jar`
2. 加载后主界面出现 **JS Beautifier** Suite Tab(内含 6 个子页)

## 📖 使用指南 / Usage

| 目标 | 操作 |
|---|---|
| 提取 API | JS Beautifier 输入 URL → **抓取并美化** → API Routes 点 **从已抓取 JS 提取路由** |
| Webpack 站点 | 切换 **Webpack 模式** 输入入口 URL → 自动抓 chunk → 自动提取路由 |
| 实时收集 | **被动监控默认开启**——浏览器走 Burp 代理的流量自动提取(白/黑名单可配) |
| 验证未授权 | API Routes 右键接口 → **发送到 Requests(GET/POST)** → **批量请求** 或 **对比检测(Cookie Jar)** |
| 验证泄露 JWT | 信息泄露 tab 复制 JWT → Requests 粘贴 → **JWT 批量测试**(红色行 = 可绕过登录) |
| 验证任意凭证 | Requests → **凭证批量测试** → 粘贴 `Cookie: SESSION=xxx` / `X-Api-Key: xxx` 等(每行一个头) |

## 🔍 敏感数据检测规则库 / Detection Rules

| 类型 | 目标 | 类型 | 目标 |
|---|---|---|---|
| `aws_access_key` | `AKIA`/`ASIA` | `aliyun_access_key` | 阿里云 `LTAI` |
| `tencent_secret_id` | 腾讯云 `AKID` | `google_api_key` | `AIza` |
| `google_oauth_token` | `ya29.` | `github_token` / `github_oauth_token` | `ghp_`/`github_pat_` / `gho_`/`ghu_` |
| `gitlab_token` | `glpat-` | `huggingface_token` | `hf_` |
| `shopify_token` | `shpat_` | `mailgun_key` | `key-`+32hex |
| `square_token` | `sq0atp-` | `notion_token` | `secret_`+43 |
| `dropbox_token` | `sl.` | `facebook_token` | `EA...` |
| `openai_key` | `sk-` | `slack_token` | `xox*` |
| `stripe_key` | `sk_/pk_/rk_` | `twilio_key` | `SK`+32hex |
| `sendgrid_key` | `SG.` | `npm_token` / `pypi_token` | `npm_` / `pypi-` |
| `jwt_token` | JWT(结构验证) | `hardcoded_cookie` | Cookie 硬编码 |
| `private_key` | PEM 私钥 | `db_connection` | 带凭据连接串 |
| `basic_auth` / `bearer_token` | 认证头 | `header_token` | `x-api-key` 等 |
| `secret_key` | 密钥变量名赋值 | `high_entropy` | 高熵字符串(熵≥3.5) |

## 🛡️ 安全与资源保护 / Safety

- 重定向同域校验(防恶意资源 302 诱导内网请求)、抓取/拉取总量上限、>20MB 响应不载入内存
- 报文截断保护(查看器 200KB/1MB,原始报文 1MB)、队列护栏(1000 条)
- 生产版默认静默;需要全量 debug 日志加 JVM 参数 `-Dburpjsbeautifier.debug=true`(日志文件 >7 天自动清理)

## 🧪 本地测试 / Local Tests

不依赖 Burp,11 个测试套件 600+ 断言:

```powershell
javac --release 17 -encoding UTF-8 -cp montoya-api-2026.4.jar -d out-test `
  (Get-ChildItem src -Filter *.java | ForEach-Object FullName) `
  (Get-ChildItem test -Filter *.java | ForEach-Object FullName)
java -cp out-test ApiExtractorTest    # API 提取 152 项
java -cp out-test SecretScannerTest   # 密钥检测 65 项
java -cp "out-test;montoya-api-2026.4.jar" PassiveMonitorTest
```

## ⚠️ 已知限制 / Limitations

- 主动抓取为 GET,不携带会话 Cookie(仅公开可达资源);被动模式覆盖需要登录态的流量
- 前端路由拼出的 URL 是 SPA 页面路径;框架归类为启发式,可用类型筛选人工纠正
- 风险/越权判定为**静态启发式**,结果必须经 Repeater 人工复核
- 美化器为手写 tokenizer,极端嵌套边界下格式可能不完美(内容不破坏)

## ⚖️ 免责声明 / Disclaimer

本工具仅用于**已获得授权的安全测试与研究**。使用者对自身行为负责,作者不承担任何滥用后果。

*This tool is intended for authorized security testing and research only. Use responsibly.*

## 📄 License

[Apache License 2.0](LICENSE) — 明确授予**专利许可**(patent grant),允许任何人使用、修改与商用,须保留版权与许可声明。

Copyright © 2026 SysN3t
