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
安裝並執行 ClamAV 病毒掃描
sudo apt install clamav clamav-daemon -y

sudo freshclam # 病毒庫更新
sudo systemctl enable --now clamav-freshclam.service # 讓系統自動定時更新病毒庫

clamscan --version #確認病毒庫版本
sudo clamscan -r /

使用/安裝 Linux Malware Detect (LMD)流程

更新系統＋安裝 wget 壓縮工具

bash
sudo apt update && sudo apt upgrade -y
sudo apt install wget tar -y
下載官方原始碼套件

bash
wget http://www.rfxn.com/downloads/maldetect-current.tar.gz
解壓縮安裝包並安裝

bash
tar -zxvf maldetect-current.tar.gz
cd maldetect-*
sudo ./install.sh
驗證安裝並更新病毒庫

bash
maldet --version
sudo maldet -u
後續使用範例
全盤掃描

bash
sudo maldet -a /

檢查 rootkit

sudo apt install rkhunter chkrootkit

sudo rkhunter --check

sudo chkrootkit

#這些工具可以幫助定位常見的 Linux 惡意程式和木馬。

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

**如安裝有問題，(permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock)
把你的帳號加入 docker group
 sudo usermod -aG docker $USER

之後登出後再新運行docker run

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
sudo apt install fcitx5 fcitx5-configtool fcitx5-table-extra

sudo apt install fcitx5-table-boshiamy

sudo apt install fcitx fcitx-table fcitx-table-zhuyin -y

#設定環境變數讓系統識別 Fcitx 輸入法：

#編輯 /etc/environment 或用戶家目錄下的環境設定檔，加入：


GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx

登出重新登入，或重啟系統，使設定生效。

使用 Fcitx 配置工具（如 fcitx5-configtool）在輸入法列表中加入「嘸蝦米（boshiamy）」輸入法即可使用。

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
###NVIDIA 顯卡驅動
查詢系統可用 NVIDIA 驅動版本

sudo apt update

sudo ubuntu-drivers devices

會顯示建議 (recommended) 驅動版本。

2. 安裝推薦驅動（自動；或手動指定版本）

sudo ubuntu-drivers autoinstall

# 如要手動指定版本

sudo apt install nvidia-driver-XXX

（XXX 請用查到的版本號，如 535、550）

3. 驗證驅動安裝完成 
重開機後，執行：

nvidia-smi

顯示 GPU 資訊才算驅動正確裝好。

4. 可再安裝 CUDA Toolkit 等延伸（如要用 PyTorch, vLLM, CUDA 環境）
例如安裝 CUDA 12.2：

sudo apt install nvidia-cuda-toolkit

若需最新版可依官方指令下載 keyring 或用 runfile 安裝。

5. Docker GPU 支援（如你要用 docker 跑 AI）

還需安裝 nvidia-docker：


curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-docker2
sudo systemctl restart docker

之後即可用：

sudo docker run --gpus all nvidia/cuda:12.2.0-base nvidia-smi


備註：
如有專案（mem0、AnythingLLM、Ollama、Qdrant）需進行設定，請於「AI工具」區段參考其官方 README，統一建議用 docker 或 Python venv 隔離。
