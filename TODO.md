# 项目任务拆分

> 说明：  
> - 优先级：1 最高，10 最低  

## 一、基础设施
- [ ] **修改 Element Plus 主题**
   - 优先级：9  
   - 模块：`src/assets/styles/` 等 
   - 复杂度：S  
   - 内容：围绕山大红 #9C0C13 设置
        [element-plus自定义主题色](https://juejin.cn/post/7264952002706096164)

- [ ] **维护 mock**
    - 优先级：7  
    - 模块：`mocks/`
    - 复杂度：M 
    - 内容：暂时让 AI 写了个 mock，目前只有 auth 用户认证基本能用，你需要保证其他模块被用到时，能用。
    可基于此 mock 继续，也可重设 mock 方案（msw、vite-plugin-mock等），重构。
    （这是个长期活，后端API能用就直接用后端提供的，后端API未开发好时才需要，优先级不高）

- [ ] **从明白书里扒奖项**
   - 优先级：P0  
   - 模块：`src/utils/award-dicts.js` 
   - 复杂度：S  
   - 内容：从 [计科本科明白书](https://dq7ann1hev.feishu.cn/wiki/wikcnKQsRyhsF0SBbUtzqlAvEyd) 里扒奖项，填入 `award-dicts.js` 中

---

## 二、登录
- [ ] **统一错误处理与登录状态续期（refresh token 自动调用）**  
    - 优先级：10  
    - 模块：`services/http.js`, `stores/auth.js`  
    - 复杂度：S  
    - 内容：  
      - 在响应拦截器中遇到 `code=1005/1006` 时，自动尝试刷新 token 或跳转登录。  
      - 提升整体体验，但不阻塞业务功能开发。

---

## 三、路由
- [ ] **补充各角色路由**
    - 优先级：1  
    - 模块：`src/router/index.js`  
    - 复杂度：S  
    - 内容：仿着学生的“身心健康：基础”，把其他模块都填充好。

---

## 四、布局与通用组件
- [ ] **layout：导航栏**
    - 优先级：2  
    - 模块：`components/layout/xxxxLayout.vue` 
    - 复杂度：S  
    - 内容：按需调整各角色 layout 的导航栏部分

- [ ] **封装：带有操作按钮的表格、嵌有表单的对话框**
    - 优先级：1  
    - 模块：`components/tables/StudentTable.vue`, `components/forms/AddApplicationForm.vue`等
    - 复杂度：M
    - 内容：利用 pinia store 和 element plus，封装带有操作按钮的表格table、嵌有表单form的对话框dialog，以供 `StudentPhysicalMentalBasic.vue` 等页面插入

---

## 五、学生申报模块（核心入口）
### 1. 学生申报基础能力
- [ ] **StudentDashboard：个人得分概览（按分类汇总）**  
   - 优先级：1  
   - 模块：`views/student/StudentDashboard.vue`, `services/applicationService.js`, `stores/application.js`  
   - 复杂度：-  
   - 内容：调用 `/applications/my/category-summary` 加载当前学期各分类得分与总分，展示卡片/图表；支持简单的学期切换（term 下拉）。

- [ ] **applicationService：完善学生申报相关接口封装**  
   - 优先级：1  
   - 模块：`services/applicationService.js`  
   - 复杂度：-  
   - 内容：按 `api.md` 补齐/校正方法签名与返回结构（`getMyApplications`, `getCategories`, `getCategorySummary`, `getApplicationsByCategory`, `getApplicationDetail`, `createApplication`, `updateApplication`, `deleteApplication`）。

- [ ] **useApplicationStore：管理学生申报状态与操作**  
   - 优先级：1  
   - 模块：`stores/application.js`  
   - 复杂度：-  
   - 内容：  
     - 统一 state：我的申报列表、按分类明细、当前选中申报、加载状态与错误信息。  
     - 方法：`fetchMyApplications`, `fetchMyCategorySummary`, `fetchMyCategoryApplications`, `fetchApplicationDetail`, `createApplication`, `updateApplication`, `deleteApplication`。  
     - 与 `applicationService` 解耦好调用参数和响应结构。

### 2. 学生按分类管理申报
- [ ] **StudentCategoryModulePage：按 route params 显示分类标题与上限分**  
   - 优先级：2  
   - 模块：`views/student/StudentCategoryModulePage.vue`, `utils/award-dicts.js`  
   - 复杂度：-  
   - 内容：  
     - 根据路由参数（如 `categoryId=physical_mental&subId=basic`）从 `CATEGORY_TREE` 中解析出标题、副标题（上限分）。  
     - 页面骨架：顶部显示当前模块名称、最大分值，底部列表区域预留。

- [ ] **在 StudentCategoryModulePage 调用 store：加载当前分类的申报列表**  
   - 优先级：2  
   - 模块：`views/student/StudentCategoryModulePage.vue`, `stores/application.js`  
   - 复杂度：-  
   - 内容：  
     - 使用 `useApplicationStore` 的 `fetchMyCategoryApplications`（包装 `/applications/my/by-category`）。  
     - 支持基本筛选：状态（`pending_ai/pending_review/approved/...`）、关键词、学期。  
     - 将结果以表格/列表形式展示（临时使用简单 `el-table`，后续可替换为封装的 `StudentTable`）。

- [ ] **学生申报表单组件：AddApplicationForm + EditApplicationForm**  
   - 优先级：2  
   - 模块：`components/forms/AddApplicationForm.vue`, `components/forms/EditApplicationForm.vue`  
   - 复杂度：-  
   - 内容：  
     - 基本字段：`award_type`, `award_level`, `title`, `description`, `occurred_at`, `attachments`。  
     - 新建表单：调用 `applicationStore.createApplication`。  
     - 编辑表单：加载已有数据，调用 `applicationStore.updateApplication`；限制只有特定状态（如 pending_ai/pending_review 之前）允许编辑。  

- [ ] **封装学生申报列表表格：StudentTable**  
   - 优先级：3  
   - 模块：`components/tables/StudentTable.vue`  
   - 复杂度：-  
   - 内容：  
     - 使用 Element Plus `el-table`，展示标题、状态、单项分、创建时间等。  
     - 操作列：查看详情、编辑、删除；通过 `emit` 通知父组件执行。  
     - 支持分页（page/size 透传 store 调用）。

- [ ] **在 StudentCategoryModulePage 中集成 StudentTable + 新建/编辑表单弹窗**  
   - 优先级：3  
   - 模块：`views/student/StudentCategoryModulePage.vue`  
   - 复杂度：-  
   - 内容：  
     - 顶部“新建申报”按钮 -> 弹出 `AddApplicationForm` 对话框。  
     - 表格操作：编辑 -> 弹出 `EditApplicationForm`；删除 -> 调用 `applicationStore.deleteApplication` 并刷新列表。

### 3. 学生申报详情与文件上传
- [ ] **文件上传逻辑：集成 files/upload 接口**  
   - 优先级：3  
   - 模块：`services/fileService.js`, `components/forms/AddApplicationForm.vue`, `components/forms/EditApplicationForm.vue`  
   - 复杂度：-  
   - 内容：  
     - 在表单中加入“上传证明材料”（图片/PDF），使用 `el-upload` 或自定义上传逻辑，调用 `fileService.uploadFile`。  
     - 将返回的 `file_id` 写入申报 `attachments` 字段；支持移除文件（调用 `fileService.deleteFile`）。

- [ ] **学生申报详情页（含 AI 审核结果预留）**  
    - 优先级：4  
    - 模块：`views/student/ApplicationDetailPage.vue`, `stores/application.js`, `services/aiAuditService.js`  
    - 复杂度：-  
    - 内容：  
      - 基本信息：标题、描述、状态、得分、创建/更新时间。  
      - 文件列表：展示附件下载链接（可先只展示 `file_id`，后续接 `getFileUrl`）。  
      - AI 审核结果区域预留（后面在“学生（审核人）审查 / AI 审核模块”中补内容）。

---

## 六、学生（审核人）审查模块
> 依赖：  
> - 学生申报模块已可创建/管理申报  
> - AI 审核模块 service 已可获取报告（基础已在 `aiAuditService`）

### 4. 审核人首页与待审核统计
- [ ] **reviewService：完善 `/reviews/**` 接口封装**  
    - 优先级：2  
    - 模块：`services/reviewService.js`  
    - 复杂度：-  
    - 内容：  
      - 校正现有方法签名，统一为 `(params)` / `(applicationId, payload)` 形式，与 `api.md` 对齐：  
        - `getPendingReviews(params)`  
        - `getPendingCategorySummary(params)`  
        - `getPendingByCategory(params)`  
        - `getReviewDetail(applicationId)`  
        - `submitReviewDecision(applicationId, decisionPayload)`  
        - `getReviewHistory(params)`  
        - `getPendingCount()`

- [ ] **useReviewStore：管理待审核列表和审核记录**  
    - 优先级：2  
    - 模块：`stores/review.js`  
    - 复杂度：-  
    - 内容：  
      - state：`pendingList`, `pendingSummary`, `pendingByCategory`, `currentDetail`, `historyList`, `pendingCount`, `loading`, `error`。  
      - actions：`fetchPendingReviews`, `fetchPendingSummary`, `fetchPendingByCategory`, `fetchReviewDetail`, `submitDecision`, `fetchReviewHistory`, `fetchPendingCount`。  
      - 与 `reviewService` 和 AI 模块数据结构对齐。

- [ ] **ReviewerDashboard：待审核数量与分类统计展示**  
    - 优先级：3  
    - 模块：`views/reviewer/ReviewerDashboard.vue`  
    - 复杂度：-  
    - 内容：  
      - 加载 `reviewStore.fetchPendingCount()`，在首页展示当前待审核数量。  
      - 加载 `reviewStore.fetchPendingSummary()`（按分类汇总），用表格或卡片展示各分类待审核数、已通过/已驳回等。

### 5. 审核人按分类管理申报
- [ ] **ReviewerCategoryModulePage：按分类展示待审核列表框架**  
    - 优先级：3  
    - 模块：`views/reviewer/ReviewerCategoryModulePage.vue`, `utils/award-dicts.js`  
    - 复杂度：-  
    - 内容：  
      - 与 `StudentCategoryModulePage` 类似，根据 `CATEGORY_TREE` 显示标题。  
      - 筛选：班级（基于当前审核人绑定的班级）、状态、学期。  
      - 使用 `reviewStore.fetchPendingByCategory` 加载列表（`/reviews/pending/by-category`）。  

- [ ] **审核详情页：加载申报详情 + AI 审核报告**  
    - 优先级：4  
    - 模块：`views/reviewer/ReviewDetailPage.vue`, `services/aiAuditService.js`, `stores/review.js`  
    - 复杂度：-  
    - 内容：  
      - 基础信息：来自 `/reviews/{application_id}` 与 `/applications/{id}`。  
      - 调用 `aiAuditService.getAuditReport(applicationId)` 展示 AI 审核报告（OCR 文本、比对结果、得分拆分等）。  
      - 支持查看学生上传的附件列表。

- [ ] **审核决策表单：通过/驳回 + 驳回原因**  
    - 优先级：4  
    - 模块：`views/reviewer/ReviewDetailPage.vue`, `components/forms/ReviewApplicationForm.vue`  
    - 复杂度：-  
    - 内容：  
      - 表单字段：`decision`(`approved/rejected`), `comment`, `reason_code`, `reason_text`。  
      - 决策提交：调用 `reviewStore.submitDecision` -> `reviewService.submitReviewDecision`。  
      - 成功后：刷新当前分类列表 & 待审核数量，必要时跳转回列表页。

---

## 七、教师审查模块
> 依赖：  
> - 学生申报流程已能产生数据  
> - 审核人已能完成一轮审核（状态到 `approved/rejected`）

### 6. 教师全局审核与统计
- [ ] **teacherService：完善 `/teacher/**` 和年级查看接口封装**  
    - 优先级：3  
    - 模块：`services/teacherService.js`  
    - 复杂度：-  
    - 内容：  
      - 与 `api.md` 对齐：  
        - `getAllApplications(params)`（全局申报查询）  
        - `recheckApplication(applicationId, payload)`  
        - `archiveApplications(payload)`  
        - `getStatistics(params)`  
        - `exportData(payload)`  
        - `getExportTaskResult(taskId)`  
        - `getGradeOverview(grade, params)` (`/counselor/grades/{grade}/classes/overview`)  
        - `getGradeClassOverview(grade, classId, params)`（已有，校对参数名）

- [ ] **useTeacherStore：教师端全局数据与状态管理**  
    - 优先级：3  
    - 模块：`stores/teacher.js`  
    - 复杂度：-  
    - 内容：  
      - state：`applications`, `statistics`, `classesOverview`, `classDetail`, `loading`, `error`。  
      - actions：`fetchApplications`, `fetchStatistics`, `fetchGradeOverview`, `fetchClassDetail`，为后续分类页和导出/归档复用。

- [ ] **TeacherDashboard：教师统计看板**  
    - 优先级：4  
    - 模块：`views/teacher/TeacherDashboard.vue`  
    - 复杂度：-  
    - 内容：  
      - 调用 `teacherStore.fetchStatistics` 和 `teacherService.getGradeOverview`。  
      - 展示全局申报数量、通过/驳回/待审核数量、总分/均分，以及按奖项类型拆分图表。  

- [ ] **TeacherCategoryModulePage：按分类/班级查看全局申报列表**  
    - 优先级：4  
    - 模块：`views/teacher/TeacherCategoryModulePage.vue`, `stores/teacher.js`  
    - 复杂度：-  
    - 内容：  
      - 支持筛选：年级、班级、奖项类型、状态、关键词。  
      - 使用 `teacherStore.fetchApplications`（封装 `/teacher/applications`）。  
      - 列表中提供查看详情、发起复核（recheck）入口。

- [ ] **教师复核申报（改判）功能**  
    - 优先级：5  
    - 模块：`views/teacher/TeacherCategoryModulePage.vue`（或单独 `TeacherReviewDetailPage.vue`）  
    - 复杂度：-  
    - 内容：  
      - 弹出对话框，提交 `target_status` 和 `reason` -> 调用 `teacherService.recheckApplication`。  
      - 成功后刷新当前列表，并在备注中标记为“教师复核修改”。

---

## 八、公示模块：学生查看公示 & 教师发布公示
> 依赖：  
> - 导出/归档模块生成的归档记录（后面会用 `archive_id` 绑定）  
> - 公示和申诉 API

### 7. 公示后端接口 service 与 store
- [ ] **announcementService：封装 `/announcements` 接口**  
    - 优先级：4  
    - 模块：`services/announcementService.js`  
    - 复杂度：-  
    - 内容：  
      - `getAnnouncements(params)`  
      - `createAnnouncement(payload)`  
      - `updateAnnouncement(id, payload)`  
      - `getAnnouncementDetail(id)`（如需要）  
      - 结构对齐 `api.md`。

- [ ] **useAnnouncementStore：管理公示列表状态**  
    - 优先级：4  
    - 模块：`stores/announcement.js`  
    - 复杂度：-  
    - 内容：  
      - 校正当前 store：使用 `announcementService` 返回的 `data`，支持分页与筛选参数。  
      - 提供 `fetchAnnouncements`, `getAnnouncementById` 等方法。

### 8. 学生查看公示
- [ ] **AnnouncementListPage：学生公示列表基础页**  
    - 优先级：4  
    - 模块：`views/announcements/AnnouncementListPage.vue`, `stores/announcement.js`  
    - 复杂度：-  
    - 内容：  
      - 使用 `announcementStore.fetchAnnouncements` 加载公示列表。  
      - 展示公示标题、适用年级/班级、起止时间、状态（进行中/已结束）。  
      - 提供“查看详情/明细表”入口（后续可指向一个统计表或静态视图）。

- [ ] **AnnouncementPage（学生查看单条公示详情）扩展**  
    - 优先级：5  
    - 模块：`views/announcements/AnnouncementPage.vue`  
    - 复杂度：-  
    - 内容：  
      - 使用路由参数 `announcementId` 加载单条公示详情。  
      - 展示公示绑定的 `archive_id` 对应的整体结果概要（与归档模块打通后进一步完善）。

### 9. 教师发布/管理公示
- [ ] **Teacher 公示管理页：AnnouncementManagePage**  
    - 优先级：5  
    - 模块：`views/teacher/AnnouncementManagePage.vue`, `services/announcementService.js`  
    - 复杂度：-  
    - 内容：  
      - 教师查看自己发布的所有公示。  
      - 新建/编辑公示：设置标题、适用年级/班级、时间范围、展示字段、绑定 `archive_id`。  
      - 支持关闭/删除公示（可选）。

---

## 九、令牌管理：教师生成/删除 & 学生绑定审核人令牌
> 依赖：  
> - 基本认证和角色系统已存在  
> - 教师端 & 学生端个人信息页将复用这一能力

### 10. 令牌 service 与表格/表单组件
- [ ] **教师端令牌相关接口封装（可复用现有 teacherService 或单独 tokenService）**  
    - 优先级：4  
    - 模块：`services/teacherService.js` 或 `services/tokenService.js`  
    - 复杂度：-  
    - 内容：  
      - `generateReviewerToken(payload)`（已有，校对参数）  
      - `getTokens(params)` -> `/tokens`  
      - `revokeToken(tokenId)` -> `/tokens/{token_id}/revoke`  

- [ ] **TokenTable：令牌管理表格组件**  
    - 优先级：5  
    - 模块：`components/tables/TokenTable.vue`  
    - 复杂度：-  
    - 内容：  
      - 展示令牌 ID、适用班级、过期时间、状态等。  
      - 操作列：失效/撤销。  

- [ ] **AddTokenForm：生成审核人令牌表单组件**  
    - 优先级：5  
    - 模块：`components/forms/AddTokenForm.vue`  
    - 复杂度：-  
    - 内容：  
      - 字段：班级列表、多选、过期时间。  
      - 提交到 `generateReviewerToken`，并刷新 `TokenTable`。

- [ ] **教师端令牌管理页：TokenManagementPage**  
    - 优先级：5  
    - 模块：`views/teacher/TokenManagementPage.vue`  
    - 复杂度：-  
    - 内容：  
      - 集成 `TokenTable` + `AddTokenForm`。  
      - 支持根据状态、班级筛选。

### 11. 学生绑定审核人令牌 & 个人信息
- [ ] **学生端绑定审核人令牌接口：reviewer token activate**  
    - 优先级：5  
    - 模块：`services/authService.js` 或 `services/tokenService.js`  
    - 复杂度：-  
    - 内容：  
      - 封装 `/tokens/reviewer/activate` 方法。  
      - 调用后需要刷新当前用户信息（`authStore.fetchCurrentUser`），以更新 `is_reviewer` 与可见菜单。

- [ ] **StudentProfilePage：学生个人信息页（含绑定令牌）**  
    - 优先级：5  
    - 模块：`views/student/StudentProfilePage.vue`, `stores/auth.js`  
    - 复杂度：-  
    - 内容：  
      - 展示基本信息：学号、姓名、班级、是否为审核人（`is_reviewer`）、邮箱等。  
      - 绑定令牌表单：输入 token 字符串 -> 调用 activate 接口，成功后通过 `authStore.fetchCurrentUser` 刷新视图。  
      - 个人信息编辑（邮箱/手机号）：调用 `authService.updateUserInfo`。

- [ ] **TeacherProfilePage：教师个人信息页（含邮箱）**  
    - 优先级：6  
    - 模块：`views/teacher/TeacherProfilePage.vue`, `stores/auth.js`  
    - 复杂度：-  
    - 内容：  
      - 与 `StudentProfilePage` 类似，展示登录教师的基本信息。  
      - 支持编辑邮箱、手机号等，调用 `authService.updateUserInfo`。

- [ ] **AdminProfilePage：管理员个人信息页**  
    - 优先级：7  
    - 模块：`views/admin/AdminProfilePage.vue`  
    - 复杂度：-  
    - 内容：  
      - 简单展示管理员信息与基本编辑能力（可选）。  

---

## 十、教师导出与归档管理
> 依赖：  
> - 教师端已能查询统计和应用列表  
> - 导出/归档 API 已在 service 层封装

### 12. 教师导出任务
- [ ] **导出任务创建与轮询结果**  
    - 优先级：5  
    - 模块：`services/teacherService.js`, `views/teacher/TeacherExportPage.vue`（可新建）  
    - 复杂度：-  
    - 内容：  
      - 使用 `teacherService.exportData` 创建导出任务。  
      - 记录 `task_id` 并使用 `teacherService.getExportTaskResult(taskId)` 轮询，获取导出文件下载地址。  
      - 提供导出记录列表简单展示（可在同一页）。

### 13. 归档管理（依赖导出）
- [ ] **archiveService：已实现基础接口，补充 store & 页面**  
    - 优先级：5  
    - 模块：`stores/archive.js`（新建）, `services/archiveService.js`  
    - 复杂度：-  
    - 内容：  
      - store state：`archives`, `currentArchive`, `loading`, `error`。  
      - actions：`createArchive`, `fetchArchiveList`, `fetchArchiveDetail`.

- [ ] **ArchiveManagementPage：教师归档管理页**  
    - 优先级：6  
    - 模块：`views/teacher/ArchiveManagementPage.vue`  
    - 复杂度：-  
    - 内容：  
      - 展示归档列表（term、grade、class_ids、总分等）。  
      - 提供创建归档表单：选择某次导出任务 `task_id`，设置归档名称、学期、年级、班级范围。  
      - 支持查看归档详情与下载归档文件（使用 `downloadArchive`）。

---

## 十一、公示申诉管理 & 邮件
> 依赖：  
> - 公示模块已可绑定某次归档  
> - 邮件与申诉 API

### 14. 邮件模块
- [ ] **notificationService：已封装，增加简单的日志查看页（可选）**  
    - 优先级：7  
    - 模块：`views/system/EmailLogsPage.vue`, `services/notificationService.js`  
    - 复杂度：-  
    - 内容：  
      - 展示 `/notifications/email-logs` 结果（状态、时间、收件人、原因）。  
      - 仅教师/管理员可访问。

### 15. 申诉模块（学生端 & 教师端）
- [ ] **appealService：封装 `/appeals/**` 接口**  
    - 优先级：6  
    - 模块：`services/appealService.js`  
    - 复杂度：-  
    - 内容：  
      - `createAppeal(payload)`  
      - `getAppeals(params)`  
      - `processAppeal(appealId, payload)`  

- [ ] **useAppealStore：申诉状态管理**  
    - 优先级：6  
    - 模块：`stores/appeal.js`  
    - 复杂度：-  
    - 内容：  
      - state：`myAppeals`, `managedAppeals`, `currentAppeal`, `loading`, `error`。  
      - actions：`fetchMyAppeals`, `fetchManagedAppeals`, `createAppeal`, `processAppeal`。

- [ ] **学生端申诉列表页：AppealListPage**  
    - 优先级：6  
    - 模块：`views/student/AppealListPage.vue`, `stores/appeal.js`  
    - 复杂度：-  
    - 内容：  
      - 学生查看自己发起的所有申诉记录；  
      - 可从某条公示/申报详情跳转过来，对应筛选。

- [ ] **学生提交申诉页：AppealCreatePage**  
    - 优先级：6  
    - 模块：`views/student/AppealCreatePage.vue`  
    - 复杂度：-  
    - 内容：  
      - 选择关联的 `announcement_id` 或 `application_id`（可先只支持从公示详情跳转带参数）。  
      - 输入内容和附件（可重用文件上传逻辑）。  
      - 提交后写入 `appealStore.createAppeal`。

- [ ] **教师申诉处理页：AppealProcessPage**  
    - 优先级：7  
    - 模块：`views/teacher/AppealProcessPage.vue`, `stores/appeal.js`, `services/notificationService.js`  
    - 复杂度：-  
    - 内容：  
      - 列表展示待处理申诉。  
      - 处理表单：`result`（accepted/rejected）、`reply`（答复）。  
      - 处理完成后：考虑调用 `notificationService.sendRejectionEmail` 或其他通知方式。

---

## 十二、管理员日志模块
### 16. 系统配置与日志
- [ ] **systemService & systemStore：与 API 对齐**  
    - 优先级：7  
    - 模块：`services/systemService.js`, `stores/system.js`  
    - 复杂度：-  
    - 内容：  
      - 修正 store 中错误的方法名（如 `getAwardTypes/getAwardLevels/getSystemConfig` 应与 service 对齐为 `getSystemConfigs/getSystemLogs/getAwardDicts`）。  
      - 统一 state 和 actions 名称：`fetchSystemConfigs`, `updateSystemConfigs`, `fetchLogs`, `fetchAwardDicts`。

- [ ] **SystemConfigPage：系统参数配置页**  
    - 优先级：8  
    - 模块：`views/system/SystemConfigPage.vue`  
    - 复杂度：-  
    - 内容：  
      - 调用 `systemStore.fetchSystemConfigs` 加载配置。  
      - 支持编辑上传限制、奖项类型/等级等，并通过 `updateSystemConfigs` 提交。

- [ ] **AwardDictsPage：奖项字典管理页**  
    - 优先级：8  
    - 模块：`views/system/AwardDictsPage.vue`, `services/systemService.js`  
    - 复杂度：-  
    - 内容：  
      - CRUD `/system/award-dicts`，维护后端使用的奖项字典。  
      - 可作为 `award-dicts.js` 的补充或后台配置。

- [ ] **SystemLogsPage：管理员日志查看页**  
    - 优先级：8  
    - 模块：`views/system/SystemLogsPage.vue`, `stores/system.js`  
    - 复杂度：-  
    - 内容：  
      - 使用 `systemStore.fetchLogs` 加载系统操作日志。  
      - 支持按操作人、动作、时间范围筛选，分页展示。

---
