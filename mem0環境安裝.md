code
Markdown
# Mem0ai 全域安裝與多帳號共用部署指南 (Linux)

本指南介紹如何在 Linux 伺服器上建置一套可供多個系統帳號共用的 Mem0ai 環境。

---

## 1. 系統依賴安裝
安裝 Python 基礎組件與虛擬環境支援。

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
2. 建立共享虛擬環境 (Virtual Env)
將虛擬環境置於 /opt 下，並透過群組權限管理讓所有開發者皆可使用。
code
Bash
# 建立目錄
sudo mkdir -p /opt/mem0-venv
sudo python3 -m venv /opt/mem0-venv

# 設定權限：讓 users 群組成員具備讀寫權限
sudo chown -R root:users /opt/mem0-venv
sudo chmod -R 0775 /opt/mem0-venv
提示： 請確保需要使用的帳號已加入 users 群組：
sudo usermod -aG users <your_username>
3. 安裝 Mem0ai 與核心套件
進入環境並安裝必要依賴。
code
Bash
source /opt/mem0-venv/bin/activate
pip install --upgrade pip
pip install mem0ai sentence-transformers qdrant-client
deactivate
4. 部署 Qdrant 向量資料庫
使用 Docker 啟動，並將資料持久化於共享目錄。
code
Bash
sudo mkdir -p /opt/qdrant_data
sudo chmod 0775 /opt/qdrant_data

sudo docker run -d \
  --name qdrant \
  -p 6333:6333 \
  -v /opt/qdrant_data:/qdrant/storage \
  --restart unless-stopped \
  qdrant/qdrant
5. 配置 Ollama (本地 LLM Server)
配置模型共享路徑，避免每個使用者重複下載模型。
code
Bash
# 安裝 Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 建立共用快取路徑
sudo mkdir -p /opt/ollama_models
sudo chmod 0775 /opt/ollama_models

# 設定環境變數 (寫入全域配置)
echo "OLLAMA_MODELS=/opt/ollama_models" | sudo tee -a /etc/environment
export OLLAMA_MODELS=/opt/ollama_models

# 拉取預設模型
ollama pull llama3.1:8b
6. 配置 HuggingFace 共享目錄
用於存放 sentence-transformers 等嵌入模型。
code
Bash
sudo mkdir -p /opt/huggingface
sudo chmod 0775 /opt/huggingface

# 設定環境變數
echo "HF_HOME=/opt/huggingface" | sudo tee -a /etc/environment
export HF_HOME=/opt/huggingface
7. 建立 Mem0ai 全域配置文件
請手動建立檔案 /opt/mem0_config/mem0_config.yaml 並貼入以下內容：
code
Yaml
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
8. 使用範例與測試
任何使用者只需進入虛擬環境，即可調用同一套服務。
範例程式碼 test_mem.py：
code
Python
from mem0 import Memory

# 載入共用配置
config_path = '/opt/mem0_config/mem0_config.yaml'
memory = Memory.from_config(config_path)

# 儲存記憶
memory.add('我是台南人，喜歡 bonsai', user_id='user_01')

# 檢索記憶
res = memory.search('該使用者喜歡什麼植物？', user_id='user_01', limit=3)
print(res)
執行方式：
code
Bash
source /opt/mem0-venv/bin/activate
python test_mem.py
⚠️ 常見問題 (Troubleshooting)
權限錯誤：若出現 Permission denied，請檢查使用者是否在 users 群組，並確認目錄權限為 0775。
變數生效：修改 /etc/environment 後，需重啟 Terminal 或執行 source /etc/environment。
向量維度：若更換 Embedder 模型，需清空 Qdrant collection 否則會因維度不符報錯。
最後更新日期: 2024-12-19
code
Code
---

### 為什麼之前的格式會亂掉？
1. **缺少空行**：在 Markdown 中，標題 (`#`)、文字和代碼塊 (` ``` `) 之間**一定要有一行空白**。
2. **語法高亮標籤**：我為代碼塊加上了 `bash`, `yaml`, `python` 等標籤，這樣 GitHub 就會幫指令上色（Syntax Highlighting）。
3. **區隔線**：我加入了 `---` 區隔線，這樣在 GitHub 上看起來會更有層次感，不會所有的字都擠在一起。

你現在把上面這一整段代碼直接複製到你的 `mem0環境安裝.md` 檔案中，按下 Save，畫面就會變得很專業了！
