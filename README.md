# image-to-layered-psd

![License: MIT](https://img.shields.io/badge/license-MIT-green)

把一张扁平的 JPG / PNG / WebP 图片，重建为 **可编辑的分层 PSD**（主体、道具、文字、双层阴影各自成层，背景独立）。

适用于"这张图给我拆成分层 PSD""图片转分层 PSD""拆分图层""从平面图恢复源文件"这类需求；不做普通修图，也不做电商主图重设计。

## 它做了什么

不是把原图切成若干块贴到图层里，而是按 **原位生成 → 全画布透明分件 → 校色去污染 → 原生 PSD** 的流程重建：

1. **清点**：建立 `manifest.json`，逐项记录文字、道具、产品、配件、背景、光效与遮挡关系。
2. **合批**：无像素交叠、无互相遮挡的元素合成一批生成；被遮挡元素单独补全，用 `scripts/plan.py` 校验清单覆盖与冲突。
3. **原位生成**：保持原画幅、坐标、比例、透视与光照，只做原位编辑，不重排素材、不紧裁居中。
4. **透明与分件**：原生 alpha 优先，遇假透明切安全纯底色再用 `scripts/asset_ops.py` 做透明转换与无损分件（RGB 棋盘格不算透明）。
5. **校色去残色**：按材质内部颜色比对，9 点 RGB 偏移 + 智能模糊 + 局部色域修正，浅底/深底/实际背景下都验细边与半透明。
6. **光影合成**：近窄深 + 远宽浅双层阴影，与承接面同组；文字用原生文字层，规则图形用形状层。
7. **验证交付**：`scripts/verify_psd.py` 结构校验 + Photoshop 重开、编辑文字、移动整组、纯背景目视检查，并从最终 PSD 导出预览 PNG。

## 依赖

- Python 3：`Pillow`、`numpy`、`opencv-python`；PSD 结构验证另需 `psd-tools`
- Photoshop（本机 COM + UTF-8 JSX，Windows）——用于建层、嵌入对象、智能滤镜与验证
- 一个可用的图像生成能力：主体与道具是**重新生成**的，不是从原背景抠出来的

缺其中任一项时，skill 会说明具体缺口和已完成的部分，不会拿扁平 PSD 冒充分层成功。

## 安装

WorkBuddy（用户级，全局可用）：

```bash
git clone https://github.com/opun12/image-to-layered-psd.git \
  ~/.workbuddy/skills/image-to-layered-psd
```

Windows 默认路径为 `C:\Users\<用户名>\.workbuddy\skills\image-to-layered-psd`。放好后重启客户端，在「技能」设置页确认已加载。

其它兼容 `SKILL.md` 约定的 agent 运行时：把整个目录放到其 skills 目录即可，入口文件是 `SKILL.md`。

## 目录结构

```
SKILL.md              入口：质量底线、执行路线与检查点
agents/openai.yaml    界面展示名、简介与默认提示词
references/
  workflow.md         七阶段工作流与每阶段判定标准
  prompts.md          原位编辑提示词写法
  tooling.md          manifest / samples / regions 参数格式与命令行
  pitfalls.md         常见故障与排查顺序
  v3-lessons.md       V3/V3.1 的根因复盘与断点复用
scripts/
  plan.py             校验 manifest 清单覆盖与批间冲突
  asset_ops.py        choose / inspect / matte / split 四个子命令
  build_psd.ps1       预检后调用 Photoshop 建文档
  compose.jsx         嵌入对象、文字、形状、智能滤镜
  preflight.py        运行环境与依赖预检
  verify_psd.py       PSD 结构与图层校验
  self_test.py        自检
  native_test.py      原生图层能力测试
```

## 已知边界

- 不承诺与原图逐像素一致；看不清的文字、品牌细节与偏差会被记录，不编造。
- 原图长边 ≤ 4096 时保持原尺寸，超过才等比缩到 ≤ 4096（计入 EXIF 方向）。
- 原图只作为隐藏参考层存在，不能靠它掩盖缺失内容。

## License

[MIT](LICENSE) © 2026 opun12

可自由使用、修改、分发与商用，保留版权声明即可。基于本 skill 生成的图片、PSD 等产出物的版权归使用者自己所有，不受本许可约束。
