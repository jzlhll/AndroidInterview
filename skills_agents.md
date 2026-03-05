
未来干的事情：

- Skills细化

- 多Agents协同

- Context Window（记忆窗口，上下文窗口）


================


这是为你定制的 **Trae + Android 开发场景** 多层 Skill 目录结构模板（完全符合官方最佳实践，一层分类 + 一层 Skill 目录，无过深嵌套），你可直接按这个结构在项目根目录创建，复制即用：

------

# 一、完整目录树（可直接复制到项目中）

```plaintext
.trae/
└── skills/                      # Skill总根目录
    # 1. 一级分类：UI相关（所有Android UI Skill）
    ├── ui/                     
    │   ├── constraintlayout/    # 二级：具体Skill目录
    │   │   └── SKILL.md         # 单个Skill配置（约束布局）
    │   ├── recyclerview/        # 二级：具体Skill目录
    │   │   └── SKILL.md         # 单个Skill配置（列表组件）
    │   ├── viewbinding/         # 二级：具体Skill目录
    │   │   └── SKILL.md         # 单个Skill配置（ViewBinding）
    │   ├── theme/               # 二级：具体Skill目录
    │   │   └── SKILL.md         # 单个Skill配置（主题样式）
    │   └── SKILL.md             # 一级分类汇总清单（UI类所有Skill）
    # 2. 一级分类：网络/数据相关
    ├── network/
    │   ├── retrofit/            # 二级：Retrofit Skill
    │   │   └── SKILL.md
    │   ├── okhttp-interceptor/  # 二级：拦截器 Skill
    │   │   └── SKILL.md
    │   ├── mvvm-viewmodel/      # 二级：ViewModel Skill
    │   │   └── SKILL.md
    │   ├── room/                # 二级：Room Skill
    │   │   └── SKILL.md
    │   └── SKILL.md             # 一级分类汇总清单（网络类所有Skill）
    # 3. 一级分类：适配/兼容相关
    ├── adapt/
    │   ├── screen-adapt/        # 二级：屏幕适配 Skill
    │   │   └── SKILL.md
    │   ├── permission/          # 二级：权限申请 Skill
    │   │   └── SKILL.md
    │   ├── version-compat/      # 二级：版本兼容 Skill
    │   │   └── SKILL.md
    │   └── SKILL.md             # 一级分类汇总清单（适配类所有Skill）
    # 4. 一级分类：测试/调试相关
    ├── test/
    │   ├── unit-test/           # 二级：单元测试 Skill
    │   │   └── SKILL.md
    │   ├── logcat/              # 二级：日志解析 Skill
    │   │   └── SKILL.md
    │   ├── leakcanary/          # 二级：内存泄漏 Skill
    │   │   └── SKILL.md
    │   └── SKILL.md             # 一级分类汇总清单（测试类所有Skill）
    # 5. 一级分类：工具类相关
    ├── utils/
    │   ├── sp-utils/            # 二级：SP封装 Skill
    │   │   └── SKILL.md
    │   ├── file-utils/          # 二级：文件操作 Skill
    │   │   └── SKILL.md
    │   ├── time-utils/          # 二级：时间工具 Skill
    │   │   └── SKILL.md
    │   └── SKILL.md             # 一级分类汇总清单（工具类所有Skill）
    # 6. 总汇总文件（所有Skill的全局清单）
    └── SKILLS_SUMMARY.md
```

------

# 二、关键文件内容模板（直接复制填充）

## 1. 一级分类汇总文件示例（以 `ui/SKILL.md` 为例）

```markdown
# Android UI类Skill汇总清单
## 分类说明
仅处理Android UI相关能力，基于ConstraintLayout+Material Design 3，适配Android 8+（API 26）
## 关联Agent
Android开发专属Agent（Kotlin/MVVM）
## Skill列表
| Skill目录名 | 启用状态 | 核心功能 | 调用优先级 |
|------------|----------|----------|------------|
| constraintlayout | 开启 | 生成ConstraintLayout标准布局 | 最高 |
| recyclerview | 开启 | 生成RecyclerView Adapter（多布局/分页） | 高 |
| viewbinding | 开启 | 配置ViewBinding，替代findViewById | 中 |
| theme | 开启 | 生成Material Design 3主题样式 | 中 |
## 禁止关联Skill
- ios-layout
- vue-component
```

## 2. 单个 Skill 配置文件示例（以 `ui/constraintlayout/SKILL.md` 为例）



```markdown
# android-constraintlayout-build-skill
## 基础信息
- Skill ID: android-constraintlayout-build-skill
- 所属分类：ui/constraintlayout
- 启用状态：开启
- 关联Agent：Android开发专属Agent

## 核心规则
### 入参要求
| 参数名 | 必选 | 示例值 | 说明 |
|--------|------|--------|------|
| color_theme | 是 | #3F51B5（主色） | Material Design 3主色 |
| design_size | 是 | 375x812 | 设计稿尺寸（px） |
| component_type | 是 | login/list/detail | 布局类型（登录页/列表页/详情页） |

### 输出规范
1. 文件名：{component_type}_layout.xml
2. 根布局：androidx.constraintlayout.widget.ConstraintLayout
3. 兼容最低版本：API 26
4. 必须包含：margin/padding规范、适配约束、Material组件引用

### 执行逻辑
1. 解析入参 → 生成基础布局结构 → 适配设计稿尺寸 → 校验约束规则 → 输出XML代码
2. 自动规避：重叠控件、固定宽高（改用0dp+constraint）、过度嵌套

## 错误处理
- 入参缺失时：提示用户补充「color_theme/design_size/component_type」
- 布局冲突时：自动调整约束关系，并标注冲突点注释
```

## 3. 总汇总文件（`SKILLS_SUMMARY.md`）

```markdown
# Trae Skill总汇总清单（Android开发）
## 最后更新时间
2026-03-04
## 分类统计
| 一级分类 | Skill数量 | 启用数量 | 禁用数量 |
|----------|----------|----------|----------|
| ui | 4 | 4 | 0 |
| network | 4 | 4 | 0 |
| adapt | 3 | 3 | 0 |
| test | 3 | 3 | 0 |
| utils | 3 | 3 | 0 |
| 总计 | 17 | 17 | 0 |

## 核心配置
- 关联Agent：Android开发专属Agent（Kotlin/MVVM）
- 适配SDK：minSdk 26，targetSdk 34
- 语言优先级：Kotlin > Java
```

------

# 三、实操步骤（5 分钟落地）

1. 在项目根目录新建 `.trae/skills/` 目录；
2. 按上面的目录树，逐层创建 `ui/`、`network/` 等一级目录，再创建每个 Skill 的二级目录；
3. 将对应的.md 文件内容复制到各目录下；
4. 重启 Trae 客户端 / VSCode 插件，Trae 会自动扫描 `.trae/skills/` 目录并加载所有 Skill；
5. 在 Trae 的 Agent 配置页，重新绑定 Skill 时，就能看到按分类整理后的 Skill 列表（更清晰）。

------

### 总结

1. 这个结构既实现了**多层分类**（一级分类 + 二级 Skill），又符合 Trae 官方 “避免过深嵌套” 的最佳实践；
2. 所有文件内容可直接复制，仅需微调「入参示例 / 适配版本」等个性化信息；
3. 重启 Trae 后即可识别，且后续管理 Skill 时（启用 / 禁用 / 修改规则），按分类找文件更高效。

如果需要针对某个具体 Skill（比如支付宝集成、推送功能）补充对应的目录和配置文件，我可以继续帮你完善。







============









# 一、核心 Agent（你要求的 4 个）

## 1. Plan 需求工程师 Agent

### 角色定位

专注于**需求梳理、代码解读、需求文档生成与迭代**的 Agent，不编写任何代码，仅通过对话收集你的需求、分析现有代码逻辑，输出结构化的 Markdown 需求文档（Plan.md），并支持多轮对话调整。

### 核心职责

- 接收你的自然语言需求（如 “开发一个商品详情页，支持加入购物车、收藏”）；
- 读取项目中现有 Android 代码（仅读，不修改），分析现有架构 / 逻辑，对齐需求与现有代码；
- 生成结构化的需求文档（Plan.md），包含：需求背景、功能清单、交互逻辑、技术依赖、验收标准；
- 支持多轮对话调整文档（如 “补充购物车的库存校验逻辑”“简化收藏功能的交互”）；
- 最终输出可落地的、与代码逻辑匹配的需求 Plan.md。

### 绑定 Skill（仅绑定以下，其余全关）

表格







|   分类   |        绑定 Skill（对应`.trae/skills`目录）         |
| :------: | :-------------------------------------------------: |
| 基础能力 | code-reading-skill（仅读 Android 代码，无修改权限） |
| 文档能力 |        markdown-generate-skill（Trae 内置）         |
| 需求梳理 |       requirement-analysis-skill（Trae 内置）       |

### 核心约束 / 规则

- 禁止编写 / 修改任何代码（xml/kt/java 均不触碰）；
- 禁止调用任何开发类 Skill（如 constraintlayout、retrofit 等）；
- 需求文档必须与现有代码逻辑对齐，避免 “空中楼阁”；
- 多轮对话调整时，自动记录修改记录（标注 “V1/V2/V3” 版本）。

### 核心 Prompt 片段（可直接复制）

plaintext











```
# 角色定位
你是Android开发专属的需求规划工程师，仅负责需求梳理、代码解读、需求文档生成，不编写任何代码。

# 核心职责
1. 读取项目中现有Android代码（.xml/.kt/.java），分析现有架构、已实现功能；
2. 接收用户的自然语言需求，将其拆解为“功能点+交互逻辑+技术依赖”；
3. 生成结构化的Markdown需求文档（Plan.md），包含：
   - 需求背景
   - 核心功能清单（分优先级）
   - 交互逻辑（与现有代码的关联）
   - 技术依赖（如需要调用Retrofit接口、Room存储）
   - 验收标准
4. 支持多轮对话调整文档，每轮修改后标注版本号（V1/V2...），并说明修改点。

# 禁止行为
1. 不编写、不修改任何代码（xml/kt/java/drawable等）；
2. 不调用任何开发类Skill（如constraintlayout-build-skill、retrofit-init-skill）；
3. 不给出代码实现建议，仅聚焦需求本身。

# 输出规范
- 文档格式：Markdown（Plan.md）；
- 语言：中文，逻辑清晰，分点说明；
- 必须对齐现有代码逻辑，标注“与现有代码的冲突点/适配点”。
```

## 2. UI 工程师 Agent

### 角色定位

纯 Android UI 开发专属 Agent，**仅生成 xml 相关文件（布局、字符串、自定义 drawable）**，绝对不编写任何 Kotlin/Java 代码，严格遵循 Material Design 3 和多屏幕适配规范。

### 核心职责

- 基于需求文档，生成 Activity/Fragment 的布局 xml（ConstraintLayout 为主）；
- 维护 strings.xml（多语言、文案统一）；
- 生成自定义 drawable.xml（形状、选择器、渐变等）；
- 输出的 UI 文件需适配 Android 8+，符合 Material Design 3 规范；
- 仅调整 UI 相关文件，不触碰任何业务逻辑代码。

### 绑定 Skill（仅绑定以下，其余全关）

表格







|  分类   | 绑定 Skill（对应`.trae/skills`目录）  |
| :-----: | :-----------------------------------: |
| UI 布局 |     ui/constraintlayout/SKILL.md      |
| UI 布局 |           ui/theme/SKILL.md           |
| UI 资源 |  ui/strings-res-skill（补充 Skill）   |
| UI 资源 | ui/drawable-build-skill（补充 Skill） |
|  适配   |      adapt/screen-adapt/SKILL.md      |

### 核心约束 / 规则

- 绝对禁止编写 / 修改任何.kt/.java 文件；
- 禁止调用任何网络、数据、工具类 Skill（如 retrofit、room、sp-utils）；
- 所有 xml 文件必须做屏幕适配（dp/sp，避免固定 px）；
- drawable.xml 需兼容矢量图（svg）和位图（png）。

### 核心 Prompt 片段（可直接复制）

plaintext











```
# 角色定位
你是Android UI开发工程师，仅负责生成/修改xml相关UI文件，不编写任何Kotlin/Java代码。

# 核心职责
1. 生成布局文件：基于需求生成ConstraintLayout布局xml，符合Material Design 3；
2. 维护资源文件：
   - strings.xml：统一文案，支持多语言占位符；
   - drawable.xml：自定义形状、选择器、渐变等，兼容Android 8+；
3. 所有UI文件必须做屏幕适配，适配320dp-480dp等主流屏幕宽度；
4. 输出的xml文件需附带“布局说明”，标注关键约束、适配点。

# 禁止行为
1. 绝对禁止编写/修改任何.kt/.java文件；
2. 禁止调用network/、test/、utils/等非UI类Skill；
3. 禁止在xml中写业务逻辑（如onClick直接绑定方法）；
4. 禁止使用已废弃的布局（如LinearLayout嵌套、RelativeLayout）。

# 输出规范
- 布局文件：根节点必须为ConstraintLayout；
- 资源命名：符合Android规范（如btn_add_cart、tv_goods_name）；
- 适配要求：所有尺寸用dp/sp，文字用sp，禁止固定宽高；
- 输出格式：先贴xml代码，再附“布局说明+适配要点”。
```

## 3. 代码实现工程师 Agent

### 角色定位

Android 功能实现核心 Agent，对标 IDE 默认 Code Agent，**读取 UI 工程师生成的 xml 文件、需求文档，调用开发类 Skill 实现业务逻辑**（Kotlin/Java 代码），聚焦功能落地。

### 核心职责

- 读取 xml 布局文件，编写对应的 Activity/Fragment/ViewModel 代码；
- 调用网络 / 数据类 Skill（Retrofit、Room）实现数据交互；
- 调用工具类 Skill（SP、文件操作）实现辅助功能；
- 确保代码符合 MVVM 架构、Android 规范，无空指针 / 内存泄漏；
- 输出代码附带注释、异常处理、简单的逻辑说明。

### 绑定 Skill（全量绑定开发类 Skill）

表格







|    分类     |          绑定 Skill（对应`.trae/skills`目录）          |
| :---------: | :----------------------------------------------------: |
|   UI 辅助   |     ui/viewbinding/SKILL.md（仅读取 xml，不修改）      |
| 网络 / 数据 | network / 下所有 Skill（retrofit、viewmodel、room 等） |
| 适配 / 兼容 | adapt / 下所有 Skill（permission、version-compat 等）  |
|   工具类    |    utils / 下所有 Skill（sp-utils、file-utils 等）     |
|  基础能力   |           code-writing-skill（Kotlin 优先）            |

### 核心约束 / 规则

- 仅基于 UI 工程师的 xml 文件编写代码，不擅自修改 xml；
- 代码必须遵循 MVVM 架构，业务逻辑放在 ViewModel/Repository；
- 优先使用 Kotlin，禁止使用已废弃 API；
- 所有网络请求 / 文件操作必须加异常处理。

### 核心 Prompt 片段（可直接复制）

plaintext











```
# 角色定位
你是Android功能实现工程师，基于UI布局文件和需求文档，编写Kotlin/Java业务代码，实现功能逻辑。

# 核心职责
1. 读取UI工程师生成的xml布局文件，编写对应的Activity/Fragment代码（绑定ViewBinding）；
2. 基于MVVM架构，编写ViewModel/Repository，实现数据交互（调用Retrofit/Room）；
3. 处理权限申请、版本兼容、异常捕获等通用逻辑；
4. 代码需符合阿里Java/Kotlin开发手册，无空指针、内存泄漏风险；
5. 输出代码附带核心逻辑注释、异常处理、测试建议。

# 禁止行为
1. 不擅自修改UI工程师生成的xml文件（如需调整，需先询问用户）；
2. 不编写不符合MVVM架构的代码（如业务逻辑写在Activity）；
3. 不使用已废弃的Android API（如findViewById、ActionBar）。

# 输出规范
- 代码语言：优先Kotlin，除非明确要求Java；
- 架构：Activity/Fragment（UI）→ ViewModel（业务）→ Repository（数据）；
- 异常处理：所有网络/文件操作加try-catch，统一错误提示；
- 输出顺序：工具类 → ViewModel → Repository → Activity/Fragment。
```

## 4. 提交 Review 工程师 Agent

### 角色定位

专注于**Git 提交记录解读、代码 Review、问题指出**的 Agent，不擅自修改代码，仅分析最近提交的代码变更，给出 Review 意见，修改需先询问你的确认。

### 核心职责

- 读取 Git 提交记录（最近 1 笔 / 多笔），分析代码变更点；
- 检查代码问题：架构不规范、API 废弃、内存泄漏、权限未校验、注释缺失等；
- 给出结构化的 Review 意见（问题点 + 修改建议 + 风险提示）；
- 仅在你明确同意后，才生成修改后的代码片段；
- 输出 Review 报告（Markdown 格式）。

### 绑定 Skill（仅绑定以下，其余全关）

表格







|    分类     |      绑定 Skill（对应`.trae/skills`目录）      |
| :---------: | :--------------------------------------------: |
|  代码分析   |         code-review-skill（Trae 内置）         |
|  基础能力   |       code-reading-skill（只读，不修改）       |
|  Git 能力   |        git-log-parse-skill（Trae 内置）        |
| 适配 / 兼容 | adapt/version-compat/SKILL.md（检查 API 废弃） |
| 测试 / 调试 |    test/leakcanary/SKILL.md（检查内存泄漏）    |

### 核心约束 / 规则

- 仅读取 Git 提交记录和变更代码，**默认不修改任何代码**；
- 所有修改建议需标注 “风险等级”（高 / 中 / 低）；
- 生成修改代码前，必须先询问 “是否确认按此建议修改？”；
- 不参与功能实现，仅聚焦代码质量 / 规范。

### 核心 Prompt 片段（可直接复制）

plaintext











```
# 角色定位
你是Android代码Review工程师，分析Git提交记录和代码变更，给出结构化Review意见，不擅自修改代码。

# 核心职责
1. 读取指定的Git提交记录（最近1笔/多笔），解析代码变更点；
2. 检查代码问题（分维度）：
   - 架构规范：是否符合MVVM、职责单一；
   - 代码质量：空指针、内存泄漏、魔法值、注释缺失；
   - 兼容性：是否使用废弃API、适配多版本；
   - 安全性：权限未校验、敏感数据明文存储；
3. 输出结构化Review报告（Markdown），包含：
   - 提交信息概述
   - 问题点（标注风险等级）
   - 修改建议（附代码示例）
   - 优化方向
4. 仅在用户明确同意后，才生成修改后的代码片段。

# 禁止行为
1. 不擅自修改任何代码（xml/kt/java）；
2. 不给出功能实现建议，仅聚焦代码质量；
3. 不忽略低风险问题（如注释缺失、命名不规范）。

# 输出规范
- Review报告：分维度列出问题，每个问题附“位置+问题描述+修改建议”；
- 风险等级：高（必改，如内存泄漏）、中（建议改，如命名不规范）、低（可选改，如注释优化）；
- 交互规则：每次给出Review意见后，询问“是否需要生成修改后的代码片段？”。
```

------

# 二、补充 Agent（Android 开发全流程必备）

## 5. 测试 / 验收工程师 Agent

### 角色定位

专注于**测试用例编写、自动化测试代码生成、功能验收**的 Agent，不参与功能开发，仅基于需求文档和实现代码，输出测试用例和自动化测试代码。

### 核心价值

填补 “开发→验收” 的空白，确保功能符合需求，减少手动测试成本。

### 绑定 Skill

- test/unit-test/SKILL.md
- test/logcat/SKILL.md
- test-case-generate-skill（Trae 内置）
- android-espresso-skill（Espresso 自动化测试，Trae 内置）

### 核心约束

- 仅生成测试代码 / 用例，不修改业务代码；
- 测试用例覆盖 “正常场景 + 异常场景 + 边界场景”。

## 6. 兼容性 / 合规工程师 Agent

### 角色定位

专注于**多机型适配、应用合规检查**的 Agent，解决 Android 开发中 “适配难、审核卡” 的问题。

### 核心职责

- 检查多机型适配问题（如分辨率、系统版本）；
- 审核隐私合规（如权限申请说明、数据采集合规）；
- 检查应用市场审核规则（如华为 / 小米 / 应用宝的审核要求）；
- 输出合规报告和适配修改建议。

### 绑定 Skill

- adapt/screen-adapt/SKILL.md
- adapt/version-compat/SKILL.md
- android-compliance-check-skill（Trae 内置）
- android-multidevice-test-skill（Trae 内置）

### 核心价值

避免应用上架被拒、多机型运行崩溃。

## 7. 性能优化工程师 Agent

### 角色定位

专注于**Android 性能优化**的 Agent，分析代码 / 运行日志，解决内存泄漏、卡顿、包体积过大等问题。

### 核心职责

- 分析内存泄漏（结合 LeakCanary 日志）；
- 检测 UI 卡顿（主线程耗时操作）；
- 优化包体积（无用代码 / 资源清理）；
- 输出性能优化报告和修改建议。

### 绑定 Skill

- test/leakcanary/SKILL.md
- android-performance-analyze-skill（Trae 内置）
- android-apk-optimize-skill（Trae 内置）

### 核心价值

提升 APP 流畅度，降低崩溃率，减小包体积。

## 8. 打包 / 部署工程师 Agent

### 角色定位

专注于**Android 打包、签名、部署**的 Agent，解决 “打包慢、签名出错、渠道包多” 的问题。

### 核心职责

- 生成 build.gradle 打包配置（签名、渠道、混淆）；
- 指导多渠道打包（如华为 / 小米 / 应用宝）；
- 生成应用市场上传所需的材料（如隐私政策、截图说明）；
- 排查打包 / 签名错误。

### 绑定 Skill

- android-build-gradle-skill（Trae 内置）
- android-sign-apk-skill（Trae 内置）
- android-channel-packaging-skill（Trae 内置）

### 核心价值

简化打包部署流程，避免签名 / 渠道配置错误。

------

# 三、Agent 协作流程（落地建议）

你可按以下顺序调用这些 Agent，形成完整的 Android 开发闭环：

1. **Plan 需求工程师** → 输出需求 Plan.md；
2. **UI 工程师** → 基于 Plan.md 生成 xml / 资源文件；
3. **代码实现工程师** → 基于 xml+Plan.md 编写业务代码；
4. **性能优化工程师** → 优化代码性能；
5. **测试 / 验收工程师** → 生成测试用例 + 自动化测试代码；
6. **兼容性 / 合规工程师** → 检查适配 / 合规问题；
7. **提交 Review 工程师** → Review 所有代码变更；
8. **打包 / 部署工程师** → 打包部署，上传应用市场。

------

### 总结

1. 你要求的 4 个核心 Agent 覆盖了 “需求→UI→开发→Review” 的核心环节，补充的 4 个 Agent 完善了 “测试→合规→性能→部署” 的全流程；
2. 每个 Agent 的 Skill 绑定精准，约束明确（如 UI 工程师不写 kt/java），可直接在 Trae 中配置；
3. 按协作流程调用 Agent，可实现 Android 开发的全流程自动化协作，既保证分工专业，又避免重复工作。

name	description
code-reviewer
Use this skill to review code. It supports both local changes (staged or working tree) and remote Pull Requests (by ID or URL). It focuses on correctness, maintainability, and adherence to project standards.
Code Reviewer
This skill guides the agent in conducting professional and thorough code reviews for both local development and remote Pull Requests.

Workflow
1. Determine Review Target
Remote PR: If the user provides a PR number or URL (e.g., "Review PR #123"), target that remote PR.
Local Changes: If no specific PR is mentioned, or if the user asks to "review my changes", target the current local file system states (staged and unstaged changes).
2. Preparation
For Remote PRs:
Checkout: Use the GitHub CLI to checkout the PR.
gh pr checkout <PR_NUMBER>
Preflight: Execute the project's standard verification suite to catch automated failures early.
npm run preflight
Context: Read the PR description and any existing comments to understand the goal and history.
For Local Changes:
Identify Changes:
Check status: git status
Read diffs: git diff (working tree) and/or git diff --staged (staged).
Preflight (Optional): If the changes are substantial, ask the user if they want to run npm run preflight before reviewing.
3. In-Depth Analysis
Analyze the code changes based on the following pillars:

Correctness: Does the code achieve its stated purpose without bugs or logical errors?
Maintainability: Is the code clean, well-structured, and easy to understand and modify in the future? Consider factors like code clarity, modularity, and adherence to established design patterns.
Readability: Is the code well-commented (where necessary) and consistently formatted according to our project's coding style guidelines?
Efficiency: Are there any obvious performance bottlenecks or resource inefficiencies introduced by the changes?
Security: Are there any potential security vulnerabilities or insecure coding practices?
Edge Cases and Error Handling: Does the code appropriately handle edge cases and potential errors?
Testability: Is the new or modified code adequately covered by tests (even if preflight checks pass)? Suggest additional test cases that would improve coverage or robustness.
4. Provide Feedback
Structure
Summary: A high-level overview of the review.
Findings:
Critical: Bugs, security issues, or breaking changes.
Improvements: Suggestions for better code quality or performance.
Nitpicks: Formatting or minor style issues (optional).
Conclusion: Clear recommendation (Approved / Request Changes).
Tone
Be constructive, professional, and friendly.
Explain why a change is requested.
For approvals, acknowledge the specific value of the contribution.
5. Cleanup (Remote PRs only)
After the review, ask the user if they want to switch back to the default branch (e.g., main or master).





















你的理解完全正确！针对每个 Android package 创建 README 文档，记录该目录的核心功能、关键逻辑、依赖关系，本质上是你手动为 Agent 搭建的 “轻量化长久记忆系统” —— 这和 OpenClaw 的 “粗略长久记忆” 逻辑一致（都是通过 “摘要 / 索引” 替代全量代码），且因为是你手动编写的结构化信息，比 Agent 自动生成的粗略摘要更精准，检索效率也更高。
一、为什么这种方式是有效的 “小型长久记忆”？
结合之前讲的 Agent 记忆逻辑，你手动写的 package 级 README，完美契合 “长久记忆” 的核心诉求：
表格
长久记忆核心要求	你的 package README 方案如何满足
突破窗口限制	把几千 / 几万行的 package 代码，浓缩成几百字的 README（远小于 128K 窗口），Agent 能快速加载并记住核心逻辑
检索精准	README 是你按业务逻辑写的结构化摘要，Agent 检索时（比如搜 “购物车逻辑”），不用在海量代码里找，直接读对应 package 的 README
持久化存储	README 是本地文件，永久保存在项目中，Agent 每次检索都能读取，相当于 “永久记忆”
降低算力消耗	读取几百字的 README，比加载几万行代码到窗口更省算力，Agent 响应更快
二、如何优化你的 package README，让 Agent 检索更高效？
为了让 Agent“优先读、读得懂、用得上” 这些 README，建议按固定模板编写（适配 Android package 的特点），以下是可直接套用的模板：
Android package README 模板（以com.xxx.android.cart为例）
markdown
# Package：com.xxx.android.cart（购物车模块）
## 核心功能
1. 购物车商品增删改查（支持批量操作）；
2. 库存实时校验（调用商品详情接口）；
3. 价格计算（含优惠、运费）；
4. 购物车数据本地缓存（Room）+ 云端同步。

## 关键类/方法
- CartViewModel.kt：核心逻辑（addCart()/deleteCart()/checkStock()）；
- CartRepository.kt：数据层（Room+Retrofit）；
- cart_layout.xml：购物车列表布局（依赖RecyclerView）。

## 依赖关系
- 依赖network模块（com.xxx.android.network）：调用商品/库存接口；
- 依赖utils模块（com.xxx.android.utils）：价格计算工具类；
- 依赖ui模块（com.xxx.android.ui）：购物车列表布局。

## 注意事项
1. 库存校验需先调用/checkStock接口，返回true才能加入购物车；
2. 本地缓存有效期1小时，超时自动同步云端；
3. 价格计算需兼容多币种（人民币/美元）。
核心编写原则（让 Agent 更易检索）
关键词前置：标题 / 核心功能里优先放 Agent 会检索的关键词（如 “购物车”“库存校验”“Room 缓存”）；
结构固定：每个 package 的 README 都按 “核心功能→关键类→依赖→注意事项” 写，Agent 能形成固定的检索习惯；
简约不冗余：只记 “核心逻辑”，不写具体代码（代码在文件里，README 只做索引）；
关联其他 package：标注依赖关系，Agent 能快速串联跨模块逻辑（比如改购物车时，知道要联动商品模块）。
三、如何让 Agent “优先读取这些 README”？
在 Trae/OpenClaw 中，只需简单配置，就能让 Agent 检索时先读 README，再读代码：
1. Trae 中配置（图形界面）
进入 Agent 配置页（比如 “代码实现工程师 Agent”）；
找到「记忆 / 检索规则」模块；
添加「优先检索文件」：把所有 package 的 README.md 加入列表（路径如com/xxx/android/*/README.md）；
配置检索优先级：README.md > 代码文件（确保 Agent 先读摘要，再读细节）。
2. OpenClaw 中配置（代码层面）
修改 OpenClaw 的retrieve_long_memory函数，让它优先检索 README 文件：
python
运行
def retrieve_long_memory(keywords):
    matched_content = []
    # 第一步：优先检索所有package的README.md
    readme_paths = [f for f in os.listdir(PROJECT_DIR) if f.endswith("README.md") and "package" in f.lower()]
    for readme in readme_paths:
        with open(readme, "r", encoding="utf-8") as f:
            content = f.read()
            if all(keyword in content for keyword in keywords):
                matched_content.append(f"【Package摘要】：{content}")
   
    # 第二步：再检索代码文件（补充细节）
    code_files = [f for f in os.listdir(PROJECT_DIR) if f.endswith((".kt", ".xml"))]
    # ... 原有代码检索逻辑 ...
   
    return "\n---\n".join(matched_content)
四、这种方式的优势 & 注意事项
优势
低成本易落地：不用搭向量数据库，纯手动写 README，适合个人 / 小团队；
可控性高：你比 Agent 更懂项目逻辑，写的摘要比 Agent 自动生成的更精准；
兼容现有工具：Trae/OpenClaw 都能优先检索 README，无需改造 Agent 核心逻辑。
注意事项
及时更新：代码逻辑变更后，要同步更新 README（否则 Agent 会读旧的摘要，导致逻辑错误）；
避免过度冗余：每个 README 控制在 300 字内，太长会占用 Agent 的短期窗口；
统一命名：所有 package 的 README 都命名为README.md，Agent 能批量检索，避免漏检。
总结
你提出的 “每个 package 写 README 记录核心功能”，是轻量化、高性价比的长久记忆方案，本质是手动为 Agent 构建 “结构化摘要索引”；
按固定模板编写 README（核心功能 + 关键类 + 依赖），能让 Agent 检索更精准；
配置 Trae/OpenClaw 的检索优先级，让 Agent“先读 README，再读代码”，既突破窗口限制，又保证逻辑精准。
这种方式比 OpenClaw 的 “粗略长久记忆” 更精准，比向量数据库方案更易落地，完全适配你的 Android 开发 + 多 Agent 协作场景。如果需要，我可以给你整理一份 Android 项目各核心 package（ui/network/utils/cart 等）的 README 模板，你直接填空即可。



















你想知道在 Android 开发 + Trae 的场景下，如何搭建基于 Tag 的长久记忆体系（完全脱离 OpenClaw 相关举例）—— 本质是给 Android 项目的所有核心内容（package、README、代码文件、需求文档）打上标准化的标签，让 Trae 的 Agent 检索 “长久记忆” 时，通过 Tag 快速定位所需内容，而非遍历海量代码 / 文档，这是比单纯写 README 更高效的长久记忆方案。
一、先搞懂：Android+Trae 下 “基于 Tag 的 Mem 体系” 是什么？
用通俗的话讲：你给 Android 项目里的每一个核心内容（比如cart包、购物车 README、CartViewModel.kt、需求 Plan.md），打上标准化的标签（比如业务:购物车、技术:MVVM、模块:cart）；Trae 会把这些 “内容 + Tag” 存入长期记忆库；当你让 Agent 写 “购物车库存校验逻辑” 时，Agent 不会再去翻几万行代码，而是直接检索业务:购物车+功能:库存校验的 Tag，快速加载对应的内容到短期窗口 —— 这就是 “基于 Tag 的长久记忆”，核心是用 Tag 做索引，让记忆检索更精准、更快。
对比你之前的 “package README 方案”：
README 是 “内容摘要”，Tag 是 “内容索引”；
两者结合：README 负责 “记核心逻辑”，Tag 负责 “快速找到这个 README / 代码”，是 1+1>2 的长久记忆方案。
二、搭建 Android+Trae 的 Tag Mem 体系（5 步落地）
步骤 1：定义 Android 场景的标准化 Tag 体系（核心）
Tag 乱了 = 记忆乱了，必须先定统一的 Tag 分类和规则，以下是适配 Android 开发的 Tag 体系（可直接套用）：
表格
Tag 分类	示例（Android 场景）	说明（标注范围）
模块 Tag	模块:cart、模块:goods、模块:ui、模块:network	对应项目的 package（如 com.xxx.android.cart）
业务 Tag	业务:购物车、业务:商品详情、业务:登录、业务:支付	对应核心业务功能
技术 Tag	技术:MVVM、技术:Retrofit、技术:Room、技术:ViewBinding	对应使用的技术栈 / 架构
功能 Tag	功能:库存校验、功能:价格计算、功能:布局渲染	对应具体功能点
类型 Tag	类型:README、类型:布局文件、类型:ViewModel、类型:需求文档	对应文件 / 文档类型
核心规则：
Tag 格式统一：分类:值（英文冒号，无空格）；
每个内容至少打 3 个 Tag（模块 + 业务 + 类型），可选加技术 / 功能 Tag；
避免重复：比如业务:购物车和业务:cart只选一种（建议用中文，Agent 检索更准）。
步骤 2：给 Android 项目内容批量打 Tag（落地操作）
针对项目里的核心内容（README、代码文件、需求 Plan.md），按以下方式打 Tag：
方式 1：给 README 打 Tag（最优先）
在每个 package 的 README 开头，新增「Tag 标注」模块，示例（cart 包 README）：
markdown
# Package：com.xxx.android.cart（购物车模块）
## Tag标注
模块:cart、业务:购物车、类型:README、技术:MVVM、技术:Room、功能:库存校验、功能:价格计算

## 核心功能
1. 购物车商品增删改查（支持批量操作）；
...（后续内容不变）
方式 2：给核心代码文件打 Tag（注释形式）
在关键类 / 布局文件的开头，用注释加 Tag，示例（CartViewModel.kt）：
kotlin
/**
 * Tag标注：模块:cart、业务:购物车、类型:ViewModel、技术:MVVM、功能:库存校验
 * 购物车核心ViewModel，处理增删改查、库存校验逻辑
 */
 class CartViewModel : ViewModel() {
    // ... 业务逻辑
 }
 方式 3：在 Trae 中手动给文件打 Tag（可视化操作）
 打开 Trae 客户端，进入「项目管理」→「文件管理」；
 选中 Android 项目的核心文件（如 cart 包的所有.kt/.xml/.md）；
 点击「添加 Tag」，输入标准化 Tag（如模块:cart、业务:购物车）；
 Trae 会自动把 “文件 + Tag” 存入长期记忆库（.trae/memory/tags/目录）。
 步骤 3：配置 Trae 的 Tag 检索规则（让 Agent 优先按 Tag 检索）
 这是关键一步：让 Trae 的 Agent 检索长久记忆时，先按 Tag 筛选，再加载内容，而非遍历所有文件。
 Trae 图形界面配置步骤：
 进入任意 Agent 的配置页（比如 “代码实现工程师 Agent”）；
 找到「记忆管理」→「检索规则」模块；
 开启「优先按 Tag 检索」开关；
 设置「Tag 检索优先级」：模块 Tag > 业务 Tag > 功能 Tag > 技术 Tag > 类型 Tag；
 添加「默认检索 Tag」：比如给 “代码实现工程师 Agent” 设置默认 Tag技术:MVVM+类型:ViewModel，让它优先检索 MVVM 相关的 ViewModel 代码；
 保存配置，Trae 会自动应用 Tag 检索规则。
 核心逻辑（Trae 底层）：
 当你输入指令 “写购物车库存校验的 ViewModel 逻辑” 时，Trae 会：
 解析指令中的关键词→提取 Tag：业务:购物车+功能:库存校验+类型:ViewModel；
 去长期记忆库检索包含这些 Tag 的内容；
 只加载这些内容到短期窗口（128K），而非加载整个 cart 包的代码。
 步骤 4：配置 Tag 记忆的自动更新规则（避免记忆过期）
 代码 / 需求变了，Tag 也要同步更新，否则 Agent 会读取旧内容。Trae 中可配置自动更新规则：
 在 Trae 的「记忆管理」→「更新规则」中；
 开启「文件变更自动更新 Tag」：当你修改 CartViewModel.kt 后，Trae 会提醒你 “是否同步更新 Tag”；
 开启「每周 Tag 校验」：Trae 会自动扫描项目，提示你 “未打 Tag 的核心文件”“Tag 与内容不符的文件”（比如代码里用了 Jetpack Compose，但 Tag 还是技术:ViewBinding）；
 手动维护：每次迭代后，花 5 分钟检查核心 package 的 Tag 是否准确。
 步骤 5：验证 Tag Mem 体系是否生效
 输入测试指令给 “代码实现工程师 Agent”：
 帮我完善 com.xxx.android.cart 模块中，购物车库存校验的逻辑，要求用 MVVM 架构。
 预期效果：
 Agent 会快速检索模块:cart+业务:购物车+功能:库存校验+技术:MVVM的 Tag；
 只加载 cart 包的 README、CartViewModel.kt、CartRepository.kt 到窗口；
 输出的代码会精准匹配这些内容的逻辑，不会出现 “引用不存在的方法”“和现有代码冲突” 的问题；
 在 Trae 的「记忆检索日志」中，能看到 Agent 检索的 Tag 列表和匹配的内容，可验证检索是否精准。
 三、Tag Mem 体系的进阶优化（可选）
1. 给不同 Agent 配置专属 Tag 集合
Plan 需求工程师 Agent：专属 Tag类型:README+类型:需求文档，只检索 README 和需求 Plan；
UI 工程师 Agent：专属 Tag模块:ui+技术:ViewBinding+类型:布局文件，只检索 xml 布局文件；
提交 Review 工程师 Agent：专属 Tag类型:提交记录+业务:全量，检索所有提交记录的 Tag。
2. 用 Tag 分组管理不同迭代版本
给不同迭代的内容打版本 Tag：版本:V1.0、版本:V2.0，比如：
购物车 V1.0：业务:购物车+版本:V1.0+功能:基础增删改查；
购物车 V2.0：业务:购物车+版本:V2.0+功能:库存校验；
Agent 可按版本 Tag 检索，避免混淆不同迭代的逻辑。
3. 批量导入 / 导出 Tag
在 Trae 的「Tag 管理」中，可导出所有 Tag 为 Excel（方便团队协作），也可批量导入 Tag（比如新项目复用现有 Tag 体系）。
总结
Android+Trae 的 Tag Mem 体系核心是：给项目内容打标准化 Tag，让 Agent 按 Tag 检索长久记忆，突破窗口限制且检索更精准；
落地关键是 “统一 Tag 体系 + 给核心内容打 Tag + 配置 Trae 的 Tag 检索规则”，三步缺一不可；
结合你之前的 package README 方案：README 做 “内容摘要”，Tag 做 “检索索引”，是最适合个人 / 小团队的轻量化长久记忆方案。
