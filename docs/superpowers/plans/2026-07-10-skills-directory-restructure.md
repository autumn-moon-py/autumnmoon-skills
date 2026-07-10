# Skills 目录重构计划

> **面向 AI 代理的工作者：** 必需子技能：使用 using-superpowers:subagent-driven-development（推荐）或 using-superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 将 `skills/skills/` 中的 skill 全部提升到根目录，同时将 `using-dev`、`using-flutter`、`using-superpowers` 的子 skill 改为 `plugins/` 架构（参考 `getx-state-management-patterns`），消除根目录与 `using-*` 之间的重复 skill 目录。

**架构：** 根目录只保留 `using-*` 入口 skill（含 `plugins/` 子目录）和独立 skill。`using-dev/plugins/ios-swift-dev/`、`using-flutter/plugins/flutter-app-dev/`、`using-superpowers/plugins/brainstorming/` 等存放子 skill。根目录下与 `using-*` 子 skill 同名的独立目录将被删除（因为内容已迁入 `plugins/`）。

**技术栈：** 文件系统操作（移动、删除目录），SKILL.md 路径引用更新

---

## 当前问题

1. **`skills/skills/` 旧版嵌套目录**：包含 29 个 SKILL.md，大部分与根目录重复
2. **根目录与 `using-*` 子目录重复**：如根目录有 `brainstorming/`，`using-superpowers/` 下也有 `brainstorming/`，内容相同
3. **`using-*` 子 skill 平铺存放**：`using-superpowers/brainstorming/SKILL.md`，应改为 `using-superpowers/plugins/brainstorming/SKILL.md`

## 目标结构

```
skills/                                    # 根目录
├── using-dev/                             # 入口 skill
│   ├── SKILL.md                           # 索引（更新路径引用）
│   └── plugins/
│       ├── ios-swift-dev/                 # 从 using-dev/ios-swift-dev/ 迁入
│       │   ├── SKILL.md
│       │   ├── agents/
│       │   └── references/
│       └── ohos-arkts-dev/                # 从 using-dev/ohos-arkts-dev/ 迁入
│           ├── SKILL.md
│           └── references/
│
├── using-flutter/                         # 入口 skill
│   ├── SKILL.md                           # 索引（更新路径引用）
│   └── plugins/
│       ├── flutter-app-dev/               # 从 using-flutter/flutter-app-dev/ 迁入
│       │   ├── SKILL.md
│       │   ├── agents/
│       │   └── references/
│       ├── flutter-ios-bridge-dev/         # 从 using-flutter/flutter-ios-bridge-dev/ 迁入
│       │   ├── SKILL.md
│       │   ├── agents/
│       │   └── references/
│       └── ohos-flutter-dev/              # 从 using-flutter/ohos-flutter-dev/ 迁入
│           ├── SKILL.md
│           └── references/
│
├── using-superpowers/                     # 入口 skill
│   ├── SKILL.md                           # 索引（更新路径引用）
│   ├── references/                        # 保留（工具参考文档）
│   └── plugins/
│       ├── brainstorming/                 # 从根目录 brainstorming/ 迁入
│       │   ├── SKILL.md
│       │   ├── scripts/
│       │   └── ...
│       ├── chinese-code-review/           # 从根目录迁入
│       ├── chinese-documentation/         # 从根目录迁入
│       ├── dispatching-parallel-agents/   # 从根目录迁入
│       ├── executing-plans/               # 从根目录迁入
│       ├── finishing-a-development-branch/ # 从根目录迁入
│       ├── karpathy-guidelines/           # 从根目录迁入
│       ├── receiving-code-review/         # 从根目录迁入
│       ├── requesting-code-review/        # 从根目录迁入
│       ├── subagent-driven-development/   # 从根目录迁入
│       ├── systematic-debugging/          # 从根目录迁入
│       ├── verification-before-completion/ # 从根目录迁入
│       ├── workflow-runner/               # 从根目录迁入
│       └── writing-plans/                 # 从根目录迁入
│
├── brandkit/                              # 独立 skill（保持不变）
├── design-taste-frontend/                 # 独立 skill（保持不变）
├── ...其他独立 skill...
├── getx-state-management-patterns/        # 独立 skill（保持不变）
├── humanizer-zh/                          # 独立 skill（保持不变）
├── pdf-converter/                         # 独立 skill（保持不变）
├── sequential-thinking/                   # 独立 skill（保持不变）
├── writing-skills/                         # 独立 skill（保持不变）
└── skills/                                # 删除整个目录
```

---

## 文件变更清单

### 移动操作（using-dev）

| 源 | 目标 |
|----|------|
| `using-dev/ios-swift-dev/` | `using-dev/plugins/ios-swift-dev/` |
| `using-dev/ohos-arkts-dev/` | `using-dev/plugins/ohos-arkts-dev/` |

### 移动操作（using-flutter）

| 源 | 目标 |
|----|------|
| `using-flutter/flutter-app-dev/` | `using-flutter/plugins/flutter-app-dev/` |
| `using-flutter/flutter-ios-bridge-dev/` | `using-flutter/plugins/flutter-ios-bridge-dev/` |
| `using-flutter/ohos-flutter-dev/` | `using-flutter/plugins/ohos-flutter-dev/` |

### 移动操作（using-superpowers）— 从根目录独立目录迁入

| 源 | 目标 |
|----|------|
| `brainstorming/` | `using-superpowers/plugins/brainstorming/` |
| `chinese-code-review/` | `using-superpowers/plugins/chinese-code-review/` |
| `chinese-documentation/` | `using-superpowers/plugins/chinese-documentation/` |
| `dispatching-parallel-agents/` | `using-superpowers/plugins/dispatching-parallel-agents/` |
| `executing-plans/` | `using-superpowers/plugins/executing-plans/` |
| `finishing-a-development-branch/` | `using-superpowers/plugins/finishing-a-development-branch/` |
| `karpathy-guidelines/` | `using-superpowers/plugins/karpathy-guidelines/` |
| `receiving-code-review/` | `using-superpowers/plugins/receiving-code-review/` |
| `requesting-code-review/` | `using-superpowers/plugins/requesting-code-review/` |
| `subagent-driven-development/` | `using-superpowers/plugins/subagent-driven-development/` |
| `systematic-debugging/` | `using-superpowers/plugins/systematic-debugging/` |
| `verification-before-completion/` | `using-superpowers/plugins/verification-before-completion/` |
| `workflow-runner/` | `using-superpowers/plugins/workflow-runner/` |
| `writing-plans/` | `using-superpowers/plugins/writing-plans/` |

### 删除操作

| 目标 | 原因 |
|------|------|
| `using-superpowers/brainstorming/` | 已迁入 plugins/，与根目录 brainstorming/ 内容相同 |
| `using-superpowers/chinese-code-review/` | 同上 |
| `using-superpowers/chinese-documentation/` | 同上 |
| `using-superpowers/dispatching-parallel-agents/` | 同上 |
| `using-superpowers/executing-plans/` | 同上 |
| `using-superpowers/finishing-a-development-branch/` | 同上 |
| `using-superpowers/karpathy-guidelines/` | 同上 |
| `using-superpowers/receiving-code-review/` | 同上 |
| `using-superpowers/requesting-code-review/` | 同上 |
| `using-superpowers/subagent-driven-development/` | 同上 |
| `using-superpowers/systematic-debugging/` | 同上 |
| `using-superpowers/verification-before-completion/` | 同上 |
| `using-superpowers/workflow-runner/` | 同上 |
| `using-superpowers/writing-plans/` | 同上 |
| `skills/` | 整个旧版嵌套目录删除 |

### 修改操作

| 文件 | 变更内容 |
|------|----------|
| `using-dev/SKILL.md` | 更新子 skill 路径引用：`ios-swift-dev` → `plugins/ios-swift-dev` |
| `using-flutter/SKILL.md` | 更新子 skill 路径引用 |
| `using-superpowers/SKILL.md` | 更新子 skill 路径引用：`brainstorming` → `plugins/brainstorming` 等 |

---

## 任务

### 任务 1：创建 plugins/ 目录结构

**文件：**
- 创建：`using-dev/plugins/`
- 创建：`using-flutter/plugins/`
- 创建：`using-superpowers/plugins/`

- [ ] **步骤 1：创建三个 plugins 目录**

```powershell
New-Item -ItemType Directory -Path "C:\Users\Administrator\.skills-manager\skills\using-dev\plugins" -Force
New-Item -ItemType Directory -Path "C:\Users\Administrator\.skills-manager\skills\using-flutter\plugins" -Force
New-Item -ItemType Directory -Path "C:\Users\Administrator\.skills-manager\skills\using-superpowers\plugins" -Force
```

- [ ] **步骤 2：验证目录已创建**

```powershell
Test-Path "C:\Users\Administrator\.skills-manager\skills\using-dev\plugins"
Test-Path "C:\Users\Administrator\.skills-manager\skills\using-flutter\plugins"
Test-Path "C:\Users\Administrator\.skills-manager\skills\using-superpowers\plugins"
```

预期：三个都返回 True

---

### 任务 2：迁移 using-dev 子 skill 到 plugins/

**文件：**
- 移动：`using-dev/ios-swift-dev/` → `using-dev/plugins/ios-swift-dev/`
- 移动：`using-dev/ohos-arkts-dev/` → `using-dev/plugins/ohos-arkts-dev/`

- [ ] **步骤 1：移动 ios-swift-dev**

```powershell
Move-Item -Path "C:\Users\Administrator\.skills-manager\skills\using-dev\ios-swift-dev" -Destination "C:\Users\Administrator\.skills-manager\skills\using-dev\plugins\ios-swift-dev"
```

- [ ] **步骤 2：移动 ohos-arkts-dev**

```powershell
Move-Item -Path "C:\Users\Administrator\.skills-manager\skills\using-dev\ohos-arkts-dev" -Destination "C:\Users\Administrator\.skills-manager\skills\using-dev\plugins\ohos-arkts-dev"
```

- [ ] **步骤 3：验证迁移结果**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-dev" -Recurse -Name
```

预期：只看到 `SKILL.md`、`plugins\ios-swift-dev\...`、`plugins\ohos-arkts-dev\...`

---

### 任务 3：迁移 using-flutter 子 skill 到 plugins/

**文件：**
- 移动：`using-flutter/flutter-app-dev/` → `using-flutter/plugins/flutter-app-dev/`
- 移动：`using-flutter/flutter-ios-bridge-dev/` → `using-flutter/plugins/flutter-ios-bridge-dev/`
- 移动：`using-flutter/ohos-flutter-dev/` → `using-flutter/plugins/ohos-flutter-dev/`

- [ ] **步骤 1：移动 flutter-app-dev**

```powershell
Move-Item -Path "C:\Users\Administrator\.skills-manager\skills\using-flutter\flutter-app-dev" -Destination "C:\Users\Administrator\.skills-manager\skills\using-flutter\plugins\flutter-app-dev"
```

- [ ] **步骤 2：移动 flutter-ios-bridge-dev**

```powershell
Move-Item -Path "C:\Users\Administrator\.skills-manager\skills\using-flutter\flutter-ios-bridge-dev" -Destination "C:\Users\Administrator\.skills-manager\skills\using-flutter\plugins\flutter-ios-bridge-dev"
```

- [ ] **步骤 3：移动 ohos-flutter-dev**

```powershell
Move-Item -Path "C:\Users\Administrator\.skills-manager\skills\using-flutter\ohos-flutter-dev" -Destination "C:\Users\Administrator\.skills-manager\skills\using-flutter\plugins\ohos-flutter-dev"
```

- [ ] **步骤 4：验证迁移结果**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-flutter" -Recurse -Name
```

预期：只看到 `SKILL.md`、`plugins\flutter-app-dev\...`、`plugins\flutter-ios-bridge-dev\...`、`plugins\ohos-flutter-dev\...`

---

### 任务 4：迁移 using-superpowers 子 skill 到 plugins/（从根目录独立目录迁入）

**文件：**
- 移动：根目录 `brainstorming/` → `using-superpowers/plugins/brainstorming/`
- 移动：根目录 `chinese-code-review/` → `using-superpowers/plugins/chinese-code-review/`
- 移动：根目录 `chinese-documentation/` → `using-superpowers/plugins/chinese-documentation/`
- 移动：根目录 `dispatching-parallel-agents/` → `using-superpowers/plugins/dispatching-parallel-agents/`
- 移动：根目录 `executing-plans/` → `using-superpowers/plugins/executing-plans/`
- 移动：根目录 `finishing-a-development-branch/` → `using-superpowers/plugins/finishing-a-development-branch/`
- 移动：根目录 `karpathy-guidelines/` → `using-superpowers/plugins/karpathy-guidelines/`
- 移动：根目录 `receiving-code-review/` → `using-superpowers/plugins/receiving-code-review/`
- 移动：根目录 `requesting-code-review/` → `using-superpowers/plugins/requesting-code-review/`
- 移动：根目录 `subagent-driven-development/` → `using-superpowers/plugins/subagent-driven-development/`
- 移动：根目录 `systematic-debugging/` → `using-superpowers/plugins/systematic-debugging/`
- 移动：根目录 `verification-before-completion/` → `using-superpowers/plugins/verification-before-completion/`
- 移动：根目录 `workflow-runner/` → `using-superpowers/plugins/workflow-runner/`
- 移动：根目录 `writing-plans/` → `using-superpowers/plugins/writing-plans/`

**注意：** 根目录下的这些独立目录与 `using-superpowers/` 下的同名子目录内容相同。移动根目录版本到 `plugins/` 后，`using-superpowers/` 下的旧平铺版本将在任务 5 中删除。

- [ ] **步骤 1：批量移动 14 个 skill 目录**

```powershell
$base = "C:\Users\Administrator\.skills-manager\skills"
$pluginsDir = "$base\using-superpowers\plugins"
$skills = @(
  'brainstorming',
  'chinese-code-review',
  'chinese-documentation',
  'dispatching-parallel-agents',
  'executing-plans',
  'finishing-a-development-branch',
  'karpathy-guidelines',
  'receiving-code-review',
  'requesting-code-review',
  'subagent-driven-development',
  'systematic-debugging',
  'verification-before-completion',
  'workflow-runner',
  'writing-plans'
)

foreach ($skill in $skills) {
  $src = "$base\$skill"
  $dst = "$pluginsDir\$skill"
  if (Test-Path $src) {
    Move-Item -Path $src -Destination $dst -Force
    Write-Output "Moved: $skill -> using-superpowers/plugins/$skill"
  } else {
    Write-Output "SKIP (not found): $skill"
  }
}
```

- [ ] **步骤 2：验证迁移结果**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-superpowers\plugins" -Directory | ForEach-Object { $_.Name }
```

预期：14 个目录全部出现

---

### 任务 5：删除 using-superpowers 下的旧平铺子 skill 目录

**文件：**
- 删除：`using-superpowers/brainstorming/`（已迁入 plugins/，且根目录版本也已迁入 plugins/）
- 删除：`using-superpowers/chinese-code-review/`
- 删除：`using-superpowers/chinese-documentation/`
- 删除：`using-superpowers/dispatching-parallel-agents/`
- 删除：`using-superpowers/executing-plans/`
- 删除：`using-superpowers/finishing-a-development-branch/`
- 删除：`using-superpowers/karpathy-guidelines/`
- 删除：`using-superpowers/receiving-code-review/`
- 删除：`using-superpowers/requesting-code-review/`
- 删除：`using-superpowers/subagent-driven-development/`
- 删除：`using-superpowers/systematic-debugging/`
- 删除：`using-superpowers/verification-before-completion/`
- 删除：`using-superpowers/workflow-runner/`
- 删除：`using-superpowers/writing-plans/`

**注意：** 这些旧目录与 `plugins/` 下的新目录内容相同，是重复的。删除前确认 `plugins/` 下已有对应目录。

- [ ] **步骤 1：批量删除旧平铺子 skill**

```powershell
$base = "C:\Users\Administrator\.skills-manager\skills\using-superpowers"
$skills = @(
  'brainstorming',
  'chinese-code-review',
  'chinese-documentation',
  'dispatching-parallel-agents',
  'executing-plans',
  'finishing-a-development-branch',
  'karpathy-guidelines',
  'receiving-code-review',
  'requesting-code-review',
  'subagent-driven-development',
  'systematic-debugging',
  'verification-before-completion',
  'workflow-runner',
  'writing-plans'
)

foreach ($skill in $skills) {
  $path = "$base\$skill"
  $pluginPath = "$base\plugins\$skill"
  # 安全检查：确认 plugins/ 下已有对应目录才删除旧目录
  if ((Test-Path $path) -and (Test-Path $pluginPath)) {
    Remove-Item -Path $path -Recurse -Force
    Write-Output "Deleted: using-superpowers/$skill (plugins/$skill exists)"
  } elseif (Test-Path $path) {
    Write-Output "SKIP DELETE: using-superpowers/$skill exists but plugins/$skill NOT found - safety check"
  } else {
    Write-Output "SKIP: using-superpowers/$skill already gone"
  }
}
```

- [ ] **步骤 2：验证 using-superpowers 目录结构**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-superpowers" -Name
```

预期：只看到 `SKILL.md`、`references/`、`plugins/`

---

### 任务 6：删除 skills/skills/ 旧版嵌套目录

**文件：**
- 删除：`skills/` 整个目录

**注意：** `skills/skills/` 中的 skill 要么已存在于根目录（如 `humanizer-zh/`、`pdf-converter/`、`sequential-thinking/`、`writing-skills/`），要么是 `using-*` 的旧版分组（`dev/`、`flutter/`、`superpowers/`），这些内容在根目录都有对应。`skills/skills/using-dev/`、`skills/skills/using-flutter/`、`skills/skills/using-superpowers/` 只有 SKILL.md 入口文件，子 skill 已在根目录的 `using-*` 中。

- [ ] **步骤 1：删除整个 skills/skills/ 目录**

```powershell
Remove-Item -Path "C:\Users\Administrator\.skills-manager\skills\skills" -Recurse -Force
Write-Output "Deleted: skills/skills/ directory"
```

- [ ] **步骤 2：验证删除**

```powershell
Test-Path "C:\Users\Administrator\.skills-manager\skills\skills"
```

预期：False

---

### 任务 7：更新 using-dev/SKILL.md 路径引用

**文件：**
- 修改：`using-dev/SKILL.md`

当前 SKILL.md 中引用子 skill 的路径需要更新，将 `ios-swift-dev` 改为 `plugins/ios-swift-dev`，`ohos-arkts-dev` 改为 `plugins/ohos-arkts-dev`。

- [ ] **步骤 1：读取当前 SKILL.md 内容**

确认需要修改的路径引用位置。

- [ ] **步骤 2：更新路径引用**

将 SKILL.md 中所有对子 skill 的路径引用从平铺路径更新为 `plugins/` 路径。具体修改点需根据实际内容确定，主要涉及：
- 技能索引部分的路径描述
- 任何 `using-dev:ios-swift-dev` 形式的引用改为 `using-dev:plugins/ios-swift-dev` 或保持 skill name 不变（取决于 skill 加载机制）

- [ ] **步骤 3：验证修改**

读取修改后的 SKILL.md，确认所有路径引用正确。

---

### 任务 8：更新 using-flutter/SKILL.md 路径引用

**文件：**
- 修改：`using-flutter/SKILL.md`

同任务 7，更新子 skill 路径引用。

- [ ] **步骤 1：读取当前 SKILL.md 内容**

- [ ] **步骤 2：更新路径引用**

将 `flutter-app-dev` → `plugins/flutter-app-dev`，`flutter-ios-bridge-dev` → `plugins/flutter-ios-bridge-dev`，`ohos-flutter-dev` → `plugins/ohos-flutter-dev`

- [ ] **步骤 3：验证修改**

---

### 任务 9：更新 using-superpowers/SKILL.md 路径引用

**文件：**
- 修改：`using-superpowers/SKILL.md`

同任务 7，更新 14 个子 skill 的路径引用。

- [ ] **步骤 1：读取当前 SKILL.md 内容**

- [ ] **步骤 2：更新路径引用**

将所有 `brainstorming` → `plugins/brainstorming`，`chinese-code-review` → `plugins/chinese-code-review` 等。

- [ ] **步骤 3：验证修改**

---

### 任务 10：最终验证

- [ ] **步骤 1：验证根目录结构**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills" -Directory | Where-Object { $_.Name -notin @('.git','.skills-manager') } | ForEach-Object { $_.Name }
```

预期：不再有 `brainstorming/`、`karpathy-guidelines/` 等独立目录（已迁入 `using-superpowers/plugins/`），不再有 `skills/` 嵌套目录。

- [ ] **步骤 2：验证 using-dev/plugins/ 结构**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-dev\plugins" -Directory | ForEach-Object { $_.Name }
```

预期：`ios-swift-dev`、`ohos-arkts-dev`

- [ ] **步骤 3：验证 using-flutter/plugins/ 结构**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-flutter\plugins" -Directory | ForEach-Object { $_.Name }
```

预期：`flutter-app-dev`、`flutter-ios-bridge-dev`、`ohos-flutter-dev`

- [ ] **步骤 4：验证 using-superpowers/plugins/ 结构**

```powershell
Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills\using-superpowers\plugins" -Directory | ForEach-Object { $_.Name }
```

预期：14 个 skill 目录

- [ ] **步骤 5：验证 SKILL.md 数量**

```powershell
# 根目录（排除 .git, .skills-manager）
$rootSkills = Get-ChildItem -Path "C:\Users\Administrator\.skills-manager\skills" -Filter "SKILL.md" -Recurse | Where-Object { $_.FullName -notmatch '\.git|\.skills-manager' }
Write-Output "Total SKILL.md count: $($rootSkills.Count)"
```

预期：与重构前相同（138 个），没有丢失

- [ ] **步骤 6：Commit**

```bash
git add -A
git commit -m "refactor: restructure skills directory - move sub-skills into plugins/ architecture

- using-dev: ios-swift-dev, ohos-arkts-dev -> plugins/
- using-flutter: flutter-app-dev, flutter-ios-bridge-dev, ohos-flutter-dev -> plugins/
- using-superpowers: 14 workflow skills -> plugins/
- Remove duplicate root-level skill directories (moved into using-superpowers/plugins/)
- Remove legacy skills/skills/ nested directory
- Update SKILL.md path references in using-* entry skills"
```
