# ComfyUI-SillyDream-GPT-Image-2 — GPT Image 2 / 2.5 文生图 / 图生图

通过任意 OpenAI 兼容的云端网关调用 GPT Image 2 与 GPT Image 2.5（Flare / Sunburst）完成文生图 / 图生图。支持最多 16 张参考图、官方自定义尺寸、透明背景、PNG/JPEG/WebP 输出与压缩，并保留 GPT Image 2 旧工作流兼容。

如果感兴趣也可以试试注册我的官网：https://wish.sillydream.top

## 特性

- **文生图 + 图生图**：最多 16 张参考图，支持 `/v1/images/generations`、`/v1/images/edits`、`/v1/responses` 三种接口，也可设为 `auto` 自动选择
- **GPT Image 2.5 双模型**：`gpt-image-2.5-flare` 速度优先，`gpt-image-2.5-sunburst` 编辑精度优先
- **2.5 原生输出控制**：透明/不透明背景、PNG/JPEG/WebP、JPEG/WebP 压缩和 `xhigh` / `max` 质量档
- **官方自定义尺寸**：`custom_size` 支持 16 像素倍数、最长边 3840、比例 1:3 至 3:1 的 `WIDTHxHEIGHT`
- **分辨率/比例互斥**：内置权威尺寸表（见下），非法的「比例 × 分辨率」组合会在发请求前直接拦截，避免浪费一次可能 5-15 分钟又 502 的请求；配套的前端脚本让下拉框直接只列出合法档位
- **不做内部重试**：网络异常/5xx 不自动重试（重试等于重复扣费），失败会抛出带完整诊断信息（server / cf-ray / x-request-id / body）的报错，方便判断故障出在 Cloudflare / nginx / 中转网关 / OpenAI 官方哪一层
- **兼容非标准返回格式**：除了标准 `data[].b64_json` / `data[].url`，也能从部分中转网关「伪装成聊天回复」的文本中兜底抠出 Markdown 图床链接或裸图片 URL
- **非敏感配置持久化**：`base_url` / 默认模型 / 默认分辨率等会自动记住，`api_key` **永远不落盘**

## 安装

**方式一：ComfyUI-Manager（推荐）**

Manager → **Install via Git URL**，填入：

```
https://github.com/qianchi7/ComfyUI-SillyDream-GPT-Image-2.git
```

装完点 Manager 里的 **Restart**（不要只点 Reload）。以后作者更新代码，在 Manager 里点该节点的 **Update** 按钮即可（本质是 `git pull`）。

**方式二：手动 clone**

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/qianchi7/ComfyUI-SillyDream-GPT-Image-2.git
pip install requests Pillow numpy
```

重启 ComfyUI 后，在节点搜索框里搜 `GPT Image 2 Generator`（分类 `GPT Image 2`）。

## 示例工作流

装好后顶部菜单 **工作流 → 浏览模板 (Browse Templates)**，可选：

- `GPT-Image-2` — 完整工作流
- `GPT-Image-2-Text2Img` — 最简纯文生图
- `GPT-Image-2-Reference` — 最简带参考图

无需手动拖 json，工作流会随 `git pull` 一起更新。（`example_workflows/` 目录里的 json 也可以直接拖进网页界面。）

## 节点参数

| 参数 | 说明 |
|------|------|
| `model` | `gpt-image-2`、`gpt-image-2.5-flare` 或 `gpt-image-2.5-sunburst`；Flare 偏速度，Sunburst 偏编辑精度 |
| `resolution` / `aspect_ratio` | 预设分辨率与比例；选 `resolution=custom` 时改填 `custom_size` |
| `custom_size` | 2.5 自定义 `WIDTHxHEIGHT`，宽高均为 16 的倍数，最长边≤3840，比例 1:3 至 3:1 |
| `quality` | GPT Image 2：`auto` / `low` / `medium` / `high`；GPT Image 2.5 另支持 `xhigh` / `max` |
| `background` | 2.5：`auto` / `opaque` / `transparent`；透明背景配合 PNG 或 WebP |
| `output_format` | 2.5：`auto` / `png` / `jpeg` / `webp` |
| `output_compression` | JPEG/WebP 压缩等级 0-100，PNG 忽略 |
| `moderation` | 2.5：`auto` / `low` |
| `n` | 生成数量，1~10 |
| `image_1`~`image_16` | 最多 16 张参考图输入 |
| `response_format` | GPT Image 2 兼容旧中转；2.5 官方始终返回 base64，节点自动省略该字段 |
| `edit_mode` | `generate` / `reference` / `outpaint`，仅用于本地选择接口 |
| `timeout` / `infinite_timeout` | 请求总超时；长耗时生成建议开启无限超时 |
| `api_endpoint` | `auto`、`/v1/images/generations`、`/v1/images/edits` 或 `/v1/responses` |

## 权威尺寸表（GPT Image 2 预设档）
|---|---|---|---|
| 1:1 | 1024x1024 | 2048x2048 | — |
| 3:2 | 1536x1024 | 2048x1360 | 3520x2352 |
| 2:3 | 1024x1536 | 1360x2048 | 2352x3520 |
| 16:9 | — | 2048x1152 | 3840x2160 |
| 9:16 | — | 1152x2048 | 2160x3840 |
| 4:3 | — | 2048x1536 | 3312x2480 |
| 3:4 | — | 1536x2048 | 2480x3312 |
| 21:9 | — | 2688x1152 | 3840x1648 |
| 1:3 | — | 1024x3072 | 1280x3840 |
| 3:1 | — | — | 3840x1280 |

「—」表示该比例在该分辨率档下没有合法尺寸（互斥）：例如 `1:1` 选不出 `4K`，`16:9` 选不出 `1K`，`3:1` 只能选 `4K`。这些尺寸经过实测，是不易触发 502 的固定档位；节点只发 `model/prompt/size/n/quality(/seed)` 等 GPT-Image-2 认可的字段，不发 `enhance_prompt/negative_prompt/style_preset` 等不支持的参数。

## 节点参数

| 参数 | 说明 |
|------|------|
| `api_key` | 你自己网关对应的 API Key |
| `base_url` | 你自己的 OpenAI 兼容网关地址，只需填 `http://IP:端口`（默认预填的是作者本人的演示网关，建议替换成你自己的） |
| `model` | 模型名称，需与你所用网关中配置的模型名一致 |
| `resolution` / `aspect_ratio` | 分辨率档位与宽高比，二者互斥（见上表）；均为 `auto` 时交给上游自选，`aspect_ratio=auto` 时会从 `image_1` 推断比例 |
| `quality` | `auto` / `low` / `medium` / `high` |
| `n` | 生成数量，1~10 |
| `seed` | 随机种子，-1 为自动随机 |
| `response_format` | `auto` 时会显式请求 `b64_json`（避免依赖下载图床图片），也可强制选 `url` |
| `edit_mode` | `generate`（纯生成）/ `reference`（参考图生图）/ `outpaint`（扩图），仅用于本地选择接口 |
| `timeout` / `infinite_timeout` | 请求总超时；4K/High 常需 5-15 分钟，建议保持较长超时或开启无限超时 |
| `api_endpoint` | `auto` 会根据是否有参考图自动选择 `/v1/images/generations` 或 `/v1/images/edits`，也可强制指定，包括 `/v1/responses` |
| `image_1`~`image_16` | 最多 16 张参考图输入 |

## 配置文件位置（自动写入，不含 API Key）

- Windows: `%APPDATA%\gpt_image_2\config.json`
- Linux/macOS: `~/.config/gpt_image_2/config.json`
- 回退: `<节点目录>/.gpt_image_2_config.json`

保存字段：`base_url`、`default_model`、`default_size`、`default_quality`、`default_endpoint`、`default_output_format`、`disable_proxy`。**不保存** `api_key`，每次都要在节点里重新填写。

## 故障排除

**❌ 装好了但搜不到节点**
- 必须**重启** ComfyUI（不是刷新网页），custom_nodes 只在启动时扫描一次
- 看终端日志有没有 `Traceback` / `ImportError` / `ModuleNotFoundError`，多半是缺依赖
- 便携版装依赖要用 `python_embeded\python.exe -m pip install requests Pillow numpy`

节点失败时会抛出带完整上下文的报错（HTTP 状态码、`server`/`cf-ray`/`x-request-id`/响应体等），可据此判断故障出在哪一层：

- **Cloudflare（`cf-ray` 头）**：多为中转反代问题
- **nginx（`server: nginx/x.y.z`）**：自建反代问题，检查 `proxy_read_timeout` 是否够长（建议 ≥1800s，因为 4K/High 生成常需 5-15 分钟）
- **new-api / one-api（`server: Go`）**：上游中继问题，检查对应渠道是否存活、token 是否有效
- **OpenAI 官方（`server: OpenAI`）**：真实上游返回的错误

**❌ 无法连接到服务器 / Connection error**
- 确认 `base_url` 能从本机浏览器/curl 正常访问

**❌ HTTP 502/503**
- 把 `resolution` 从 4K 降到 2K，`quality` 从 high 降到 medium 先验证链路
- 如果是自建反代/中转网关，检查其读超时配置

**❌ HTTP 400**
- 提示 `Unknown parameter` 时节点会自动去掉该字段重试一次；若持续失败，检查 `model` 名称是否与网关配置一致

**❌ 接口未返回任何图像**
- 节点已兼容「图片链接被伪装成聊天文本」的返回格式，若仍失败，把报错里附带的完整响应体拿去核对网关实际返回结构

## 协议

[MIT License](./LICENSE)
