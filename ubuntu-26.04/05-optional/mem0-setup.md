# Mem0 多帳號共用部署筆記（Linux）

本文件保留原本的多帳號部署方向，但**不建議把同一個可寫入的 Python virtualenv 開放給多個一般使用者共同修改**。虛擬環境中的套件與執行檔可能被任一可寫入者替換；更安全的方式是由管理者維護唯讀環境，或每個專案／使用者各自建立 venv。

## 1. 系統依賴

~~~bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip
~~~

Ubuntu 26.04 的 Python 版本與套件管理政策可能與舊版教學不同。不要對系統 Python 使用 sudo pip install；請使用 venv 或 uv。

## 2. 建立專案虛擬環境（建議）

~~~bash
mkdir -p ~/projects/mem0
cd ~/projects/mem0
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install mem0ai sentence-transformers qdrant-client
~~~

若希望由管理者維護共享環境，請將環境目錄設為管理者擁有、一般使用者唯讀，並以受控服務/API 提供功能；不要使用 chmod -R 0777 或讓所有使用者對同一 venv 有寫入權限。

## 3. 部署 Qdrant

建議先只綁定 loopback，避免資料庫 API 暴露到 LAN 或公網。先建立資料目錄：

~~~bash
mkdir -p ~/services/qdrant/storage
~~~

使用 Docker Compose：

~~~yaml
services:
  qdrant:
    image: qdrant/qdrant:latest
    container_name: qdrant
    restart: unless-stopped
    ports:
      - "127.0.0.1:6333:6333"
    volumes:
      - ./storage:/qdrant/storage
~~~

在包含上述內容的目錄執行：

~~~bash
docker compose up -d
docker compose ps
curl http://127.0.0.1:6333/healthz
~~~

正式環境請固定經測試的 image tag，並規劃備份與存取控制。Docker 對外發布的 port 可能繞過 UFW 的一般規則；因此應明確限制 bind address，並檢查 Docker firewall rules。

## 4. Ollama（選用）

若要使用 Ollama，請依官方安裝文件安裝，並確認服務只對需要的網路介面提供 API。預設本機使用時，請勿額外開放 11434 到公網。

官方安裝說明：https://ollama.com/download

## 5. Hugging Face 快取（選用）

個人使用可設定在家目錄：

~~~bash
mkdir -p ~/ai-models/huggingface
export HF_HOME="$HOME/ai-models/huggingface"
~~~

若要永久生效，將環境變數加入使用者自己的 shell 設定檔。多使用者共享快取時，需另外規劃群組、權限與磁碟配額，不建議直接將整個共享目錄設為所有人可寫。

## 6. Mem0 設定範例

以下是概念範例；provider 名稱、設定欄位與 API 會隨 Mem0 版本變更，實際使用前請核對目前版本的官方文件。

~~~yaml
version: "v1.1"
vector_store:
  provider: qdrant
  config:
    host: 127.0.0.1
    port: 6333
    collection_name: memories
llm:
  provider: ollama
  config:
    model: llama3.1:8b
    ollama_base_url: http://127.0.0.1:11434
embedder:
  provider: sentence-transformers
  config:
    model: all-MiniLM-L6-v2
~~~

## 7. 最小測試

~~~python
from mem0 import Memory

memory = Memory.from_config("mem0_config.yaml")
memory.add("測試記憶", user_id="test-user")
result = memory.search("測試記憶", user_id="test-user", limit=3)
print(result)
~~~

請依目前 Mem0 版本確認 Memory.from_config、add 與 search 的參數及回傳格式。

## 安全與維運提醒

- Qdrant、Ollama 與 vLLM API 預設只綁定 localhost；遠端存取請使用 VPN／反向代理與驗證。
- 不要把資料庫資料目錄與 Python venv 設成所有使用者可任意寫入。
- 備份 Qdrant storage 與設定檔；更新前先確認相容性。
- API key、密碼及私鑰不可寫入 Git。
- 官方文件：
  - https://docs.mem0.ai/
  - https://qdrant.tech/documentation/
  - https://docs.astral.sh/uv/
