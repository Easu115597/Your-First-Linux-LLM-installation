# Python 專案環境：uv 與 venv

Ubuntu 系統 Python 由 apt 管理。請勿使用 sudo pip 將套件安裝進系統 Python；每個專案應使用獨立虛擬環境，避免 mem0、vLLM、Agent SDK 等套件互相污染。

## 1. 安裝 uv

官方安裝方式：

~~~bash
curl -LsSf https://astral.sh/uv/install.sh | sh
~~~

重新載入 shell，確認：

~~~bash
uv --version
~~~

也可以依官方文件選擇 apt、pipx 或其他安裝方式。

## 2. 建立專案

~~~bash
mkdir -p ~/projects
cd ~/projects
uv init my-agent-project
cd my-agent-project
uv python pin 3.14
uv sync
~~~

Ubuntu 26.04 提供 Python 3.14，但第三方 AI 套件不一定已全面支援最新 Python。若特定套件（例如 vLLM）尚未支援 3.14，請依該套件官方矩陣選擇支援的 Python 版本，不要強行沿用系統預設版本。

## 3. 加入套件

~~~bash
uv add requests pydantic
uv add --dev pytest ruff
uv run pytest
~~~

uv 會維護 pyproject.toml 與 uv.lock。建議將這兩個檔案提交 Git，讓不同機器可重現環境。

## 4. 進入虛擬環境（選用）

~~~bash
source .venv/bin/activate
python --version
~~~

也可以不啟動環境，直接使用 uv run：

~~~bash
uv run python main.py
~~~

## 5. GPU／PyTorch／vLLM 特別注意

不要假設 uv add vllm 一定會安裝適合所有 GPU 的組合。vLLM、PyTorch、CUDA runtime、Python 版本有相依性，請先閱讀 vLLM 官方安裝矩陣，並在獨立環境依指示安裝。

官方文件：
- https://docs.astral.sh/uv/
- https://docs.astral.sh/uv/guides/projects/
- https://docs.vllm.ai/en/latest/getting_started/installation/
