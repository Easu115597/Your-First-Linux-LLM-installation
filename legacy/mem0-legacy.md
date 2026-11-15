# Mem0ai 全域安裝與多帳號共用部署指南（舊版封存）

> 本檔保留舊版內容供追溯，不建議直接照抄。共享可寫入的 Python venv、寬鬆目錄權限、公開綁定 Qdrant port 等做法有安全及維運風險。請以目前的 [Mem0 安裝筆記](../mem0環境安裝.md) 為準。

## 1. 系統依賴安裝

sudo apt update
sudo apt install -y python3 python3-pip python3-venv

## 2. 建立共享虛擬環境（舊做法）

sudo mkdir -p /opt/mem0-venv
sudo python3 -m venv /opt/mem0-venv
sudo chown -R root:users /opt/mem0-venv
sudo chmod -R 0775 /opt/mem0-venv
sudo usermod -aG users <your_username>

## 3. 安裝 Mem0ai 與核心套件

source /opt/mem0-venv/bin/activate
pip install --upgrade pip
pip install mem0ai sentence-transformers qdrant-client
deactivate

## 4. 部署 Qdrant（舊做法）

sudo mkdir -p /opt/qdrant_data
sudo chmod 0775 /opt/qdrant_data
sudo docker run -d \
  --name qdrant \
  -p 6333:6333 \
  -v /opt/qdrant_data:/qdrant/storage \
  --restart unless-stopped \
  qdrant/qdrant

## 5. 配置 Ollama

curl -fsSL https://ollama.com/install.sh | sh
sudo mkdir -p /opt/ollama_models
sudo chmod 0775 /opt/ollama_models
echo "OLLAMA_MODELS=/opt/ollama_models" | sudo tee -a /etc/environment
export OLLAMA_MODELS=/opt/ollama_models
ollama pull llama3.1:8b

## 6. HuggingFace 共用快取

sudo mkdir -p /opt/huggingface
sudo chmod 0775 /opt/huggingface
echo "HF_HOME=/opt/huggingface" | sudo tee -a /etc/environment
export HF_HOME=/opt/huggingface

## 7. 舊版 Mem0 設定範例

~~~yaml
version: v1.1
vector_store:
  provider: qdrant
  config:
    host: localhost
    port: 6333
    collection_name: memories
llm:
  provider: ollama
  config:
    model: llama3.1:8b
    ollama_base_url: http://localhost:11434
embedder:
  provider: sentence-transformers
  config:
    model: all-MiniLM-L6-v2
~~~

## 8. 舊版測試範例

~~~python
from mem0 import Memory

config_path = "/opt/mem0_config/mem0_config.yaml"
memory = Memory.from_config(config_path)
memory.add("我是台南人，喜歡 bonsai", user_id="user_01")
res = memory.search("該使用者喜歡什麼植物？", user_id="user_01", limit=3)
print(res)
~~~

## 舊版注意事項

- 共享 venv 不應開放所有使用者寫入。
- 不應無限制對外公開 Qdrant 6333。
- Ollama 11434 預設應限制在本機或可信任網路。
- Mem0 API、provider 設定與回傳格式需依實際安裝版本核對。
