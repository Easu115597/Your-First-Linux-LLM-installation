# Your-First-Linux-LLM-installation

Linux 主機、NVIDIA GPU、本地 LLM 與 AI Agent 安裝筆記。

> **文件狀態**：Ubuntu 26.04 LTS 新文件以本 README 的導覽為準。根目錄舊筆記保留作為歷史參考；其中部分指令（例如舊版 apt-key、nvidia-docker2、系統層級 pip3、未限制來源的服務 port）不應直接套用到新系統。

## Ubuntu 26.04 LTS 建議安裝順序

1. [Ubuntu 26.04 基礎環境](01-system/ubuntu-26.04-baseline.md)
2. [SSH、防火牆與 Tailscale 遠端管理](01-system/security-and-remote-access.md)
3. [Docker Engine 與 NVIDIA Container Toolkit](02-container/docker-nvidia-container-toolkit.md)
4. [Claude Code 安裝與遠端工作流程](03-ai-development/claude-code.md)
5. [多 GPU 與 vLLM 部署注意事項](04-local-llm/multi-gpu-vllm.md)

## 原有文件（保留）

- [Mem0 多帳號共用部署筆記](mem0環境安裝.md) — 已更新共享 venv、資料夾權限及服務綁定的安全提醒。
- [環境安裝（舊版純文字筆記）](環境安裝) — 歷史參考，與上方 Mem0 文件部分重複，不作為 Ubuntu 26.04 的最新安裝指引。

## 通用原則

- 先確認作業系統、核心、GPU 與驅動版本，再安裝 CUDA／PyTorch／vLLM。
- 優先使用官方 apt repository、官方安裝文件與隔離的 Python 虛擬環境。
- Docker Engine 與 NVIDIA Container Toolkit 分開安裝；主機端 nvidia-smi 正常後才測試容器 GPU。
- 不要將 SSH、RDP、Qdrant、Ollama 或 vLLM API 無條件暴露到公網。
- 不要把 API key、SSH 私鑰、交易金鑰或 .env 提交到 Git。
- Agent 與交易程式、錢包、個人資料分開；對刪除、部署、對外訊息及交易保留人工確認。

## 官方參考

- [Ubuntu 26.04 LTS release notes](https://documentation.ubuntu.com/release-notes/26.04/)
- [Docker Engine installation](https://docs.docker.com/engine/install/ubuntu/)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/)
- [Claude Code documentation](https://code.claude.com/docs/en/)
- [vLLM documentation](https://docs.vllm.ai/en/latest/)
