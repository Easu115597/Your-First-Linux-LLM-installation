# Ubuntu 26.04 LTS 基礎環境

適用於全新安裝的 Ubuntu 26.04 LTS（amd64）伺服器。本文先建立可維護的基本環境；NVIDIA 驅動、Docker、Claude Code 與本地 LLM 請依後續文件分開安裝。

> Ubuntu 26.04 LTS 的代號為 Resolute Raccoon。安裝前請先確認實際版本：`lsb_release -a`、`uname -r`。

## 1. 更新系統並安裝基本工具

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install -y \
  build-essential ca-certificates curl wget git gnupg \
  vim nano htop tmux rsync unzip zip jq pciutils \
  openssh-server ufw
```

確認版本與服務：

```bash
lsb_release -a
uname -r
git --version
ssh -V
systemctl status ssh --no-pager
```

## 2. 建議的目錄規劃

不要把模型、程式碼、資料庫與服務設定全部混在同一個目錄。

```text
~/projects/          # Git 專案與 Agent 工作目錄
~/ai-models/         # 個人模型檔案（依磁碟容量調整）
~/data/              # 專案資料
/opt/services/       # 長期運行的服務設定（由管理者維護）
```

建立個人工作目錄：

```bash
mkdir -p ~/projects ~/ai-models ~/data
```

## 3. 時區、時間同步與磁碟檢查

```bash
timedatectl
sudo timedatectl set-timezone Asia/Taipei
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINTS,MODEL
df -hT
```

不要對含有資料的磁碟直接執行格式化指令。先用 `lsblk`、`blkid` 確認裝置名稱、檔案系統及掛載點，再決定是否分割與格式化。

## 4. 系統更新策略

伺服器可先採手動更新，確認服務相容後再重開機：

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
```

若啟用自動安全更新，請先了解 unattended-upgrades 的重開機策略，避免在模型推論或長時間任務期間意外重啟。

## 5. 後續安裝順序

1. 完成 SSH 與防火牆設定（見 [安全與遠端管理](security-and-remote-access.md)）。
2. 安裝 NVIDIA 驅動並確認主機端 `nvidia-smi` 正常。
3. 安裝 Docker Engine 與 NVIDIA Container Toolkit（見 [Docker 與 GPU 容器](../02-container/docker-nvidia-container-toolkit.md)）。
4. 安裝 uv 與 Claude Code（見 [Claude Code](../03-ai-development/claude-code.md)）。
5. 最後才依模型需求部署 vLLM／Ollama。

## 官方文件

- [Ubuntu Releases](https://wiki.ubuntu.com/Releases)
- [Ubuntu Server documentation](https://ubuntu.com/server/docs)
