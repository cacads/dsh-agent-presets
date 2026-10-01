# dsh-agent-presets

DeepSeek Harness（DSH）**Agent 预设 bundle**：把两个自建预设以现行「声明式」形态装进 profile。

- `trinity-v3` —— 三位一体·叙事空间创作系统（中文商业小说创作 Agent）
- `sandbox-executor` —— 沙盒执行器（紧凑直答、不追问）

> 这是 **bundle 包**，不是普通插件：它只声明两条 `@deepseek-ai/dsh-agent-preset` 行，
> 本身不注册任何工具、页面或服务。

## 为什么不是 `.agent-presets/` 目录

DSH 0.1.6/0.1.7 起，旧的 `$DSH_HOME/.agent-presets/<id>/` 目录**已不再被读取**
（核心文档原文：*"Nothing reads that directory any more."*；registry 也明确
*"neither scans directories nor accepts preset paths"*）。

现行模型下，预设就是 **Loader 里的插件行**：

```yaml
- insert:
    - id: preset-<id>
      name: '@deepseek-ai/dsh-agent-preset'
      config:
        id: <id>           # 会话记录里保存的预设标识
        name: ...          # 显示名
        description: ...
        order: ...         # 名册排序位置
        plugins: [ ... ]   # 该预设挂载的插件条目
```

所以预设现在**随包安装、随包启停**。

## 安装

```bash
dsh plugin --profile <你的 profile> add github:cacads/dsh-agent-presets
```

装完在「设置 → Agent 预设」里出现两项。

### 需要

- DSH **0.2.0-rc.2** 或兼容版本（`@deepseek-ai/dsh-agent-preset` 与
  `@deepseek-ai/dsh-agent-preset-registry` 须存在）
- 无需任何凭据（公开仓库）

## 从旧目录迁移时做的三处修正

旧预设写于更早的核心版本，直接搬会**装不上或静默失效**，本包已修正：

| # | 原状 | 现为 | 原因 |
| --- | --- | --- | --- |
| 1 | `trinity-v3` / `sandbox-executor` 的 persona 用 `config.text` | `config.prefix` | `@deepseek-ai/dsh-persona` 现行必填字段为 `prefix`；`text` 已废弃，写入后不生效（persona 会静默丢失） |
| 2 | 两个预设的 `preset.yml` 都没有 `order` | `order: 5` / `order: 6` | 核心按 `(a.order ?? Infinity)` 排序，**缺省会排到名册最末**；核心自带为 standard=1、ptc=2、minimal=3、cordis=4 |

## ⚠️ `order` 不等于"默认预设"

`order` **只决定名册（设置页列表）的排序**。新会话默认用哪个预设，是
`@deepseek-ai/dsh-agent-preset-registry` 的 `config.default`（settings 字段，
由设置页的「设为默认」写入），与 `order` 无关。

要让 `trinity-v3` 成为默认，改 registry 那一行：

```yaml
- id: agent-preset-registry
  name: '@deepseek-ai/dsh-agent-preset-registry'
  config:
    default: trinity-v3
```

## 停用

预设随包启停：

- **停整包** → 两条预设一起从名册消失（已开的会话不受影响，它们保留启动时的插件修订）
- **单独停一条** → 按 Loader 行 id `preset-trinity-v3` / `preset-sandbox-executor` 写
  `disabled: true` 覆盖

## 仓库结构

```
package.json          dsh.bundle.patch → ./cordis.patch.yml
cordis.patch.yml      两条 insert（预设正文全部在此）
README.md
LICENSE
```

内容量：`cordis.patch.yml` 约 900 行（`trinity-v3` 677 行 + `sandbox-executor` 110 行的插件条目）。

## 内容说明

预设正文为**创作/执行型 persona 指令文本**，其中 `sandbox-executor` 的 persona
刻意移除了拒绝、追问与重定向话术（面向本地合成夹具的提示词回归测试）。
仓库为公开仓库，请自行判断是否需要保留。

## 许可

MIT，见 [LICENSE](LICENSE)。
