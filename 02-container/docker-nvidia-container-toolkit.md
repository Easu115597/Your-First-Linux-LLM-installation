# Docker Engine 與 NVIDIA Container Toolkit（Ubuntu 26.04）

本文件使用 Docker 官方 apt repository，並以 NVIDIA Container Toolkit 將主機 GPU 提供給容器。先安裝並確認 NVIDIA 主機驅動正常，再設定容器 GPU runtime。

## 1. 移除可能衝突的舊套件

若這是全新系統，可先檢查是否裝過 Ubuntu 發行版的 Docker 套件：

```bash
dpkg -l | grep -E 'docker.io|docker-compose|containerd|runc'
```

Docker 官方文件列出的衝突套件包括 `docker.io`、`docker-compose`、`docker-compose-v2`、`containerd`、`runc` 等。**不要在已運行服務的主機上直接移除**；先備份容器資料並確認影響。

## 2. 安裝 Docker Engine 官方版本

```bash
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

驗證：

```bash
sudo systemctl enable --now docker
sudo docker run --rm hello-world
docker compose version
```

建議先使用 `sudo docker`。若決定讓特定帳號免 sudo 使用 Docker：

```bash
sudo usermod -aG docker "$USER"
```

登出再登入才會套用群組。注意：Docker 群組成員可取得近似 root 的主機權限，不應把一般使用者或不受信任的 Agent 加入此群組。

## 3. 安裝 NVIDIA 主機驅動

先確認 GPU 與系統建議的驅動：

```bash
lspci -nn | grep -i nvidia
sudo apt update
sudo ubuntu-drivers devices
```

可使用 Ubuntu 建議的驅動安裝流程：

```bash
sudo ubuntu-drivers install
sudo reboot
```

重新登入後：

```bash
nvidia-smi
```

只有在主機端 `nvidia-smi` 正常後，才繼續安裝容器工具。不要把 CUDA Toolkit、NVIDIA driver、容器 CUDA runtime 當成同一個套件；一般執行 PyTorch／vLLM 容器通常不需要先在主機安裝完整 CUDA Toolkit。

## 4. 安裝 NVIDIA Container Toolkit

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## 5. 驗證容器能否看到 GPU

請選擇與已安裝驅動相容的 CUDA base image tag；不要盲目使用比驅動支援版本更新的 CUDA runtime。

```bash
sudo docker run --rm --gpus all \
  nvidia/cuda:13.2.0-base-ubuntu24.04 nvidia-smi
```

若該 tag 不存在或與目前驅動不相容，請至 NVIDIA CUDA container tags 選擇有效且相容的版本。

## 6. 常見問題排查

```bash
# 主機驅動
nvidia-smi

# Docker 服務
sudo systemctl status docker --no-pager

# NVIDIA runtime 設定
sudo nvidia-ctk runtime list
sudo cat /etc/docker/daemon.json

# Docker GPU 測試
sudo docker run --rm --gpus all \
  nvidia/cuda:13.2.0-base-ubuntu24.04 nvidia-smi
```

如果主機端 `nvidia-smi` 已經失敗，先處理驅動／核心模組／NVML 版本不一致，不要先重裝 Docker。

## 官方文件

- [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [NVIDIA Container Toolkit：Supported platforms](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/supported-platforms.html)
- [NVIDIA Container Toolkit：Install guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
- [NVIDIA Driver Installation Guide for Ubuntu](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/latest/ubuntu.html)
