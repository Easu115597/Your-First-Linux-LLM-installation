# Your-First-Linux-LLM-installation
First Linux LLM installation
# Linux 安裝手順與環境安全配置指南

## 一、系統基本安裝與安全加固

### 1. 更新系統 & 安裝必要工具
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential curl wget git vim htop tmux rsync zsh ufw -y



### 2. 磁碟加密（乾淨新機建議於安裝時選 LUKS 全磁碟加密）  
> *建議於安裝作業系統時啟用*

### 3. 防火牆設定（UFW 為例）
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
sudo ufw status numbered

 

### 4. 防毒與惡意程式偵測工具
sudo apt install clamav clamav-daemon -y
sudo freshclam # 病毒庫更新
sudo apt install linux-malware-detect -y
sudo maldet -u # LMD 病毒庫更新

 

### 5. IDS/IPS 基礎（選用 snort/suricata）
sudo apt install snort -y

或 sudo apt install suricata -y
 
--- 

## 二、AI/本地 LLM 工具安裝

### 6. Docker & Python 基本環境
sudo apt install docker.io docker-compose python3 python3-pip -y

 

### 7. Node.js & npm
sudo apt install nodejs npm -y

 

### 8. Qdrant本地安裝  
docker run -d --name qdrant -p 6333:6333 qdrant/qdrant

 
> 或參考 Qdrant 官網說明安裝到本機

### 9. Ollama (本地 LLM)
參考[Ollama官網](https://ollama.com/download)或：
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
至 [Chrome官網](https://www.google.com/chrome/) 下載 deb 檔，或：
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb

 

### 14. Terminator 終端分割視窗
sudo apt install terminator -y

 

### 15. Fcitx 輸入法平台與嘸蝦米
sudo apt install fcitx fcitx-table fcitx-table-zhuyin -y

嘸蝦米需額外下載表格/engine，依官方步驟安裝
 

### 16. 遠端 xrdp 用於本地遠端桌面
sudo apt install xrdp -y
sudo systemctl enable xrdp
sudo ufw allow 3389/tcp # 本機用(勿對外開放)

 
> 非常建議搭配 VPN/Cloudflared Tunnel，也可考慮只開啟 SSH 隧道穿透。

### 17. Cloudflare Tunnel（如需）
wget -O cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/
cloudflared tunnel login

 
> 詳細串接 xrdp/VNC/SSH 請參考 Cloudflare Tunnel 文件。

---

## 四、補充維運工具

- VLAN/網路規劃（由 UniFi 控管）
- backup tools: Timeshift, rsync

---

備註：
如有專案（mem0、AnythingLLM、Ollama、Qdrant）需進行設定，請於「AI工具」區段參考其官方 README，統一建議用 docker 或 Python venv 隔離。
