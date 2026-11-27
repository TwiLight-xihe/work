# 个人健康管理 Agent —— README

> 赛道 A（AI 原生）· HarmonyOS 7（API 26）· ArkTS + ArkUI · Stage 模型

面向需要记录运动、饮食和睡眠的学生，提供**非医疗性质**的生活方式摘要与建议。核心闭环：**记录数据 → 云端 AI 分析/建议 → 保存与复盘**，同时支持多轮智能问答（带会话记忆）与拍照食物识别。

---

## 1. 项目简介

- 三类记录（运动/饮食/睡眠）的增删改查与本地持久化；
- 基于当日记录，调用云端大语言模型生成非医疗健康摘要/建议；
- 多轮智能问答，带会话记忆（可新建/切换/删除会话）；
- 拍照/选图/样例，调用云端视觉模型识别食物并支持「保存为饮食记录」；
- 未配置密钥或调用失败时自动本地降级，保证演示闭环。

## 2. 开发环境与 SDK 版本

| 项目 | 版本 |
| --- | --- |
| 操作系统 | HarmonyOS 7（API 26） |
| 开发工具 | DevEco Studio 26.0.0.x |
| 工程模型 | Stage 模型（`apiType: stageMode`） |
| 语言/UI | ArkTS + ArkUI（声明式） |
| hvigor modelVersion | `26.0.0` |
| targetSdkVersion | `26.0.0` |
| compatibleSdkVersion | `6.1.1(24)` |
| 包名 | `com.example.h20261123` |

## 3. 目录结构

```
entry/src/main/ets/
├── entryability/EntryAbility.ets   # 应用入口，注入全局 Context
├── pages/                          # 视图层：Index / RecordPage / AiAdvicePage / FoodCameraPage
├── service/                        # 服务层：HealthDataService / HealthRepository / StorageService / HealthAiService / AiAdviceService
├── model/                          # 模型层：HealthModels / AiModels / PageParams
└── common/                         # 通用层：Config / GlobalContext / DataEvents / Logger / Constants / DateUtil / AsyncUtil / ImageUtil
```

## 4. 构建、安装与运行步骤

1. 用 DevEco Studio 26.0.0.x 打开工程根目录，等待 Sync。
2. 若提示签名，按向导自动生成签名。
3. 选择 Phone 模拟器，点击 Run。
4. 启动后进入首页仪表盘。

> 修改 `module.json5`（权限）或 `common/Config.ets`（AI 配置）等编译型文件后，需重新编译（建议「卸载 → 重新 Run」）。

## 5. 测试设备 / 模拟器信息

- 设备类型：Phone 模拟器（DevEco Studio 内置）。
- 权限：`ohos.permission.INTERNET`。
- 相册测试：把本地图片拖入模拟器窗口导入系统相册，再通过「从相册选择图片」读取。

## 6. 默认数据 / 测试账号说明

- **无需登录、无测试账号**：应用为本地单机，数据存 Preferences。
- 首次启动预置 3 条演示记录（`HealthRepository.ensureSeeded()`）：
  - 运动：跑步 25 分钟 / 180 千卡 / 3.2 公里
  - 饮食：早餐「燕麦牛奶」320 千卡
  - 睡眠：23:30–07:00 / 7.5 小时 / 良好
- 预置只执行一次，删除后不会重复预置。

## 7. 核心功能演示路径

| 功能 | 路径 |
| --- | --- |
| 记录 CRUD | 首页 →「记录管理」→ 运动/饮食/睡眠 Tab → 新增/编辑/删除 |
| AI 结构化分析 | 首页 →「AI 分析建议」→「生成今日分析」 |
| 智能问答（多轮） | 「AI 分析建议」→ 底部输入框提问；顶部会话栏可新建/切换/删除 |
| 拍照识别（相册） | 首页 →「拍照识别」→ 拖图进模拟器 →「从相册选择图片」 |
| 拍照识别（样例） | 「拍照识别」→ 点「苹果/米饭/牛奶」 |
| 保存与复盘 | 识别结果「保存为饮食记录」/「保存建议」；AI 页「历史」Tab 查看 |

## 8. 云端 AI 配置方式与密钥占位提醒

云端 AI 集中在 `entry/src/main/ets/common/Config.ets`：

- **文本模型** `AiConfig`：默认 DeepSeek，`MODEL = deepseek-chat`。
- **视觉模型** `VisionConfig`：默认 DeepSeek 视觉，`MODEL = deepseek-v4-flash-vision-exp`，`API_KEY` 直接引用 `AiConfig.API_KEY`。

```typescript
// AiConfig
static readonly API_KEY: string = 'sk-替换成你的Key';   // ← 占位，务必替换
// VisionConfig
static readonly API_KEY: string = AiConfig.API_KEY;      // 复用文本 Key
```

> **密钥提醒**：API Key 仅填在 `Config.ets`，以占位符形式提供；严禁提交到日志/注释/文档/Git。`isApiKeyConfigured()` / `isConfigured()` 通过判断「非空且不含 REPLACE」决定是否走云端。

## 9. 使用的云端 AI 服务、免费/付费方式与额度限制

- 服务：DeepSeek 云端 API（文本 `deepseek-chat` + 视觉 `deepseek-v4-flash-vision-exp`），OpenAI 兼容协议。
- 付费方式：**按 token 计费**，不存在稳定免费额度承诺；演示规模下费用极低，但仍需自行确认账户余额。
- 额度/费用风险：402（余额不足）、429（限流）；请求期间按钮会禁用防重复计费；图片先压缩（1024px/JPEG80）控制成本。
- 降级：未配置 Key 或调用失败时自动走本地规则，零成本演示。

## 10. 失败时的提示方式

- 未配置 Key 或云端失败 → 界面显示「本地降级建议 / 本地降级识别」角标，摘要中以 `【降级】` 前缀写明具体原因（401/402/403/404/超时等）。
- 关键日志走 `Logger`（hilog 域 `0x2A2A`），标签 `HealthAgent.Service/Index/Record/Advice/Camera`。

## 11. 权限说明

`entry/src/main/module.json5` 已声明 `ohos.permission.INTERNET`（`reason` 引用 `string.json` 的 `internet_permission_reason`）。

## 12. AI 工具使用说明

本项目在开发过程中使用 AI 助手完成：工程骨架梳理、分层设计、ArkTS 代码生成、云端 AI 接口对接、故障排查。所有生成代码均经人工理解、修改、编译运行核验；未使用第三方开源项目，样例图为自绘占位图。
