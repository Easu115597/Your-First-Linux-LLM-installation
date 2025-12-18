Mem0ai 安裝全手順（Linux, 多帳號共用模式）
1. 安裝 Python（建議系統預設已安裝，建議 3.8 以上版本）
bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
（如 Python 3.12+ 建議用 venv 虛擬環境；如多帳號共用，建議創建在 /opt 下）

2. 建立共享型虛擬環境 venv（所有帳號皆可使用）
bash
sudo mkdir -p /opt/mem0-venv
sudo python3 -m venv /opt/mem0-venv
sudo chown -R root:users /opt/mem0-venv
sudo chmod -R 0775 /opt/mem0-venv
加入共用群組 users（將所有用戶都加入該群組）

3. 安裝 Mem0ai 及相關依賴（在 venv 內執行）
bash
source /opt/mem0-venv/bin/activate
pip install --upgrade pip
pip install mem0ai sentence-transformers qdrant-client
deactivate
＊若 HuggingFace 其它套件依需求追加安裝。

4. 部署 Qdrant（本地向量資料庫）
bash
sudo mkdir -p /opt/qdrant_data
sudo chmod 0775 /opt/qdrant_data
sudo docker run -d \
  -p 6333:6333 \
  -v /opt/qdrant_data:/qdrant/storage \
  --restart=unless-stopped \
  qdrant/qdrant
＊所有帳號只要能存取 6333 port，皆可用同一資料目錄。

5. 安裝及配置 Ollama（本地 LLM server + 模型快取共用）
bash
curl -fsSL https://ollama.com/install.sh | sh
# 建共用模型快取
sudo mkdir -p /opt/ollama_models
sudo chmod 0775 /opt/ollama_models
echo "OLLAMA_MODELS_PATH=/opt/ollama_models" | sudo tee -a /etc/environment
ollama pull llama3.1:8b
（如需多模型，可重複拉取；OLLAMA啟動後自動開啟本地 11434 port ）

6. HuggingFace模型快取共用（句子嵌入）
bash
sudo mkdir -p /opt/huggingface
sudo chmod 0775 /opt/huggingface
echo "HF_HOME=/opt/huggingface" | sudo tee -a /etc/environment
7. 建立並配置 Mem0ai config（建議放在 /opt/mem0_config 或 /etc/mem0）
/opt/mem0_config/mem0_config.yaml（範本）

text
vector_store:
  provider: qdrant
  config:
    host: localhost
    port: 6333
    collection_name: memories
    path: /opt/qdrant_data
llm:
  provider: ollama
  config:
    model: llama3.1:8b
    ollama_base_url: http://localhost:11434
embedder:
  provider: sentence-transformers
  config:
    model: all-MiniLM-L6-v2
    cache_dir: /opt/huggingface
version: v1.1
8. 管理多帳號共用權限
所有資料目錄（Qdrant, Ollama, HuggingFace）都設 0775，群組要包含所有使用者。

各帳號執行前 source /opt/mem0-venv/bin/activate，即可共用虛擬環境及所有安裝套件。

9. 測試安裝及功能
main.py 範例：

python
from mem0 import Memory

memory = Memory.from_config('/opt/mem0_config/mem0_config.yaml')

memory.add('我是台南人，喜歡 bonsai', user_id='user1')
res = memory.search('bonsai 植物', user_id='user1', limit=3)
print(res['results'])
常見注意/排錯
pip 安裝遇 error，建議用 venv，避免破壞系統原生 Python。

Qdrant, Ollama, HuggingFace 快取路徑都設為共用目錄。

config 權限、embedding model dims 要與本地模型一致。

如遇某帳號讀寫權限問題，確認該用戶權限與群組關聯。

只需依上述步驟，無論有幾個登入帳號都可以共用同一份 Mem0ai 服務與資料，適用本地多用戶/LLM/嵌入/向量共用安全方案。如需擴充 systemd、自動啟動服務、API route、專案結構建議，再追問即可！
