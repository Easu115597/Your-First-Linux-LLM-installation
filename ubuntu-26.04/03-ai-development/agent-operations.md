# AI Agent 長時間工作與多 Agent 隔離

本文件介紹 Claude Code 等 Agent 在 Linux 主機上的基本運作方式。tmux、systemd、Tailscale、Git worktree 解決的是不同問題，不能互相取代。

## 1. 元件角色

| 元件 | 用途 | 不負責的事情 |
|---|---|---|
| Claude Code | 編輯程式、理解專案、執行工具 | 不保證主機關機後仍繼續 |
| tmux | SSH 中斷後保留終端 session | 不會自動修復程式錯誤 |
| systemd | 開機啟動、服務監控與重啟 | 不提供 Agent 任務隔離 |
| Tailscale | 私有網路連線 | 不會自動替每個服務做好授權 |
| Git branch/worktree | 隔離程式碼變更 | 不隔離作業系統權限或 secrets |

## 2. 使用 tmux 保留互動式工作

~~~bash
tmux new -s agent-one
cd ~/projects/my-agent-project
claude
~~~

Detach：Ctrl+B，然後 D。

重新連線：

~~~bash
tmux attach -t agent-one
~~~

## 3. 多 Agent 使用獨立工作目錄

不要讓兩個 Agent 同時任意修改同一份工作樹。可用 Git worktree：

~~~bash
cd ~/projects/my-agent-project
git worktree add ../my-agent-project-agent-a -b agent-a
git worktree add ../my-agent-project-agent-b -b agent-b
git worktree list
~~~

各 Agent 在不同目錄工作，完成後由人員檢查 diff，再整合回主分支。

## 4. systemd 適用於非互動式服務

長期 API、worker、排程程式可考慮 systemd service。互動式 Claude Code session 通常先使用 tmux；若要讓 Agent 在開機後自動執行，應額外設計明確的任務入口、日誌、失敗處理與權限，不建議把互動式 shell 簡單包成 root service。

## 5. Agent 安全邊界

- 每個 Agent 使用專屬 Linux 帳號或隔離容器；不要預設給 sudo。
- 工作目錄不要包含交易 API key、錢包、SSH 私鑰或個人資料。
- 不要把 Agent 加入 docker 群組，除非理解該群組近似 root 權限。
- MCP 僅授予任務必要的讀寫權限。
- 刪除檔案、部署服務、資料庫 migration、發送外部訊息與交易動作，應要求人工核准。
- 重要任務要有 Git commit、日誌與備份，避免只依賴聊天紀錄。

## 官方文件

- https://code.claude.com/docs/en/
- https://man7.org/linux/man-pages/man1/tmux.1.html
- https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html
- https://git-scm.com/docs/git-worktree
