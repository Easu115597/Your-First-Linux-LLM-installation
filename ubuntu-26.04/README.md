# Ubuntu 26.04 LTS：獨立安裝指南

這是一套**獨立於原始安裝筆記**的 Ubuntu 26.04 LTS 安裝文件。

- 原版文件仍在 repository 根目錄，保持原樣。
- 本資料夾只記錄 Ubuntu 26.04 LTS 的新安裝流程。
- 未來如果新增其他 Ubuntu 版本，請建立另一個版本資料夾，不要覆寫本資料夾或根目錄舊文件。

## 建議安裝順序

1. [Ubuntu 26.04 基礎環境](01-system/ubuntu-26.04-baseline.md)
2. [SSH、防火牆與 Tailscale](01-system/security-and-remote-access.md)
3. [Docker Engine 與 NVIDIA Container Toolkit](02-container/docker-nvidia-container-toolkit.md)
4. [Python 專案環境：uv](03-ai-development/python-uv.md)
5. [Claude Code 與 Remote Control](03-ai-development/claude-code.md)
6. [Agent 長時間工作與隔離](03-ai-development/agent-operations.md)
7. [多 GPU 與 vLLM](04-local-llm/multi-gpu-vllm.md)
8. [Mem0（選用）](05-optional/mem0-setup.md)

## 安裝前提醒

- 先確認主機確實安裝 Ubuntu 26.04 LTS。
- 先完成基本系統與遠端管理，再安裝 GPU driver。
- 主機端 `nvidia-smi` 正常後，才設定 NVIDIA Container Toolkit 或 vLLM。
- 不要直接把 SSH、RDP、Qdrant、Ollama、vLLM API 暴露到公網。
- 不要把 API key、交易金鑰、SSH 私鑰或 `.env` 提交到 Git。
- 文件中的安裝指令尚未在你的實體主機逐條執行；請依硬體與官方版本矩陣驗證。

## 其他版本

| 版本 | 文件位置 | 狀態 |
|---|---|---|
| 原始／舊版 Linux 筆記 | repository 根目錄 `README.md` | 保留原樣 |
| Ubuntu 26.04 LTS | `ubuntu-26.04/` | 新版獨立文件 |

## 官方參考

- https://documentation.ubuntu.com/release-notes/26.04/
- https://docs.docker.com/engine/install/ubuntu/
- https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/
- https://code.claude.com/docs/en/
- https://docs.vllm.ai/en/latest/
