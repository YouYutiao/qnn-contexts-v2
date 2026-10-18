# qnn-contexts-v2

Hiyori 超分 NPU 引擎使用的 **QNN HTP 离线 context** 公共托管仓。仓库与所有 Release 资产均为公开、无鉴权，App 内不带任何凭证。

## 资产命名（必须与 App 侧 `UpscaleModelCatalog.qnnBase()` 完全一致）

```
<family>-x<scale>-<noise>.v<arch>.bin
```

| 家族 | base | 示例 |
|---|---|---|
| Real-CUGAN SE | `realcugan-se-x{2,3,4}-<noise>` | `realcugan-se-x2-denoise1x.v79.bin` |
| Real-CUGAN Pro | `realcugan-pro-x{2,3}-<noise>` | `realcugan-pro-x3-conservative.v81.bin` |

`noise` 取 `no-denoise` / `denoise1x` / `denoise2x` / `denoise3x` / `conservative`（x3/x4 只有 no-denoise / denoise3x / conservative）。
`arch` 取 `69` / `73` / `75` / `79` / `81`，对应 SM8475 / SM8550 / SM8650 / SM8750 / SM8850。

## 生成（QAIRT 2.44.0.260225；禁止复用旧 qnn-binaries-256）

```sh
export QNN_SDK_ROOT=<qairt>/2.44.0.260225
python tools/qnn/export_models.py ...        # 固定 256x256 输入 -> ONNX
python tools/qnn/prepare_calibration.py ...  # NHWC RGB float32 [0,1]
bash  tools/qnn/convert_models.sh            # FP16
bash  tools/qnn/convert_int8_models.sh       # W8A16
```

校准集必须是 **NHWC**：用 NCHW 校准会导致严重偏色，即使转换与建 context 都"成功"。

## 发布

```sh
gh release create v1 --repo YouYutiao/qnn-contexts-v2 --title "QNN contexts v1" dist/*.bin
```

发布后必须同步 App 侧两处：

1. `UpscaleModelCatalog.QNN_RELEASE_BASE_URL` → `https://github.com/YouYutiao/qnn-contexts-v2/releases/download/v1`
2. `UpscaleModelCatalog.SHA256_MANIFEST` 补每个 `.bin` 的 sha256（键为完整文件名，如 `realcugan-se-x2-no-denoise.v79.bin`）。

## 校验

```sh
curl -I https://github.com/YouYutiao/qnn-contexts-v2/releases/download/v1/realcugan-se-x2-no-denoise.v79.bin
```
