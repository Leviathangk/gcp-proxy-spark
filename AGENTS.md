# Agent 工作约定

Claude Code 与 Codex 共用本文件（Claude Code 经 `.claude/CLAUDE.md` 导入）。冲突时以本文件为准。

## 目录地图

| 路径 | 内容 |
|---|---|
| `spark/` | Python 包：配置生成 `core.py`、部署 `deploy.py`、流量 `traffic.py`、订阅/页面 `web.py`、锁定版本 `versions.json`、页面资源 `assets/` |
| `tests/` | 单元/HTTP 测试、原生与实连 `native_check.py`、隔离计数 `linux_counters.py`、生命周期 `hosted_deploy.py`、发布审计 `release_audit.py` |
| `docs/` | `agent-handoff.md` 交接流程、`deployment.md`、`traffic.md`、`compatibility.md`、`verification.md`、`notice.md` 来源说明 |
| `scripts/` | 专用机依赖准备 `prepare-host.sh` |
| `.agents/skills/` | 部署技能（Codex 自动发现；Claude Code 经 `.claude/skills/` 入口加载） |
| `.github/` | CI、`SECURITY.md` |

## 交接（人工最少化）

- 用户交付任务时，按 `docs/agent-handoff.md` 读取交接单；缺失信息一次性列全再问，不逐条追问。
- 只在三个点停下等人：需要用户登录/授权、展示计划后的唯一确认、客户端真机导入。其余检查、验证、记录由 Agent 完成。
- 交接单中明确勾选的授权（例如云防火墙放行指定端口）视为显式授权；未勾选即视为未授权。

## 开发

- 先读 README、docs/verification.md；更改部署行为时读 docs/deployment.md。
- 只支持显式锁定的核心版本；更改版本后同步 versions.json、能力表、原生校验与实连测试结果。
- 配置通过结构化对象生成，不用字符串替换 YAML/JSON。私钥不得进入客户端输出。
- 本地环境：`uv venv .venv && uv pip install --python .venv/bin/python -e .`（或 `python3 -m venv .venv && .venv/bin/pip install -e .`）。
- 单元测试：`python -m unittest discover -v`；原生与实连：`spark fetch singbox|mihomo --directory .cache/bin` 后 `python -m tests.native_check --connections`。
- Linux 计数测试只能在独立 network namespace 内运行。不要在用户主机直接创建测试防火墙规则。
- 新增或移动仓库文件时同步 `tests/release_audit.py` 的白名单与本目录地图。

## 部署与授权

- 使用 `.agents/skills/gcp-proxy-deploy/SKILL.md`。先只读检查，再展示计划；明确确认后才 apply。
- 不自动创建 GCP 资源，不自动修改 IAM/云防火墙（交接单显式授权的端口放行除外），不接管 `/etc/s-box`，不升级旧生产服务。
- SSH、gcloud 登录令牌、IP/项目真实资料、订阅 URL、manifest、证书、历史计数都不要提交；交接单填好后同样不提交。
- 不能绕过 TLS、下载摘要、目标系统、端口冲突或所有权检查。失败应保留证据、停止扩张操作。
- 完成配置写入不等于部署完成；检查 systemd、可信 HTTPS、订阅导入、外网 TCP/UDP 连通、流量增量和续期。
- 无设备验证的 Shadowrocket 必须标“真机待验证”，不能写“全客户端兼容”。

## 发布

- 审核 Git 暂存白名单，运行 `python -m tests.release_audit`，确认没有生产数据。
- 保留 LICENSE 和 docs/notice.md。新增依赖记录许可证和用途。
- 发布后验证远端提交与 CI；测试未通过或实际未验证的场景写进 docs/verification.md。
- 不因为部署授权而推送无关项目、替换域名或改动其他服务。
