# 更新修复日志（CHANGELOG）

> 分支 `director-optimizations-v7`：导演台（MiniMax H3 Director）导出稳定性与随机种子台账系列优化，共 9 个提交。
> 本分支改动**不推上游 PR**（上游 PR 状态见「项目来源」）。

## 项目来源

- 上游项目：**AIMixer/ComfyUI_MiniMaxH3_Director** — https://github.com/AIMixer/ComfyUI_MiniMaxH3_Director
- 本仓库为其 fork：**JiuChenTian/ComfyUI_MiniMaxH3_Director**，用于维护上述优化的可覆盖版本。
- 基点：上游 `main`@`4c6245e`。
- 上游 PR #280（「导出导演包」修复）冻结在 `cdee04b`；此后全部改动仅维护在本地与 fork 分支，**不再推进 PR**。

## 修复与功能日志

| 提交 | 版本 | 内容 |
|---|---|---|
| `31149e2` | 导出修复 | 大时间线导出导演包失败：素材按源去重、ZIP_STORED 免二次压缩、导出移入线程池（不再卡事件循环）、大包改走原生下载流式落盘。实测 202 文件/1.73 GB/4.2 s（修复前 4.05 GB/88.7 s 且失败）。 |
| `f0cb69c` | UI | 素材组批量删除（配合「选择运行」，全选保留第一组）+ 每组最后生成种子展示初版。 |
| `cdee04b` | 种子 v1 | 种子记录改服务端每段落盘：只记完整跑完的段，ComfyUI 重启不丢。（上游 PR #280 止于此） |
| `8e97198` | 种子 v2 | 「最新使用 / 已跑完」双记录；片段身份 recordId（删除/插位不错行）；删除片段联动清行；种子台账随导演包导入导出（包内 `seed_records.json`）。 |
| `2f21a8a` | 种子 v3 | 排队即写「最新使用」（任务跑挂也留痕）；全局模式支持片段提示词覆盖；fl2v 台账按镜头身份记录。 |
| `0eb83f7` | 种子 v4 | 种子徽标常显（无记录显示「种子 —」），补齐三处位置：fl2v「本镜提示词」旁、全局模式「片段提示词（覆盖全局）」旁、段面板「参考图尺寸」左侧。 |
| `ef2d4f4` | 种子 v5 | 修复刷新后徽标“消失”：recordId 跨刷新经 localStorage 复用；服务端孤儿行按 index 归位；全局模式徽标改常显。 |
| `705511d` | 种子 v6 | 修复「导出导演包 → 清空所有分段 → 重新导入」后部分种子记录丢失：身份行 index 全程贯穿（迁移不抹除、导入回写保留、GET 按当前 map 刷新/回填）；批处理与 fl2v 载荷携带 recordId；recordId 跨刷新改内容寻址复用。分段换位时种子**跟随片段**（不消失、不占位替换）。 |
| `d8e728b` | 选择运行 v7 | 「选择运行」勾选跟随分段：拖拽重排（fl2v/批处理/gen/视频四分支）勾选随段迁移、被跨过位次顺移；gen/视频/fl2v/批处理删除腾出的位次不再把勾占位替换给下一段。 |

## 涉及文件（共 12 个）

- 后端 `director\`：`pack.py`、`segment_cache.py`、`executor_core.py`、`fl2v_timeline.py`、`http_routes.py`、`plan.py`
- 前端 `web\js\`：`minimax_timeline.js`、`minimax_fl2v.js`、`minimax_image_batch.js`、`minimax_pack.js`、`minimax_i18n.js`、`minimax_prompt_mentions.js`

## 安装与升级说明

1. 关闭 ComfyUI，将上述 12 个文件覆盖到 `custom_nodes\ComfyUI_MiniMaxH3_Director\` 同名位置。
2. 启动 ComfyUI；浏览器 Ctrl+F5。
3. 种子台账行存于 `output\minimax_seg_cache\<节点缓存key>\`（NovalAI 重定向到 ComfyUI-Shared 时在其下）下的 `rec_<recordId>.seed.json`；旧 `seg_XXXX.seed.json` 首次加载自动迁移，v5 之前缺 `index` 的旧行自动回填（v6）。
4. 建议把工作流 Ctrl+S 保存一次：recordId 写进工作流文件最稳。
5. 「清空分段缓存」不清种子台账；只有删除片段才清对应行。

## 兼容性与边界

- 旧工作流直接打开即可：台账行自动迁移/回填，无需手工处理。
- 复制分组（duplicate）会共享 recordId（两卡片同一台账行，可接受）。
- 外部连线组（i2v/r2v groups）片段无前端 recordId，台账走索引回退（与旧版一致）。
