# TritonServer

本專案用於快速啟動與管理 NVIDIA Triton Inference Server（以下簡稱 Triton），透過容器化與設定檔範本，協助部署 GPU 推論服務。此專案將作為 PPE V2 專案中模型服務的基礎元件。

## 專案在做什麼

- 提供基本的專案骨架與範例設定（例如 `docker-compose.yml`）。
- 說明如何選擇正確的 Triton 與 CUDA 組合版本。
- 提供模型倉庫（Model Repository）的標準目錄結構說明。
- 作為未來擴充（模型 Repository、健康檢查、監控）的起點。

## 使用前置需求

- 已安裝 Docker 與 NVIDIA Container Toolkit（需具備 NVIDIA 顯示卡與驅動）。
- 依據下方對應表選擇合適的 Triton/CUDA 版本標籤。

## 模型倉庫結構

TritonServer 需要遵循特定的目錄結構來組織模型檔案：

```text
model-repository/
├── <model-name>/                    # 模型名稱目錄
│   ├── config.pbtxt                 # 模型設定檔（建議提供）
│   └── <version>/                   # 版本目錄（通常是數字，如 1）
│       └── <model-file>             # 實際的模型檔案
```

### 模型檔案命名規則

TritonServer 會根據模型類型自動尋找預設的檔案名稱：

| 模型類型 | 預設檔案名稱 | 範例 |
|---------|-------------|------|
| TensorFlow SavedModel | `model.savedmodel` | `model.savedmodel/` |
| ONNX | `model.onnx` | `model.onnx` |
| TensorRT | `model.plan` | `model.plan` |
| PyTorch | `model.pt` | `model.pt` |

**重要：** 如果您的模型檔案名稱與預設名稱不同（例如：`best320.onnx`、`yolov8n.onnx`），則需要在 `config.pbtxt` 中指定 `default_model_filename`：

```protobuf
name: "yolov8n"
backend: "onnxruntime"
default_model_filename: "best320.onnx"  # 指定實際的檔案名稱
```

如果模型檔案名稱符合預設規則（如 `model.onnx`），則**不需要**在 `config.pbtxt` 中設定 `default_model_filename`。

### config.pbtxt 設定檔說明

雖然 TritonServer 可以自動推斷某些模型類型（如 ONNX、TensorFlow SavedModel）的基本配置，但**強烈建議**提供 `config.pbtxt` 檔案，原因如下：

- **明確指定模型名稱**：確保模型名稱與目錄名稱一致
- **自訂配置參數**：設定批次大小、輸入輸出張量詳細資訊等
- **指定後端平台**：明確指定使用 ONNX Runtime、TensorRT 等後端
- **效能優化**：設定動態批次處理、實例群組等進階參數

**注意**：對於某些模型類型或複雜配置需求，`config.pbtxt` 是**必需的**。

## 快速開始

1. 依據環境選擇對應的 NGC Triton 容器標籤（例如：`nvcr.io/nvidia/tritonserver:<release>-py3`）。
2. 更新 `docker-compose.yml` 中的鏡像標籤與掛載路徑（模型倉庫、設定）。
3. 將您的模型檔案放置在正確的目錄結構中。
4. 啟動服務：

```bash
docker compose up -d
```

## Triton 與 CUDA 對應版本（參考）

NGC 發佈的 Triton 容器以年月版號標示（如 `24.08`），每個版本綁定對應的 CUDA 版本。以下列出常見近年組合，實際以 NGC 容器標籤與發行說明為準。

| Triton 容器版本 | 大致對應 CUDA 版本 | 備註 |
| --- | --- | --- |
| 23.10 – 24.01 | 12.2 | Ampere/後期 Turing 良好支援 |
| 24.02 – 24.05 | 12.3 | 部分 PyTorch/TensorRT 需對應套件 |
| 24.06 – 24.09 | 12.4 | Hopper 支援改善，常見於 2024 下半年 |
| 24.10 – 25.02 | 12.5/12.6 | 依發行月略有差異，請以標籤為準 |

查詢方式：

- 造訪 NGC Triton 容器頁面查看該版對應 CUDA 與驅動需求（搜尋 `NGC Triton Inference Server`）。
- 參考 Triton Release Notes 與容器標籤描述（如 `tritonserver:24.09-py3`）。

> 提醒：CUDA 與顯示卡驅動需相容，驅動版本通常需不低於 CUDA runtime 所需的最低版本。

## 授權（License）

本專案採用 Apache License 2.0 授權。詳細條款請見專案根目錄之 `LICENSE` 檔案。

## 參考連結

- NGC Triton Inference Server（查詢容器版本與說明）
- Triton Inference Server Release Notes
