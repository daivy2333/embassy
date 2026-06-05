## Purpose

记录项目开发过程中学到的知识，避免重复探索，加速问题解决。

## Requirements

### Requirement: API 路径记录

项目中使用的关键 API 路径 SHALL 记录，包含用途和使用示例。

#### Scenario: 发现新 API

- **WHEN** 开发者发现或使用了新的 API 端点
- **THEN** 必须记录到 learned/spec.md，包含：API 路径、用途、请求/响应格式

### Requirement: 踩坑经验记录

遇到的技术陷阱和解决方案 MUST 记录，防止重复踩坑。

#### Scenario: 解决棘手问题

- **WHEN** 开发者花费大量时间解决了一个技术问题
- **THEN** 必须记录踩坑档案，包含：症状、根因、解决方案、预防措施

### Requirement: 技巧模式记录

有效的开发技巧和模式 SHALL 记录，促进知识共享。

#### Scenario: 发现高效做法

- **WHEN** 开发者发现了一种高效的开发技巧或模式
- **THEN** 必须记录到技巧模式区，包含：技巧名称、适用场景、使用方法

### Requirement: 文件速查表

关键文件和目录的位置 SHALL 记录，加速代码导航。

#### Scenario: 定位关键文件

- **WHEN** 开发者频繁访问某些文件或目录
- **THEN** 必须记录到文件速查表，包含：文件路径、用途、关键内容

---

## API 路径速查

### embassy-executor

| API | 用途 | 示例 |
|-----|------|------|
| `Spawner::spawn()` | 异步任务派发 | `spawner.spawn(task_name(arg)).unwrap()` |
| `#[embassy_executor::task]` | 任务宏 | 标记异步函数为 embassy 任务 |
| `#[embassy_executor::main]` | 主函数宏 | 标记入口点 |

### embassy-time

| API | 用途 | 示例 |
|-----|------|------|
| `Timer::after_millis()` | 延时等待 | `Timer::after_millis(100).await` |
| `Instant::now()` | 获取当前时间 | `let now = Instant::now()` |
| `Duration::from_millis()` | 创建时长 | `let d = Duration::from_millis(50)` |

### embassy-nrf (nRF 平台)

| API | 用途 | 示例 |
|-----|------|------|
| `Output::new()` | GPIO 输出 | `Output::new(pin, Level::Low, OutputDrive::Standard)` |
| `Input::new()` | GPIO 输入 | `Input::new(pin, Pull::Up)` |
| `Uarte::new()` | UART 初始化 | 见 examples/nrf52840 |

---

## 踩坑档案

### MkDocs `custom_dir: null` 触发 TypeError(2026-06-05)

**症状**:GitHub Actions 跑 `mkdocs build --strict` 报错:

```
File "mkdocs/config/config_options.py", line 844, in run_validation
    if 'custom_dir' in theme_config and not os.path.isabs(theme_config['custom_dir']):
TypeError: expected str, bytes or os.PathLike object, not NoneType
```

**根因**:`mkdocs.yml` 写了 `theme.custom_dir: null`(本想"显式声明不用自定义模板")。
MkDocs validator 用 `if 'custom_dir' in theme_config` 判断 key 是否存在 — `null` 也算"存在",然后 `os.path.isabs(None)` 直接炸。

**解决**:**删除** `custom_dir: null` 这一行。MkDocs 默认就没这个 key,根本不需要"显式 null"。

**预防**:
- 不要在 `mkdocs.yml` 里写"显式 null"占位(`key: null` / `key: ~` / `key:` 留空均不可)
- "我不用这个特性"= 不写,**不是**写 null
- 这与 Python `dict.get(k, None)` 的容忍语义不同,YAML schema validator 严格区分"key 存在但值是 None" vs "key 不存在"

**适用范围**:同样的雷区可能存在于其它 MkDocs config options(只要 validator 写了 `if 'X' in cfg:` 后直接对值操作)。配置项若官方文档说"optional",意思是"可省略",不是"可写 null"。

---

## 技巧模式

### MkDocs Material 中文技术文档站(2026-06-05)

**适用场景**:中文 + 大量代码片段 + Mermaid 图 + 表格的技术文档,部署到 GitHub Pages。

**核心配置要点**:

| 维度 | 配置 |
|------|------|
| 主题 | `theme.name: material` + `language: zh` |
| Mermaid 渲染 | `pymdownx.superfences` + `custom_fences: name=mermaid` 一行搞定(Material 9.5+ 内建 Mermaid.js) |
| 中文搜索 | `plugins.search.lang: [zh, en]`(无需 jieba 插件,Material 内建 Lunr 中文分词) |
| 暗色跟随系统 | `palette` 两段配置 + `media: "(prefers-color-scheme: ...)"` |
| 字体 | `theme.font.text: Noto Sans SC`(中文) + `code: JetBrains Mono` |
| 代码复制按钮 | `features: content.code.copy` 一行 |
| 排除文件 | `not_in_nav: \|\n  /README.md`(避免 README.md 和 index.md 双首页冲突) |

**GitHub Pages 部署模式选择(★ 易踩坑)**:

```
错误:Settings → Pages → Source = "Deploy from a branch" + main/docs
  → GitHub 用默认 Jekyll,只渲染 README,看不到导航

正确:Settings → Pages → Source = "GitHub Actions"
  → 让 .github/workflows/docs.yml 控制构建产物,部署到 gh-pages 环境
```

**Workflow 关键设置**:
- `permissions: pages: write + id-token: write`(部署 Pages 必需)
- `concurrency.group: pages` + `cancel-in-progress: false`(避免并发部署冲突)
- `mkdocs build --strict`(broken link 直接失败,质量门)
- `fetch-depth: 0`(若用 git-revision-date 插件需要完整 history)

**预防项**:
1. `.gitignore` 必须加 `site/`(MkDocs build 输出,不要 commit)
2. `Settings → Actions → General → Workflow permissions = Read and write`(否则部署 403)
3. 本地 `python3 -c "yaml.unsafe_load(open('mkdocs.yml'))"` 会因为 `!!python/name:` tag 报错,这是预期(需先装 mkdocs-material),CI 里没问题

**参考来源**:本项目 `mkdocs.yml` + `.github/workflows/docs.yml`(2026-06-05 搭建)。

---

## 文件速查表

| 路径 | 用途 |
|------|------|
| `embassy-executor/src/` | 异步执行器核心 |
| `embassy-time/src/` | 时间管理 |
| `embassy-nrf/src/` | nRF 平台 HAL |
| `embassy-stm32/src/` | STM32 平台 HAL |
| `embassy-rp/src/` | RP2040/235x 平台 HAL |
| `embassy-net/src/` | 网络栈 |
| `embassy-usb/src/` | USB 栈 |
| `examples/` | 示例代码 |
| `tests/` | 集成测试 |
