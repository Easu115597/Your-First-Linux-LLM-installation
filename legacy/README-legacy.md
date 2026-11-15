# Linux 安裝手順與環境安全配置指南

## 一、系統基本安裝與安全加固

### 1. 更新系統 & 安裝必要工具
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential curl wget git vim htop tmux rsync zsh ufw -y

### 2. 磁碟加密（乾淨新機建議於安裝時選 LUKS 全磁碟加密）
> 建議於安裝作業系統時啟用

### 3. 防火牆設定（UFW 為例）
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
sudo ufw status numbered

### 4. 防毒與惡意程式偵測工具

安裝並執行 ClamAV：
sudo apt install clamav clamav-daemon -y
sudo freshclam
sudo systemctl enable --now clamav-freshclam.service
clamscan --version
sudo clamscan -r /

使用／安裝 Linux Malware Detect (LMD)：
sudo apt update && sudo apt upgrade -y
sudo apt install wget tar -y
wget http://www.rfxn.com/downloads/maldetect-current.tar.gz
tar -zxvf maldetect-current.tar.gz
cd maldetect-*
sudo ./install.sh
maldet --version
sudo maldet -u
sudo maldet -a /

檢查 rootkit：
sudo apt install rkhunter chkrootkit
sudo rkhunter --check
sudo chkrootkit

### 5. IDS/IPS 基礎（選用 snort/suricata）
sudo apt install snort -y
或 sudo apt install suricata -y

---

## 二、AI/本地 LLM 工具安裝

### 6. Docker & Python 基本環境
sudo apt install docker.io docker-compose python3 python3-pip -y

### 7. Node.js & npm
sudo apt install nodejs npm -y

### 8. Qdrant 本地安裝
docker run -d --name qdrant -p 6333:6333 qdrant/qdrant

如遇 docker.sock permission denied，可將帳號加入 docker group：
sudo usermod -aG docker $USER
之後登出再登入。

### 9. Ollama (本地 LLM)
參考 https://ollama.com/download 或：
curl https://ollama.com/install.sh | sh

### 10. HuggingFace sentence-transformers
pip3 install sentence-transformers

### 11. mem0 / AnythingLLM / Void / LM Studio
參考各專案 README，建議以 docker 或 python venv 部署。

---

## 三、桌面、中文輸入法與遠端功能

### 12. 桌面環境 (GNOME/XFCE等)
sudo apt install xfce4 xfce4-goodies -y

### 13. Google Chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb

### 14. Terminator 終端分割視窗
sudo apt install terminator -y

### 15. Fcitx 輸入法平台與嘸蝦米
sudo apt install fcitx5 fcitx5-configtool fcitx5-table-extra
sudo apt install fcitx5-table-boshiamy
sudo apt install fcitx fcitx-table fcitx-table-zhuyin -y

設定環境變數（依桌面環境及輸入法版本調整）：
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx

登出重新登入後，使用 fcitx5-configtool 加入所需輸入法。

### 16. 遠端 xrdp
sudo apt install xrdp -y
sudo systemctl enable xrdp
sudo ufw allow 3389/tcp

> 歷史筆記曾建議開放 3389；新部署請勿直接對公網開放，應限制在 Tailscale 或可信任 LAN。

### 17. Cloudflare Tunnel（如需）
wget -O cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/
cloudflared tunnel login

---

## 四、補充維運工具

- VLAN/網路規劃（由 UniFi 控管）
- backup tools: Timeshift, rsync

---

## NVIDIA 顯卡驅動（歷史筆記）

查詢系統可用 NVIDIA 驅動版本：
sudo apt update
sudo ubuntu-drivers devices

安裝推薦驅動：
sudo ubuntu-drivers autoinstall

手動指定版本（請先確認支援性）：
sudo apt install nvidia-driver-XXX

重開機後驗證：
nvidia-smi

舊筆記曾建議安裝 CUDA 12.2：
sudo apt install nvidia-cuda-toolkit

Docker GPU 支援（舊版 nvidia-docker2 流程，僅供歷史參考）：

curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-docker2
sudo systemctl restart docker
sudo docker run --gpus all nvidia/cuda:12.2.0-base nvidia-smi

---

## vLLM 專用環境（歷史範例）

~~~bash
deactivate 2>/dev/null
mkdir -p ~/vllm_server
cd ~/vllm_server
python3 -m venv venv-vllm
source venv-vllm/bin/activate
pip install --upgrade pip
pip install vllm
pip install openai
~~~

> 本檔為舊版安裝筆記封存，不代表所有指令仍適用 Ubuntu 26.04。尤其舊版 apt-key、nvidia-docker2、系統 pip、CUDA 固定版本與公開服務 port，請改看新版文件及各專案官方安裝說明。
