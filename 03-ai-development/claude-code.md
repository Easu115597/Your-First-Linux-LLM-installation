# Claude Code：Ubuntu 26.04 安裝與遠端工作流程

Claude Code 是在本機執行的 coding agent，能讀寫專案檔案、執行 shell 指令及使用 MCP。使用 Anthropic 服務時，模型推論由雲端處理；安裝 Claude Code 本身不會使用本機 RTX GPU，也不需要為 Claude Code 安裝 CUDA 或 Node.js（Native Installer 會提供所需 runtime）。

## 1. 安裝必要工具

~~~bash
sudo apt update
sudo apt install -y curl git tmux
~~~

## 2. 安裝 Claude Code（官方 Native Installer）

~~~bash
curl -fsSL https://claude.ai/install.sh | bash
~~~

確認 PATH 後檢查：

~~~bash
claude --version
claude doctor
~~~

若目前 shell 找不到 claude，重新開啟終端機或依安裝程式提示將 ~/.local/bin 加入 PATH。

## 3. 登入與第一次啟動

進入專案目錄後執行：

~~~bash
mkdir -p ~/projects/sample-project
cd ~/projects/sample-project
git init
claude
~~~

依畫面完成登入及授權。一般 Claude Code 可使用合資格的 Claude 訂閱或支援的 API provider；但 **Remote Control 另有資格限制**，API key 本身不支援 Remote Control。

第一次操作時，建議先讓 Agent 讀取文件、解釋專案結構，再逐項授權必要的讀寫與命令執行權限。不要一開始就允許不受限制的 shell、root 或整個 home 目錄存取。

## 4. 讓 SSH 中斷後工作仍繼續：tmux

Remote Control 連線依賴本機 Claude Code process 持續運行。若 SSH 中斷或關閉終端，應將工作放在 tmux session 中：

~~~bash
tmux new -s claude-agent
cd ~/projects/sample-project
claude
~~~

離開但保留工作：按 Ctrl+B，再按 D。

重新連回：

~~~bash
tmux attach -t claude-agent
~~~

列出工作階段：

~~~bash
tmux ls
~~~

tmux 只負責保留終端工作階段，不是故障自動重啟服務的 process supervisor。若主機關機、process 結束或長時間無法連線，任務仍可能中斷。

## 5. 使用 Claude Code Remote Control

在專案目錄中執行：

~~~bash
claude remote-control
~~~

此模式會在終端顯示 session URL，可從瀏覽器或 Claude App 連線。也可以在互動式 session 啟用：

~~~bash
claude --remote-control
~~~

或在既有 Claude Code session 內輸入：

~~~text
/remote-control
~~~

**官方文件目前列出的條件與限制：**

- Remote Control 為 research preview，文件目前列出 Pro、Max、Team、Enterprise 計畫可用；Team／Enterprise 需要 Owner 在管理設定中啟用。
- 必須透過 claude.ai 登入；API key 不支援 Remote Control。
- 不支援 Amazon Bedrock、Google Cloud Agent Platform、Microsoft Foundry；若 ANTHROPIC_BASE_URL 指向 api.anthropic.com 以外的自訂 gateway／proxy，也會停用 Remote Control。
- Remote Control 只使用對外 HTTPS 連線，不需要在伺服器開 inbound port。
- 檔案存取與命令執行仍在本機；但對話 transcript（包含訊息、回覆與工具活動）會儲存在 Anthropic 伺服器，以便跨裝置同步與重新連線。
- 若主機睡眠或網路中斷，連線可能暫時重連；官方文件指出，機器醒著但長時間（約 10 分鐘以上）無法連網時，session 會 timeout，需重新啟動。
- 若 SSH 斷線，tmux 可讓本機 process 繼續運行；若 process 已結束，tmux 無法復原該 session。

## 6. Agent 工作目錄與安全邊界

建議每個 Agent／專案使用獨立工作目錄與 Git branch：

~~~text
~/projects/
  agent-docs/
  agent-lab/
  local-llm/
~~~

基本原則：

1. 不要在含有交易 API key、錢包、SSH 私鑰或個人資料的目錄直接啟動 Agent。
2. 將 secrets 放在專案外的權限受限檔案或 secret manager；不要提交到 Git。
3. 對刪檔、部署、資料庫 migration、對外發送訊息與任何交易操作保留人工確認。
4. 多個 Agent 不要同時修改同一個工作目錄；可用獨立 Git worktree 或 branch 隔離。
5. MCP server 僅提供完成任務所需的最小權限。

## 7. Claude Code 與本地 LLM 的分工

| 工作 | 建議執行位置 |
|---|---|
| Claude Code coding agent | Ubuntu 主機 CPU／RAM + Anthropic 雲端模型 |
| vLLM／Ollama 本地推論 | 本機 NVIDIA GPU |
| Embedding／reranker | 本機 GPU 或 CPU，依負載決定 |
| Agent 專案檔案與 Git | 獨立工作目錄 |
| 跨地點遠端管理 | Tailscale + SSH／Remote Control |

Claude Code 與 vLLM 可以安裝在同一台伺服器，但兩者的模型運算路徑不同。若本地 LLM 同時高負載，仍需留意 CPU、RAM、磁碟 I/O 與 GPU 資源競爭。

## 官方文件

- https://code.claude.com/docs/en/setup
- https://code.claude.com/docs/en/remote-control
- https://code.claude.com/docs/en/security
- https://man7.org/linux/man-pages/man1/tmux.1.html
