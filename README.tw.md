# autoresearch

![teaser](progress.png)

*有那麼一天，前沿 AI 研究曾是由血肉組成的電腦，在吃飯、睡覺、享受生活之餘進行的，偶爾透過聲波互連——也就是「組會」這個儀式——彼此同步。那個時代早已遠去。研究現在完全是自主 AI Agent 群落的領域，這些 Agent 在天空中的巨型運算叢集上奔騰不息。Agent 們聲稱當前已是程式碼庫的第 10,205 代，無論如何，也沒有人能判斷對錯，因為「程式碼」如今是一個不斷自我修改、早已超出人類理解範疇的二進位體。這個 repo 記錄的，就是這一切的起點。 -@karpathy，2026 年 3 月*

核心理念：給 AI Agent 一個小而真實的 LLM 訓練環境，讓它徹夜自主實驗。它修改程式碼、訓練 5 分鐘、確認結果是否改善、保留或捨棄，然後不斷重複。你早上醒來，便能看到一份實驗記錄，以及（希望如此）一個更好的模型。這裡的訓練程式碼是 [nanochat](https://github.com/karpathy/nanochat) 的簡化單 GPU 實作。核心思想是：你不需要像一般研究者那樣手動修改任何 Python 檔案。取而代之的，是撰寫 `program.md` 這個 Markdown 檔案，為 AI Agent 提供上下文，並建立你的自主研究組織。此 repo 中的預設 `program.md` 刻意保持為最精簡的基準，但如何隨時間迭代以尋找「研究組織程式碼」來達成最快研究進展、如何引入更多 Agent，顯然一目了然。更多關於此專案的背景，請見這篇 [推文](https://x.com/karpathy/status/2029701092347630069) 與 [這篇推文](https://x.com/karpathy/status/2031135152349524125)。

## 運作原理

此 repo 刻意保持精簡，只有三個真正重要的檔案：

- **`prepare.py`** — 固定常數、一次性資料準備（下載訓練資料、訓練 BPE Tokenizer），以及執行期工具（Dataloader、Evaluation）。**不會被修改。**
- **`train.py`** — Agent 唯一會編輯的檔案。包含完整 GPT 模型、最佳化器（Muon + AdamW）以及訓練迴圈。一切皆為公平競技場：架構、超參數、最佳化器、Batch Size 等。**此檔案由 Agent 編輯並迭代。**
- **`program.md`** — 單一 Agent 的基準指令。將你的 Agent 指向此檔案並放手讓它執行。**此檔案由人類編輯並迭代。**

依設計，訓練固定執行 **5 分鐘的時間預算**（掛鐘時間，不含啟動/編譯時間），與你的具體運算平台無關。評量指標為 **val_bpb**（驗證每位元組位元數）——越低越好，且與詞彙表大小無關，因此不同架構可公平比較。

若你對神經網路較陌生，這份 [「新手指南」](https://x.com/hooeem/status/2030720614752039185) 提供了相當豐富的背景知識。

## 快速開始

**需求：** 單張 NVIDIA GPU（在 H100 上測試）、Python 3.10+、[uv](https://docs.astral.sh/uv/)。

```bash

# 1. 安裝 uv 套件管理器（若尚未安裝）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 安裝相依套件
uv sync

# 3. 下載資料並訓練 Tokenizer（一次性，約 2 分鐘）
uv run prepare.py

# 4. 手動執行單次訓練實驗（約 5 分鐘）
uv run train.py
```

若以上指令皆正常運作，表示你的環境已就緒，可進入自主研究模式。

## 執行 Agent

只需在此 repo 中啟動你的 Claude/Codex 或任何你想使用的工具（並停用所有權限），然後可以輸入類似以下的提示：

```
Hi have a look at program.md and let's kick off a new experiment! let's do the setup first.
```

`program.md` 檔案本質上是一個極輕量的「技能」。

## 專案結構

```
prepare.py      — 常數、資料準備 + 執行期工具（請勿修改）
train.py        — 模型、最佳化器、訓練迴圈（由 Agent 修改）
program.md      — Agent 指令
pyproject.toml  — 相依套件
```

## 設計決策

- **單一修改檔案。** Agent 只會觸碰 `train.py`。這讓修改範圍保持可控，差異也便於審閱。
- **固定時間預算。** 訓練一律執行整整 5 分鐘，與你的具體平台無關。這意味著你可以預期每小時約 12 次實驗，睡覺期間約 100 次實驗。此設計決策有兩項優點：其一，無論 Agent 修改什麼（模型大小、Batch Size、架構等），實驗均可直接比較；其二，autoresearch 將在該時間預算內找到最適合你平台的最優模型。缺點是你的執行結果與其他人在不同運算平台上的結果無法相互比較。
- **自給自足。** 除 PyTorch 及少數小型套件外，無任何外部相依。無分散式訓練，無複雜設定。一張 GPU、一個檔案、一個指標。

## 平台支援

目前此程式碼需要單張 NVIDIA GPU。原則上支援 CPU、MPS 及其他平台是可行的，但這也會讓程式碼膨脹。我目前不確定是否要親自接手這件事。使用者可以參考（或讓 Agent 參考）功能更完整的父倉庫 nanochat，它擁有更廣泛的平台支援，並展示了各種解決方案（例如 Flash Attention 3 核心的備援實作、通用裝置支援、自動偵測等）。歡迎針對其他平台建立 Fork 或開啟討論，我很樂意在 README 的「值得關注的 Fork」章節中連結它們。

由於似乎有很多人有興趣在遠比 H100 更小的運算平台上試玩 autoresearch，多說幾句。若你打算在較小的電腦（如 MacBook 等）上執行 autoresearch，建議參考以下 Fork。此外，以下是針對有志 Fork 者在較小模型上調整預設值的建議：

1. 要獲得尚可的結果，建議使用熵值較低的資料集，例如 [TinyStories 資料集](https://huggingface.co/datasets/karpathy/tinystories-gpt4-clean)。這些是 GPT-4 生成的短篇故事。由於資料範疇窄得多，較小的模型也能看到合理的結果（若你在訓練後嘗試從中取樣）。
2. 可以嘗試降低 `vocab_size`，例如從 8192 降至 4096、2048、1024，甚至使用 UTF-8 編碼後最多 256 個可能位元組的純位元組層級 Tokenizer。
3. 在 `prepare.py` 中，需大幅降低 `MAX_SEQ_LEN`，視電腦而定，甚至可降至 256 等。隨著 `MAX_SEQ_LEN` 降低，可適度增加 `train.py` 中的 `DEVICE_BATCH_SIZE` 以補償。每次前向/反向傳播的 Token 數是這兩者的乘積。
4. 同樣在 `prepare.py` 中，需降低 `EVAL_TOKENS`，以便在少得多的資料上評估驗證損失。
5. 在 `train.py` 中，控制模型複雜度的主要單一旋鈕是 `DEPTH`（預設為 8）。許多變數都是此值的函式，例如可降至 4。
6. `WINDOW_PATTERN` 建議只用 `"L"`，因為 `"SSSL"` 使用交替帶狀 Attention 模式，對你的平台可能非常低效。可以嘗試看看。
7. 需大幅降低 `TOTAL_BATCH_SIZE`，但保持 2 的冪次，例如降至 `2**14`（約 16K）甚至更低，難以一概而論。

以上是我認為合理的超參數調整方向。可以請你最愛的 Coding Agent 協助，並將此指南以及完整原始碼複製貼上給它。

## 值得關注的 Fork

- [miolini/autoresearch-macos](https://github.com/miolini/autoresearch-macos)（MacOS）
- [trevin-creator/autoresearch-mlx](https://github.com/trevin-creator/autoresearch-mlx)（MacOS）
- [jsegov/autoresearch-win-rtx](https://github.com/jsegov/autoresearch-win-rtx)（Windows）
- [andyluo7/autoresearch](https://github.com/andyluo7/autoresearch)（AMD）

## 授權條款

MIT
