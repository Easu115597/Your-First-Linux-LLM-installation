# 多張 NVIDIA GPU 與本地 LLM 部署注意事項

本文件針對混合使用 RTX 3060、RTX 3080 等消費級顯卡的 Linux 主機。GPU 型號、VRAM、PCIe 拓樸與散熱不同，部署前必須先盤點，不應直接假設多張卡會變成一張大顯卡。

## 1. 先盤點硬體

```bash
nvidia-smi -L
nvidia-smi --query-gpu=index,name,memory.total,driver_version,pci.bus_id --format=csv
nvidia-smi topo -m
lspci -tv
```

請保存輸出，確認每張卡的實際 VRAM、PCIe link 與 GPU index。相同型號也可能因不同板卡版本而有不同記憶體容量。

## 2. VRAM 不會自動合併

兩張 10 GB GPU 不會被作業系統當成一張連續 20 GB GPU。模型需由推論框架切分到多張卡，並且會產生跨 GPU 通訊與記憶體配置成本。

- Tensor Parallel（TP）會把模型計算切分到多張 GPU。
- Pipeline Parallel（PP）會把模型層分配到不同 GPU。
- GPU 型號混搭時，較慢的卡、較小的 VRAM 與 PCIe 拓樸可能限制整體效能。
- 必須保留 KV cache、CUDA graph、runtime buffer 等額外 VRAM；不能只用模型權重大小判斷能否啟動。

## 3. 建議先單卡，再逐步擴充

先用單張 GPU 跑通模型，再測試兩張同型號 GPU，最後才測混合型號。記錄每次的模型、量化格式、context length、batch token、GPU memory utilization、吞吐量與 OOM 狀況。

```bash
# 限制只使用指定 GPU；請先用 nvidia-smi 對照 index
CUDA_VISIBLE_DEVICES=0 vllm serve <model-id> \
  --host 127.0.0.1 \
  --port 8000
```

雙卡測試示例（實際參數需依模型、vLLM 版本與 GPU 拓樸調整）：

```bash
CUDA_VISIBLE_DEVICES=0,1 vllm serve <model-id> \
  --tensor-parallel-size 2 \
  --host 127.0.0.1 \
  --port 8000
```

不要把這個範例視為所有 RTX 3060／3080 混卡組合都能成功的保證。遇到 NCCL、P2P 或 OOM 問題時，先確認拓樸、driver、PyTorch、CUDA runtime 與 vLLM 版本相容，再針對問題調整。

## 4. vLLM 安裝原則

vLLM 對 Python、PyTorch、CUDA 與 GPU 架構有版本相依性。不要把不同教學中的安裝指令拼在一起，也不要把舊版 CUDA wheel 與新版 vLLM 任意混用。

建議每個 vLLM 版本使用獨立虛擬環境，依 vLLM 官方安裝頁面選擇對應的 Python／CUDA／PyTorch 安裝方式：

- [vLLM Installation](https://docs.vllm.ai/en/latest/getting_started/installation/)
- [vLLM GPU installation](https://docs.vllm.ai/en/latest/getting_started/installation/gpu/)

基本環境檢查：

```bash
nvidia-smi
python3 --version
python3 -m venv ~/venvs/vllm
source ~/venvs/vllm/bin/activate
python -m pip install --upgrade pip
```

接著依官方文件安裝相容的 vLLM wheel。不要在系統 Python 使用 `sudo pip install`。

## 5. API 服務安全

本地測試建議只綁定 loopback：

```bash
--host 127.0.0.1 --port 8000
```

若要讓 LAN／Tailscale 上其他主機使用，請先設定防火牆、API authentication 與來源限制，再調整 bind address。不要把沒有驗證的推論 API 直接暴露到公網。

## 6. 部署前檢查清單

- [ ] `nvidia-smi` 在主機端正常
- [ ] 記錄所有 GPU 的實際 VRAM 與 PCIe 拓樸
- [ ] 驅動版本與 CUDA runtime 相容
- [ ] vLLM／PyTorch／Python 版本來自同一套相容安裝指引
- [ ] 先完成單卡 smoke test
- [ ] 再測試同型號多卡與混卡
- [ ] context length 與 batch 設定留有 KV cache 空間
- [ ] API 僅對可信任網路開放
