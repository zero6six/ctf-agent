# CTF Agent

Autonomous CTF (Capture The Flag) solver that races multiple AI models against challenges in parallel. Built in a weekend, we used it to solve all 52/52 challenges and win **1st place at BSidesSF 2026 CTF**.

Built by [Veria Labs](https://verialabs.com), founded by members of [.;,;.](https://ctftime.org/team/222911) (smiley), the [#1 US CTF team on CTFTime in 2024 and 2025](https://ctftime.org/stats/2024/US). We build AI agents that find and exploit real security vulnerabilities for large enterprises.

## Zero6six Note

从夯到拉评价 Agent，这里给到拉完了，自己修改后接入模型 glm-4.5-air（别问我为什么不接 glm-5，那个由于并发限制高跑的更慢），做[题目](https://ctf.xidian.edu.cn/training/22?challenge=884)，最终成功拿到错误的 Flag，日志如下：

```powershell
python backend/cli.py init prob1 `
  --category web `
  --value 500 `
  --connection-info "http://127.0.0.1:57540" `
  --description "省流：http请求有很多讲究！试试吧！"

  python backend/cli.py run `
--challenge ./prob1 `
--offline `
--models generic-openai/glm-4.5-air

Challenge: prob1 (web, 500 pts)
[15:44:14] INFO     Sandbox started: 2dd44dbecac6
[15:44:14] INFO     [prob1/glm-4.5-air] Solver started
[15:44:14] INFO     [prob1/glm-4.5-air] Agent requesting model...
[15:44:18] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540', 'method': 'GET', 'body': ''})
[15:44:18] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────      <!DOCTYPE html>     <html lang="zh">     <...
[15:44:22] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540/stone_golem', 'method': 'GET', 'body': ''})
[15:44:22] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────      <!DOCTYPE html>     <html lang="zh">      ...
[15:44:24] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540/stone_golem?key=xdsec', 'method': 'GET', 'body': ''})
[15:44:24] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────      <!DOCTYPE html>     <html lang="zh">      ...
[15:44:27] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540/cloud_weaver', 'method': 'GET', 'body': ''})
[15:44:27] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────      <!DOCTYPE html>     <html lang="zh">     <...
[15:44:29] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540/cloud_weaver', 'method': 'POST', 'body': 'declaration=织云阁...)
[15:44:29] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────              <!DOCTYPE html>             <html ...
[15:44:33] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540/cloud_weaver', 'method': 'POST', 'body': 'declaration=织云阁...)
[15:44:33] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────              <!DOCTYPE html>             <html ...
[15:45:12] INFO     [prob1/glm-4.5-air] Tool call: bash({'command': 'curl -X POST "http://host.docker.internal:57540/cloud_weaver" -d "declaration=织云阁=第一" -...)
--------------此处省略若干--------------
[15:46:09] INFO     [prob1/glm-4.5-air] Tool result: <!DOCTYPE html>             <html lang="zh">             <head>                 <meta charset="UTF-8...
[15:46:13] INFO     [prob1/glm-4.5-air] Tool call: web_fetch({'url': 'http://host.docker.internal:57540/final_success', 'method': 'GET', 'body': ''})
[15:46:13] INFO     [prob1/glm-4.5-air] Tool result: HTTP 200 OK ────────────────────────────────────────      <!DOCTYPE html>     <html lang="zh">     <...
[15:46:27] INFO     [prob1/glm-4.5-air] Tool call: submit_flag({'flag': 'moecode{C0rz4u114710n5_9mvxg7N_pDv2M2_x5_r30ly_lx19ah=}'})

[MOCK] Submitting flag for prob1: moecode{C0rz4u114710n5_9mvxg7N_pDv2M2_x5_r30ly_lx19ah=}
[15:46:27] INFO     [prob1/glm-4.5-air] Tool result: MOCK SUBMISSION: moecode{C0rz4u114710n5_9mvxg7N_pDv2M2_x5_r30ly_lx19ah=}
[15:46:36] WARNING  Could not calculate cost for glm-4.5-air
[15:46:36] INFO     [prob1/glm-4.5-air] Analysis: 
[15:46:36] INFO     [prob1] Flag found by generic-openai/glm-4.5-air: moecode{C0rz4u114710n5_9mvxg7N_pDv2M2_x5_r30ly_lx19ah=}
[15:46:37] INFO     Sandbox stopped

FLAG FOUND: moecode{C0rz4u114710n5_9mvxg7N_pDv2M2_x5_r30ly_lx19ah=}

Cost Summary:
  prob1/glm-4.5-air: 646.0k in / 592.4k cached (92% hit) / 3.5k out | $0.00 | 142.0s
  Total: $0.00
```

上述日志中已经获取到了此题所需要的所有 flag 片段，只需拼接加 base64 解码即可，但是模型却连这一点都没有做到。

可见由于上下文不断膨胀，模型的理解能力和效率都会受到影响，且较差的模型影响更为明显，而好的模型又花费巨大，因此在当下的技术条件下，使用这个项目并不方便，不如 Codex 与 Github Copilot 之类。

## Results

| Competition | Challenges Solved | Result |
|-------------|:-:|--------|
| **BSidesSF 2026** | 52/52 (100%) | **1st place ($1,500)** |

The agent solves challenges across all categories — pwn, rev, crypto, forensics, web, and misc.

## How It Works

A **coordinator** LLM manages the competition while **solver swarms** attack individual challenges. Each swarm runs multiple models simultaneously — the first to find the flag wins.

```
                        +-----------------+
                        |  CTFd Platform  |
                        +--------+--------+
                                 |
                        +--------v--------+
                        |  Poller (5s)    |
                        +--------+--------+
                                 |
                        +--------v--------+
                        | Coordinator LLM |
                        | (Claude/Codex)  |
                        +--------+--------+
                                 |
              +------------------+------------------+
              |                  |                  |
     +--------v--------+ +------v---------+ +------v---------+
     | Swarm:          | | Swarm:         | | Swarm:         |
     | challenge-1     | | challenge-2    | | challenge-N    |
     |                 | |                | |                |
     |  Opus (med)     | |  Opus (med)    | |                |
     |  Opus (max)     | |  Opus (max)    | |     ...        |
     |  GPT-5.4        | |  GPT-5.4       | |                |
     |  GPT-5.4-mini   | |  GPT-5.4-mini  | |                |
     |  GPT-5.3-codex  | |  GPT-5.3-codex | |                |
     +--------+--------+ +--------+-------+ +----------------+
              |                    |
     +--------v--------+  +-------v--------+
     | Docker Sandbox  |  | Docker Sandbox |
     | (isolated)      |  | (isolated)     |
     |                 |  |                |
     | pwntools, r2,   |  | pwntools, r2,  |
     | gdb, python...  |  | gdb, python... |
     +-----------------+  +----------------+
```

Each solver runs in an isolated Docker container with CTF tools pre-installed. Solvers never give up — they keep trying different approaches until the flag is found.

## Quick Start

```bash
# Install
uv sync

# Build sandbox image
docker build -f sandbox/Dockerfile.sandbox -t ctf-sandbox .

# Configure credentials
cp .env.example .env
# Edit .env with your API keys and CTFd token

# Run against a CTFd instance
uv run ctf-solve \
  --ctfd-url https://ctf.example.com \
  --ctfd-token ctfd_your_token \
  --challenges-dir challenges \
  --max-challenges 10 \
  -v
```

## Coordinator Backends

```bash
# Claude SDK coordinator (default)
uv run ctf-solve --coordinator claude ...

# Codex coordinator (GPT-5.4 via JSON-RPC)
uv run ctf-solve --coordinator codex ...
```

## Solver Models

Default model lineup (configurable in `backend/models.py`):

| Model | Provider | Notes |
|-------|----------|-------|
| Claude Opus 4.6 (medium) | Claude SDK | Balanced speed/quality |
| Claude Opus 4.6 (max) | Claude SDK | Deep reasoning |
| GPT-5.4 | Codex | Best overall solver |
| GPT-5.4-mini | Codex | Fast, good for easy challenges |
| GPT-5.3-codex | Codex | Reasoning model (xhigh effort) |

## Sandbox Tooling

Each solver gets an isolated Docker container pre-loaded with CTF tools:

| Category | Tools |
|----------|-------|
| **Binary** | radare2, GDB, objdump, binwalk, strings, readelf |
| **Pwn** | pwntools, ROPgadget, angr, unicorn, capstone |
| **Crypto** | SageMath, RsaCtfTool, z3, gmpy2, pycryptodome, cado-nfs |
| **Forensics** | volatility3, Sleuthkit (mmls/fls/icat), foremost, exiftool |
| **Stego** | steghide, stegseek, zsteg, ImageMagick, tesseract OCR |
| **Web** | curl, nmap, Python requests, flask |
| **Misc** | ffmpeg, sox, Pillow, numpy, scipy, PyTorch, podman |

## Features

- **Multi-model racing** — multiple AI models attack each challenge simultaneously
- **Auto-spawn** — new challenges detected and attacked automatically
- **Coordinator LLM** — reads solver traces, crafts targeted technical guidance
- **Cross-solver insights** — findings shared between models via message bus
- **Docker sandboxes** — isolated containers with full CTF tooling
- **Operator messaging** — send hints to running solvers mid-competition

## Configuration

Copy `.env.example` to `.env` and fill in your keys:

```bash
cp .env.example .env
```

```env
CTFD_URL=https://ctf.example.com
CTFD_TOKEN=ctfd_your_token
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GEMINI_API_KEY=...
```

All settings can also be passed as environment variables or CLI flags.

## Requirements

- Python 3.14+
- Docker
- API keys for at least one provider (Anthropic, OpenAI, Google)
- `codex` CLI (for Codex solver/coordinator)
- `claude` CLI (bundled with claude-agent-sdk)

## Acknowledgements

- [es3n1n/Eruditus](https://github.com/es3n1n/Eruditus) — CTFd interaction and HTML helpers in `pull_challenges.py`
