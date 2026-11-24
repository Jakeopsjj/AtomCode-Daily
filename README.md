<div align="center">

<sub><strong>L0NE-6 · DAILY SIGNAL</strong></sub>

# 🧬 AtomCode Daily

**探测 AtomCode 当前 CodingPlan，并在没有有效套餐时按 Max → Pro → Lite 级联领取，最后回读套餐档位与有效期。**

> 当前版本：**v1.0.3**（2026-11-12） · 核心能力：套餐状态查询、免费 CodingPlan 领取、凭据续期、服务端回读

[简体中文](README.md) · [English](docs/README.en.md) · [繁體中文](docs/README.zh-Hant.md) · [日本語](docs/README.ja.md) · [한국어](docs/README.ko.md) · [Português (BR)](docs/README.pt-BR.md)

<a href="https://github.com/L0NE-6/AtomCode-Daily/releases"><img src="https://img.shields.io/github/v/release/L0NE-6/AtomCode-Daily?style=for-the-badge&label=Release&color=2ea44f&logo=github" alt="Release" /></a>
<img src="https://img.shields.io/github/stars/L0NE-6/AtomCode-Daily?style=for-the-badge&label=Stars&color=FFC75F&logo=github" alt="Stars" />
<img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python" />
<img src="https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20Qinglong-4EAA25?style=for-the-badge&logo=linux&logoColor=white" alt="Platform" />
<img src="https://img.shields.io/github/license/L0NE-6/AtomCode-Daily?style=for-the-badge&label=License&color=F472B6" alt="License" />
<a href="https://github.com/L0NE-6/AtomCode-Daily/releases"><img src="https://img.shields.io/badge/Download-Releases-2ea44f?style=for-the-badge&logo=github" alt="Download" /></a>

[🚫 使用边界](#usage-policy) · [✨ 30 秒速览](#overview) · [🧭 这是什么](#what-is-this) · [🔁 工作原理](#protocol) · [🔑 凭据](#credentials) · [🚀 本地](#local) · [🐧 青龙](#qinglong) · [☁️ Actions](#actions) · [⌨️ CLI](#cli-output) · [🧾 结果码](#result-codes) · [✅ 成功判据](#success) · [🧪 测试](#tests) · [🧱 设计原则](#design) · [🔒 隐私](#privacy) · [📁 目录](#layout) · [📦 版本](#release) · [⚠️ 免责声明](#disclaimer) · [☕ 赞赏](#donate) · [💬 反馈与 Star](#feedback)

</div>

---

<a id="usage-policy"></a>
## 🚫 使用边界与转载声明

> [!IMPORTANT]
> **官方版本永久免费。** 未经作者书面许可，禁止付费包装、搬运引流、冒充作者，或使用作者身份与二维码制造官方授权假象。

本项目仓库、脚本、部署说明和 Release 下载始终免费公开。未经作者明确书面许可，禁止将其包装为付费资源、内部版、独家版、会员权益、收费课程、付费部署或包更新服务；禁止搬运到其他账号、群组、频道、网盘、电商或付费平台进行涨粉、截流、倒卖和引流；禁止删除作者与版权信息后冒充原创。

如需商业合作、品牌授权、联合分发或使用作者名义提供付费服务，请先取得作者明确书面授权。完整规则和收款码防诈骗说明见 [NOTICE.md](NOTICE.md)。

---

<a id="overview"></a>
## ✨ 30 秒速览

| 维度 | 说明 |
| :--- | :--- |
| **做什么** | 读取 AtomCode CodingPlan 状态；没有有效套餐时依次尝试 Max、Pro、Lite。 |
| **输入** | `ATOMCODE_TOKENS` 多账号，或 `ATOMCODE_TOKEN` / `ATOMCODE_REFRESH_TOKEN` 单账号。 |
| **输出** | 套餐档位、到期时间、剩余天数、额度窗口、领取回执和最终回读结论。 |
| **部署** | Windows、Linux、青龙面板、GitHub Actions。 |
| **依赖** | Python 3 + `requests`；Actions 固定 Python 3.11。 |
| **成功判据** | `status-v2` 回读到可识别的 Max / Pro / Lite 档位和有效期，不能只凭 claim 回执。 |

---

<a id="what-is-this"></a>
## 🧭 这是什么

AtomCode 的免费权益是 **CodingPlan 套餐授权**，不是每天增加的积分。脚本先读取当前套餐；如果已有有效套餐，直接输出状态并停止。没有套餐或套餐已过期时，才按 `Max → Pro → Lite` 提交领取，命中即停。

脚本同时处理三件事：

- 解析 `expires_at`、`remaining_days` 和额度窗口，避免把过期套餐当成有效。
- 在 access token 失效时使用 refresh token 续期，并原子保存轮换后的凭据。
- 领取回执出现 `success` 或 `duplicate` 后重新读取 `status-v2`，只有档位和有效期都确认才报告成功。

---

<a id="protocol"></a>
## 🔁 协议、工作原理与回读

默认 CodingPlan API 基址为 `https://api.gitcode.com/api/v5`，认证代理基址为 `https://acs.atomgit.com`。两处均可通过环境变量覆盖。

```text
GET /coding-plan/status-v2
  ├─ 有效套餐 ──> 输出档位、到期时间、额度窗口
  └─ 无有效套餐
       ├─ --query ──> 只报告状态，不提交领取
       └─ 正常执行
            ├─ POST /coding-plan/claim-v2  {"plan_type":"Max"}
            ├─ POST /coding-plan/claim-v2  {"plan_type":"Pro"}
            └─ POST /coding-plan/claim-v2  {"plan_type":"Lite"}
                 └─ success / duplicate
                      └─ GET /coding-plan/status-v2
                           ├─ 档位 + 有效期确认 ──> 成功
                           └─ 无法确认 ──> 待确认 / 失败
```

回读优先级固定：可解析的 `expires_at` 优先；缺失或不可解析时才使用 `remaining_days`。领取回执本身只是动作证据，服务端套餐状态才是完成证据。

---

<a id="credentials"></a>
## 🔑 凭据与环境变量

### 账号与运行

| 变量 | 必填 | 说明 |
| :--- | :---: | :--- |
| `ATOMCODE_TOKENS` | 二选一 | 多账号，每行 `备注\|access\|refresh\|expires_at\|created_at\|expires_in` |
| `ATOMCODE_TOKEN` | 二选一 | 单账号 access token |
| `ATOMCODE_REFRESH_TOKEN` | 建议 | 单账号 refresh token，用于自动续期 |
| `ATOMCODE_EXPIRES_AT` | 否 | 毫秒或秒时间戳，也支持可解析日期时间 |
| `ATOMCODE_AUTH_FILE` | 否 | 官方客户端 auth 文件路径 |
| `ATOMCODE_STATE_FILE` | 否 | 续期状态文件，默认 `atomcode_state.json` |
| `ATOMCODE_PROXY` | 否 | HTTP(S) 或 SOCKS 代理 |
| `ATOMCODE_NO_NOTIFY` | 否 | `1` 表示关闭推送 |
| `ATOMCODE_BROKER_BASE` | 高级 | 覆盖认证代理基址 |
| `ATOMCODE_CODINGPLAN_API_BASE` | 高级 | 覆盖 CodingPlan API 基址 |

### 推送

| 变量 | 说明 |
| :--- | :--- |
| `PLUSPLUS_TOKEN` | PushPlus 令牌 |
| `BARK_URL` | Bark 地址 |
| `WECOM_WEBHOOK` | 企业微信机器人 webhook 或 key |
| `QYWX_TOKEN` | 企业微信 key 的兼容变量名 |
| `DINGTALK_WEBHOOK` | 钉钉机器人 webhook |

---

<a id="local"></a>
## 🚀 本地快速开始

```bash
python -m pip install -r requirements.txt
```

如果本机已有官方客户端 auth 文件，可以先转换：

```bash
python atomcode_auth.py --file path/to/auth.toml
python atomcode_auth.py --file path/to/auth.toml --write-env .env
```

单账号环境变量：

```text
ATOMCODE_TOKEN=ACCESS_TOKEN
ATOMCODE_REFRESH_TOKEN=REFRESH_TOKEN
ATOMCODE_EXPIRES_AT=1799999999000
```

多账号示例：

```text
ATOMCODE_TOKENS=
主账号|ACCESS_TOKEN|REFRESH_TOKEN|1799999999000
小号|ACCESS_TOKEN|REFRESH_TOKEN
```

先只读核对，再执行领取：

```bash
python atomcode_daily.py --query
python atomcode_daily.py
```

---

<a id="qinglong"></a>
## 🐧 青龙面板

```bash
pip3 install requests
```

在青龙环境变量中填写 `ATOMCODE_TOKENS` 或单账号变量，然后创建任务：

```text
20 8 * * * python3 atomcode_daily.py
```

续期后的 access token 和 refresh token 写入 `atomcode_state.json`。该文件已被 `.gitignore` 屏蔽；青龙应使用可写且可持久化的脚本目录，或通过 `ATOMCODE_STATE_FILE` 指定持久路径。

---

<a id="actions"></a>
## ☁️ GitHub Actions

工作流位于 [`.github/workflows/daily.yml`](.github/workflows/daily.yml)，当前只启用 `workflow_dispatch`；定时段保持注释，启用前需要把北京时间换算成 UTC。

在 `Settings → Secrets and variables → Actions` 中配置：

| Secret | 必填 | 说明 |
| :--- | :---: | :--- |
| `ATOMCODE_TOKENS` | 二选一 | 多账号令牌 |
| `ATOMCODE_TOKEN` | 二选一 | 单账号 access token |
| `ATOMCODE_REFRESH_TOKEN` | 否 | 单账号 refresh token |
| `ATOMCODE_EXPIRES_AT` | 否 | 到期时间提示 |
| `ATOMCODE_PROXY` | 否 | 代理地址 |
| `PLUSPLUS_TOKEN` / `BARK_URL` / `WECOM_WEBHOOK` / `DINGTALK_WEBHOOK` | 否 | 推送渠道 |

Actions 不保留本地状态文件。refresh token 轮换后，需要重新导出并更新 Secret；长期运行优先使用可持久化的青龙目录。

---

<a id="cli-output"></a>
## ⌨️ CLI 与输出

| 命令 | 作用 |
| :--- | :--- |
| `python atomcode_daily.py` | 查询并按需领取 |
| `python atomcode_daily.py --query` | 只查询，不提交领取请求 |
| `python atomcode_daily.py --only 1` | 只处理第一个账号 |
| `python atomcode_daily.py --no-notify` | 本轮关闭推送 |

输出按账号分块，包含套餐档位、到期时间、剩余天数、额度窗口、领取动作和 `status-v2` 回读结果。推送内容与终端摘要使用同一份结论。

---

<a id="result-codes"></a>
## 🧾 结果码

| 结果 | 含义 |
| :--- | :--- |
| 退出码 `0` | 套餐有效、已持有，或领取后回读确认成功 |
| 退出码 `1` | 未找到令牌、选中账号不存在、三档均拒绝，或回读未确认 |
| `success` | 领取接口接受动作，仍必须继续回读 |
| `duplicate: true` | 服务端表示已持有，仍必须确认当前档位和有效期 |
| HTTP `401/403` | 可能触发 refresh token 续期；续期失败则结束该账号 |
| `待确认` | claim 有回执，但 `status-v2` 回读失败或未出现有效套餐 |

---

<a id="success"></a>
## ✅ 成功判据

一次账号执行只有在以下条件同时成立时才算成功：

1. `status-v2` 返回有效套餐。
2. 套餐名或 `plan_type` 可识别为 `Max`、`Pro` 或 `Lite`。
3. `expires_at` 仍在有效期内；缺失或不可解析时才接受 `remaining_days`。
4. 若本轮发生领取，claim 回执与后续 `status-v2` 回读一致。

HTTP 200、`success: true` 或 `duplicate: true` 单独出现都不构成成功。

---

<a id="tests"></a>
## 🧪 测试

```bash
python -m unittest discover -s tests -v
python -m py_compile atomcode_daily.py atomcode_auth.py
git diff --check
```

现有 6 项离线契约测试覆盖：领取成功后必须回读、duplicate 必须确认、回读失败标待确认、回读无有效套餐不算成功、档位不可识别不算成功、已有有效套餐跳过领取。测试使用假客户端，不发送真实账号请求。

---

<a id="design"></a>
## 🧱 设计原则

- **状态优先**：先读服务端套餐，再决定是否写入。
- **动作与结果分离**：claim 回执和最终套餐状态分开报告。
- **严格失败**：回读失败或档位不可识别时返回待确认或失败，不用猜测补齐。
- **幂等**：已有有效套餐直接结束，不重复提交。
- **凭据隔离**：单账号失败不阻断其他账号。
- **最小依赖**：主脚本单文件，运行只依赖 `requests`。

---

<a id="privacy"></a>
## 🔒 隐私与数据边界

- 仓库不包含账号、access token、refresh token 或设备信息。
- 日志不输出完整凭据；认证错误只保留必要状态和业务信息。
- `atomcode_state.json`、`.env` 和本机 auth 文件均不应提交。
- 请只在本地环境变量、青龙环境变量或 GitHub Secrets 中保存凭据。
- Issue、日志和聊天记录公开前必须脱敏。

---

<a id="layout"></a>
## 📁 目录结构

```text
AtomCode-Daily/
├── .github/workflows/daily.yml   # GitHub Actions
├── assets/                       # 微信与支付宝赞赏码
├── docs/                         # 五份翻译文档
├── tests/                        # 离线契约测试
├── atomcode_auth.py              # auth 文件转换工具
├── atomcode_daily.py             # 主脚本
├── CHANGELOG.md
├── LICENSE
├── NOTICE.md
├── requirements.txt
└── README.md
```

---

<a id="release"></a>
## 📦 版本与发布

- 当前版本：`v1.0.3`，核心变更是领取后重读 `status-v2` 并校验套餐档位与有效期。
- 历史版本保留在 [Releases](https://github.com/L0NE-6/AtomCode-Daily/releases)，不覆盖、不合并。
- 青龙或 Actions 需要固定版本时，请下载对应 Release 资产；跟随最新行为使用 `main`。
- 变更记录见 [CHANGELOG.md](CHANGELOG.md)。

---

<a id="disclaimer"></a>
## ⚠️ 免责声明

本项目用于个人学习与合法自动化实践。使用者应自行确认账号、服务条款、所在地区规则和部署环境；因误配置、凭据泄露、服务策略变化或账号异常产生的风险由使用者自行承担。

---

<a id="donate"></a>
## ☕ 支持与投喂：微信与支付宝赞赏码

脚本完全免费，赞赏纯自愿，不影响任何功能。

<p align="center">
  <img src="assets/donate-wechat.png" width="260" alt="微信赞赏码" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="assets/donate-alipay.jpg" width="260" alt="支付宝收款码" />
</p>

<p align="center"><sub>微信支付（左） · 支付宝（右）</sub></p>

> [!WARNING]
> 收款码仅用于向作者自愿赞赏。任何第三方以代下载、代部署、内部资源、会员群或官方授权为由收费，均与作者无关，请停止付款并保留证据。

---

<a id="feedback"></a>
## 💬 反馈与贡献：Star History

提交问题前请先移除 token、Cookie、手机号、UID 和设备标识，并附上可复现步骤、脚本版本和脱敏日志。

- [Issues](https://github.com/L0NE-6/AtomCode-Daily/issues)
- [Pull Requests](https://github.com/L0NE-6/AtomCode-Daily/pulls)
- [Releases](https://github.com/L0NE-6/AtomCode-Daily/releases)

<a href="https://star-history.com/#L0NE-6/AtomCode-Daily&Date">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=L0NE-6/AtomCode-Daily&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=L0NE-6/AtomCode-Daily&type=Date&theme=light" />
    <img alt="AtomCode Daily Star History" src="https://api.star-history.com/svg?repos=L0NE-6/AtomCode-Daily&type=Date&theme=light" />
  </picture>
</a>

<div align="center">

[⭐ Star](https://github.com/L0NE-6/AtomCode-Daily) · [🍴 Fork](https://github.com/L0NE-6/AtomCode-Daily/fork) · L0NE-6 · DAILY SIGNAL

</div>
