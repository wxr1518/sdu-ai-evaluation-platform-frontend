## 1. 工程简介

本项目是一个基于 **Vite + Vue3 + Pinia + Vue Router + Element Plus** 的单页应用，用于支持“综测智能评定”业务的前端界面。

核心目标：

- 支持学生在线申报各类综测加分项目；
- 接入 AI 审核结果，辅助班委/教师进行审核；
- 为教师提供统计、导出、归档、公示、申诉处理等管理能力；
- 为管理员提供系统配置与日志查看能力。

node版本：v20.19.1

后端接口规范见仓库内的 [`api.md`](./api.md) 和 [`all-api.md`](./all-api.md)。

后端地址 <待补充>

---

### 2. 业务角色

系统内主要有 4 类角色

- `student`：学生  
  - 可发起/管理本人申报、查看公示、发起申诉、绑定审核人令牌（成为审核人）。
- `reviewer`：审核人（一般是班委，由学生账号 + 审核人标记构成）  
  - 可查看并审核所管辖班级的学生申报，参考 AI 审核报告给出决策。
- `teacher`：教师/辅导员  
  - 可全局查看与复核申报、统计分析、导出与归档数据、发布公示、处理申诉、管理审核人令牌。
- `admin`：管理员  
  - 可进行系统配置与系统日志查询等后台管理操作。

前端通过登录后返回的用户信息 & 角色字段，控制路由访问与菜单展示。

## 3. 主要功能模块一览

下面按“功能模块”划分，便于从业务视角快速理解整个系统。

### 3.1 登录与认证

- 相关接口：`/auth/login`, `/auth/refresh`, `/auth/logout`, `/users/me` …
- 主要能力：
  - 用户登录，获取 `access_token` / `refresh_token`；
  - 登录态持久化与续期（前端通过拦截器和 Pinia `auth` store 管理 token）；
  - 基于角色的路由守卫与菜单控制。

---

### 3.2 学生申报模块（学生侧核心）

学生可为当前学期的各类奖项/活动发起申报，并上传证明材料。

主要页面与能力（典型设计，具体以代码实现为准）：

- **学生首页 / 综测概览（StudentDashboard）**
  - 调用：`GET /applications/my/category-summary`
  - 展示当前学期各大类（德智体美劳等）的汇总得分和总分；
  - 支持切换学期（`term` 下拉）。

- **按分类管理申报（StudentCategoryModulePage）**
  - 调用：
    - `GET /applications/categories`
    - `GET /applications/my/by-category`
  - 通过路由参数（如 `category=physical_mental&sub_type=basic`）解析分类名称与上限分；
  - 以表格展示该分类下的所有申报，支持状态 / 学期 / 关键字筛选。

- **新建 / 编辑申报**
  - 接口：
    - `POST /applications`
    - `PUT /applications/{application_id}`
    - `DELETE /applications/{application_id}`
  - 表单字段（示例）：
    - `award_type`, `award_level`, `title`, `description`, `occurred_at`, `attachments` 等；
  - 限制：
    - 仅在特定状态（如 `pending_ai` / `pending_review` 之前）允许编辑或删除。

- **文件上传**
  - 接口：
    - `POST /files/upload`
    - `DELETE /files/{file_id}`
    - `GET /files/{file_id}/url`
  - 主要用于上传各类证书、奖状扫描件（PDF/图片）。

- **申报详情页**
  - 接口：
    - `GET /applications/{application_id}`
    - 预留：`GET /ai-audits/{application_id}/report`
  - 展示：基本信息、得分、附件列表、AI 审核结果区域（学生通常看到脱敏结果）。

---

### 3.3 学生（审核人）审查模块

当学生绑定了审核人令牌，并被系统标记为 `reviewer` 后，将看到审核人视图。

- **审核人首页（ReviewerDashboard）**
  - 接口：
    - `GET /reviews/pending-count`
    - `GET /reviews/pending/category-summary`
  - 展示自己管辖班级当前的待审核数量、按分类统计情况。

- **按分类/班级查看待审核列表（ReviewerCategoryModulePage）**
  - 接口：
    - `GET /reviews/pending`
    - `GET /reviews/pending/by-category`
  - 支持按班级、分类、状态、学期筛选，列表化展示待审核申报。

- **审核详情 + AI 报告（ReviewDetailPage）**
  - 接口：
    - `GET /reviews/{application_id}`
    - `GET /applications/{application_id}`
    - `GET /ai-audits/{application_id}/report`
  - 展示：
    - 申报详情、附件；
    - AI 审核结果（OCR 文本、身份信息匹配情况、奖项信息一致性、风险点提示等）。

- **提交审核决策**
  - 接口：`POST /reviews/{application_id}/decision`
  - 字段：
    - `decision`: `approved` / `rejected`
    - `comment`: 处理意见
    - `reason_code` / `reason_text`: 驳回原因（仅驳回时必填）

---

### 3.4 教师审查与统计模块

教师可以从全局视角管理所有学生申报记录，并进行复核。

- **全局申报查询**
  - 接口：`GET /teacher/applications`
  - 支持按年级、班级、奖项类型、状态、关键词等多维度筛选。

- **教师复核 / 改判**
  - 接口：`POST /teacher/applications/{application_id}/recheck`
  - 用于在特殊情况下对已审核结果进行人工复核与状态修改。

- **统计看板（TeacherDashboard）**
  - 接口：
    - `GET /teacher/statistics`
    - `GET /counselor/grades/{grade}/classes/overview`
    - `GET /counselor/grades/{grade}/classes/{class_id}`
  - 展示：全局申报数、通过/驳回/待审核数量、按奖项类型/班级/年级拆分的统计数据。

- **年级视图**
  - 用于辅导员从年级-班级角度查看整体情况，支持按学期查看。

---

### 3.5 教师导出与归档管理模块

这一部分是“数据闭环”和“对外输出”的关键。

#### 3.5.1 导出任务（Export）

- 接口：
  - `POST /teacher/exports` 创建导出任务
  - `GET /teacher/exports/{task_id}` 查询导出结果
- 流程：
  1. 教师设置导出范围（如：年级=2023，状态=approved，格式=xlsx）；
  2. 系统创建导出任务，返回 `task_id`（如 `exp_10001`）；
  3. 前端轮询查询导出任务状态，拿到最终下载链接。

#### 3.5.2 归档管理（Archive）

- 接口：
  - `POST /archives/exports` 创建归档记录
  - `GET /archives/exports` 列表
  - `GET /archives/exports/{archive_id}` 详情（含下载地址）
  - `GET /archives/exports/{archive_id}/download` 下载文件
- 典型使用方式：
  1. 教师在某一时间点（如学期末），确认全部申报处理完毕；
  2. 使用 `/teacher/applications/archive` 对一批申报做“业务归档”（记录状态为 `archived`）；
  3. 再使用 `/teacher/exports` 导出数据（例：`exp_10123`）；
  4. 调用 `/archives/exports` 创建归档记录，将此次导出绑定到档案系统；
  5. 今后教师可在“归档列表”中查看/下载这些历史归档结果，并将其用作公示或留档。

---

### 3.6 公示与申诉模块

#### 3.6.1 公示（Announcement）

- 接口：
  - `POST /announcements`
  - `GET /announcements`
- 教师发布公示：
  - 在归档完成后，选择某个 `archive_id`，设置：
    - 公示标题
    - 适用年级/班级范围
    - 公示时间（开始/结束）
    - 展示字段（如：姓名 / 班级 / 分数 / 排名）
- 学生查看公示：
  - 在公示期内查看班级/年级整体成绩与名次；
  - 可从公示详情跳转到申诉入口。

#### 3.6.2 申诉（Appeal）

- 接口：
  - `POST /appeals`
  - `GET /appeals`
  - `POST /appeals/{appeal_id}/process`
- 学生：
  - 在公示期内或公示结束后，在合理时间窗口内发起申诉；
  - 说明申诉理由，可附带附件（证明材料）。
- 教师：
  - 在“教师申诉处理”页面查看待处理申诉；
  - 根据实际情况更新某一条申报的成绩/状态，并对申诉进行“采纳/驳回”并给出回复。

---

### 3.7 教师/学生个人信息与令牌管理模块

#### 3.7.1 教师生成/删除审核人令牌

- 接口：
  - `POST /tokens/reviewer` 生成令牌
  - `GET /tokens` 查询令牌列表
  - `POST /tokens/{token_id}/revoke` 撤销令牌
- 教师可：
  - 为若干个班级生成可分享给班委的“审核人令牌”；
  - 管理这些令牌：查看状态（有效 / 过期 / 已撤销）、撤销不再使用的令牌。

#### 3.7.2 学生绑定令牌成为审核人

- 接口：`POST /tokens/reviewer/activate`
- 学生在“个人信息”页输入教师发放的 reviewer 令牌：
  - 激活成功后，账号获得审核人权限，并能访问审核模块。

#### 3.7.3 学生 / 教师个人信息管理

- 接口：
  - `GET /users/me`
  - `PUT /users/me`
- 学生 / 教师可：
  - 查看自己的基本信息（学号/工号、姓名、所属班级/院系等）；
  - 修改个人邮箱、手机号等联系方式；
  - 学生侧：在该页面附带“绑定审核人令牌”的交互。

---

### 3.8 邮件通知模块

- 接口：
  - `POST /notifications/reject-email`
  - `GET /notifications/email-logs`
- 主要用途：
  - 在 AI 审核或人工审核驳回时，向学生发送邮件通知（原因说明、后续操作提示）；
  - 教师/管理员可查看邮件发送日志（成功/失败记录）。

---

### 3.9 管理员日志与系统配置模块

- 接口：
  - `GET /system/configs`
  - `PUT /system/configs`
  - `GET /system/logs`
  - `GET/POST/PUT/DELETE /system/award-dicts`
- 能力：
  - 维护系统级配置（如可选奖项类型/等级、上传大小限制、邮箱模板等）；
  - 管理奖项字典（后端统一使用的奖项枚举）；
  - 查看系统操作日志，支持按操作人、动作、时间范围筛选。

---

## 4. 关键业务流程（文字版）

下面用一条典型“从申报到公示”的主线，串起上面所有模块。

1. **学生提交申报**
   - 学生在某个分类下新建申报 -> 状态：`pending_ai`。

2. **AI 审核（系统自动）**
   - 系统将新申报放入 AI 队列；
   - 获取 AI 报告：
     - 若存在异常 -> 状态可能为 `ai_abnormal`，并可通过邮件通知学生；
     - 若 AI 判断正常 -> 状态流转到 `pending_review`。

3. **学生修改后再次提交（如 AI 异常 or 人工驳回）**
   - 学生根据 AI 报告或审核意见修改材料，再次提交；
   - 状态重新进入 `pending_ai` -> AI 审核 -> `pending_review`。

4. **审核人（班委）审核**
   - 审核人在待审核列表中查看该申报；
   - 结合 AI 审核结果与材料，做出决策：
     - 不通过 -> 状态：`rejected`，可触发邮件通知；
     - 通过 -> 状态：`approved`。

5. **教师复核与统计**
   - 教师可随时查看全局统计和任意一条申报记录；
   - 若发现问题，可使用“复核”功能更改申报的最终状态；
   - 根据需要，按年级 / 班级 / 状态导出数据。

6. **教师归档**
   - 当确认某一学期申报流程结束（如 pending_xx 全部处理完或系统时间到达设定截止时间）：
     - 教师调用 `/teacher/applications/archive` 将一批记录标记为 `archived`（本学期业务闭环结束）；
     - 使用 `/teacher/exports` 导出本学期该批次数据，获得导出任务 ID（如 `exp_10123`）；
     - 使用 `/archives/exports` 创建归档记录，将 `exp_10123` 挂到某个 `archive_id` 下。

7. **教师发布公示**
   - 选择上一步生成的 `archive_id`，发布综测公示；
   - 公示期间学生可以查看个人和班级/年级的公示结果。

8. **学生申诉与教师处理**
   - 学生在公示页发起申诉（接口 `/appeals`），说明理由并上传附件；
   - 教师在申诉处理页面查看并处理：
     - 若采纳：可能需要调整对应申报成绩，并在公示数据中体现；
     - 若驳回：给出原因，可选择发送邮件通知。

9. **再归档 / 更新公示**
   - 若申诉导致成绩变化，可对已归档数据进行二次归档；
   - 教师可更换公示绑定的 `archive_id`，使公示内容与最新归档一致。

10. **管理员操作**
    - 维护学期切换、系统配置、日志审计等。

---

## 5. 项目结构（简要）

仅列出与业务强相关的目录，具体以实际代码为准：

- `src/`
  - `views/`
    - `student/`：学生相关页面（仪表盘、分类模块、申报详情、个人信息、申诉等）
    - `reviewer/`：审核人相关页面（待审核列表、审核详情等）
    - `teacher/`：教师相关页面（统计看板、全局申报、导出、归档、公示管理、申诉处理、个人信息等）
    - `system/`：系统配置、日志、邮件日志等
    - `announcements/`：公示列表、公示详情
  - `stores/`：Pinia 状态管理（auth、application、review、teacher、announcement、archive、appeal、system 等）
  - `services/`：对接后端 REST API 的封装（authService, applicationService, reviewService, teacherService, fileService, aiAuditService, announcementService, archiveService, notificationService, systemService, appealService 等）
  - `router/`：路由配置，按角色区分访问范围
  - `assets/styles/`：全局样式与 Element Plus 主题定制
  - `utils/`：工具方法（如 `award-dicts.js`，定义综测分类树与上限分配置）

---

## 6. 本地开发与构建

> 实际脚本请以 `package.json` 为准

```bash
npm install # 安装依赖

npm run dev # 本地开发

npm run build # 构建生产包
```

目前 mock 独立于本项目，启动需新开 Node 进程：

```bash
# cd src\mock
npm install
npm run dev
```