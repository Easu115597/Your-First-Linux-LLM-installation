# SSH、防火牆與 Tailscale 遠端管理

目標：讓伺服器只接受必要的管理連線。建議以 Tailscale 作為跨地點私有網路，SSH 僅允許內網或 Tailscale 介面連入；不要把 SSH、RDP、資料庫管理介面直接暴露到公網。

## 1. 先確認 SSH 可用

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh --no-pager
hostname -I
```

先從另一台 LAN 電腦測試：

```bash
ssh <linux_user>@<server_lan_ip>
```

確認可以登入後，再進行防火牆調整。若尚未測試第二條管理通道，不要直接啟用會阻斷目前連線的規則。

## 2. 建議改用 SSH 金鑰

在管理端電腦建立金鑰（若已有可沿用）：

```bash
ssh-keygen -t ed25519 -a 64
ssh-copy-id <linux_user>@<server_lan_ip>
```

確認可使用金鑰登入後，再考慮停用密碼登入。修改前先保留一個已登入的 SSH 工作階段，避免設定錯誤造成鎖在伺服器外。

編輯：

```bash
sudoedit /etc/ssh/sshd_config
```

確認或設定：

```text
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

驗證設定並重新載入：

```bash
sudo sshd -t
sudo systemctl reload ssh
```

請先確認金鑰登入成功，再停用密碼登入。若使用雲端映像、集中式登入或其他驗證機制，需先確認不會因此失去管理權限。

## 3. 安裝 Tailscale

官方安裝方式：

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

依終端顯示的網址完成登入，然後檢查：

```bash
tailscale status
tailscale ip -4
```

Tailscale 官方套件清單已列出 Ubuntu 26.04（Resolute）的安裝套件。若偏好不用安裝腳本，可依官方套件頁面手動設定 apt repository。

## 4. UFW 防火牆

**先確認 Tailscale 已連線，且你能透過 Tailscale SSH 或 LAN SSH 登入，再啟用以下規則。**

若 SSH 只允許從 Tailscale 介面進入：

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw status numbered
sudo ufw enable
```

若還需要 LAN SSH，另外加入你自己的 LAN 網段（請替換成實際網段）：

```bash
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
```

不要照抄範例網段；請先用 `ip route` 確認實際網段。不要同時保留不必要的 `sudo ufw allow 22/tcp`，否則會讓 SSH 對所有可到達主機的來源開放。

## 5. RDP／XRDP（只有需要 Linux 桌面時才裝）

無桌面伺服器不需要安裝 XRDP。若確實需要：

- 只允許 Tailscale 或指定 LAN 網段連入 TCP 3389。
- 不要使用 `sudo ufw allow 3389/tcp` 對所有來源開放。
- 先確認桌面環境、登入管理器及 XRDP session 相容，再部署。

例如只允許 Tailscale：

```bash
sudo ufw allow in on tailscale0 to any port 3389 proto tcp
```

## 6. 重要安全原則

- 不要把 SSH、RDP、Qdrant 6333、Ollama 11434 或 vLLM 8000 直接開到公網。
- Tailscale 是私有網路連線工具，不代表每個 tailnet 裝置都應有伺服器管理權；請在管理控制台設定最小權限 ACL／Grants。
- Docker 的 `docker` 群組具有近似 root 的權限；只加入可信任的管理帳號。
- Agent 工作目錄與交易程式、金鑰、個人資料分開；不要讓 Agent 預設讀取整個 home 目錄。
- 不要把 API key、SSH 私鑰、`.env` 或存取 token 提交到 Git。

## 官方文件

- [Tailscale：Install on Linux](https://tailscale.com/docs/install/linux)
- [Tailscale：Ubuntu stable packages](https://pkgs.tailscale.com/stable/)
- [Ubuntu UFW documentation](https://help.ubuntu.com/community/UFW)
- [Ubuntu OpenSSH Server](https://ubuntu.com/server/docs/openssh-server)
