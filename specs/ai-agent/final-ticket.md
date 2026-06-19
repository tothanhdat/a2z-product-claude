# Ticket Title: Action - AI Agent - Core (Sprint 1)

## Epic Summary

**Epic Name:** AI – AI Agent (Core)

**Goal:**
Cho phép automation workflow builder thêm AI Agent node để gọi đến một trong 4 AI providers (`AI Internal`, ChatGPT, Claude AI, Google Gemini) với configurable Prompt và Instructions, trả về AI response cho downstream steps tiêu thụ. Đây là foundation node để chuẩn bị tích hợp Tools/MCP ở Sprint sau.

**Context:**
Node hoạt động stateless single-shot — mỗi run độc lập, chưa hỗ trợ session/memory. Connection cho ChatGPT/Claude/Gemini dùng lại logic Connected Account hiện có của platform; provider `AI Internal` không yêu cầu connect account. Model list được hard-code phía Backend theo từng provider (mỗi provider có 1 default model). Test step thực sự gọi provider, hiển thị response + token usage + latency.

**Expected UI Fields:**
- AI Provider * (Connect)
- Account * (Connect — conditional, ẩn khi AI Provider = `AI Internal`)
- Model * (Connect)
- Prompt * (Configure)
- Instructions (Advanced)

---

## User Stories

### Story 1: Configure AI Provider, Account, and Model

**Story:** As an automation workflow builder, I want to chọn AI Provider, Account, và Model trong Connect step, so that node biết gọi tới đúng AI service với đúng identity.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. AI Provider options và default value**
Given user mở Connect step lần đầu,
When dropdown `AI Provider` được load,
Then hệ thống hiển thị các options sau và pre-select `AI`:

| UI Label | Description |
|---|---|
| AI | AI Internal cho workflow automation. |
| ChatGPT (OpenAI) | Content generation and reasoning. |
| Claude AI (Anthropic) | Long-document analysis and reasoning. |
| Google Gemini | Multimodal AI for text and images. |

**Note:** Provider identifier (API value) do Dev quyết định, align với convention hiện có của các AI nodes khác.

**AC.02. Provider = AI ẩn Account field và load model list của AI Internal**
Given user chọn `AI` ở `AI Provider`,
When UI re-render,
Then field `Account` bị ẩn hoàn toàn,
And field `Model` enable ngay với danh sách models của AI Internal (hard-code BE),
And `Model` được pre-select default model `Qwen 3.5`

**AC.03. Provider ≠ AI hiển thị Account và load model list của provider tương ứng**
Given user chọn `ChatGPT (OpenAI)`, `Claude AI (Anthropic)`, hoặc `Google Gemini`,
When UI re-render,
Then field `Account` hiển thị và là required,
And field `Model` enable với danh sách models của provider vừa chọn (hard-code BE),
And `Model` được pre-select default model của provider đó

**Note:** Default models cho ChatGPT, Claude AI, Google Gemini do PM cung cấp riêng (separate spec doc), không hard-code trong ticket này.

**AC.04. Account is required — cannot continue if not selected (Provider ≠ AI)**
Given `AI Provider` = `ChatGPT (OpenAI)`, `Claude AI (Anthropic)`, hoặc `Google Gemini`,
When field `Account` không có account nào được chọn,
Then nút `Continue` bị disabled,
And inline validation error hiển thị ngay dưới field: "Account is required."

**AC.05. User selects or connects an account via the "Select" button**
Given user click nút `Select` ở field `Account`,
Then hệ thống hiển thị danh sách accounts đã connect với provider tương ứng,
And user có thể chọn một account từ list,
And nếu chưa có account nào, hệ thống redirect sang OAuth flow để connect account mới (dùng lại logic Connected Account hiện có)

**AC.06. Selected account displays name and email for confirmation**
Given user đã chọn account,
Then field `Account` hiển thị account name và email (vd: `Nguyen (nguyennnp@company.io)`),
And user có thể đổi account bằng cách click `Select` lần nữa

**AC.07. Model list không phụ thuộc Account state**
Given user đổi `AI Provider`,
When provider mới được chọn,
Then `Model` list load ngay theo provider (kể cả khi user chưa chọn Account hoặc Account chưa connect xong),
And `Model` luôn có default value khả dụng

**AC.08. Đổi AI Provider reset Account và load model list mới**
Given user đã chọn xong Provider + Account + Model,
When user đổi `AI Provider` sang provider khác,
Then `Account` bị reset về empty (nếu provider mới ≠ AI),
And `Model` load lại danh sách của provider mới và pre-select default model của provider đó

**Edge Cases:**
- Backend chưa trả model list cho provider (network/internal error): hiển thị empty state `"No models available. Please check your connection or permissions."` và disable Continue.
- User đang ở Provider = AI rồi đổi sang provider khác chưa có account nào: Account empty, validation error hiển thị ngay, Model vẫn load được.

---

### Story 2: Enter Prompt for the AI Agent

**Story:** As an automation workflow builder, I want to nhập Prompt cho AI Agent trong Configure step, so that AI biết được task cụ thể cần thực hiện trong từng run.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Prompt is required — cannot continue if empty**
Given Configure step đang mở,
When field `Prompt` để trống sau khi touched,
Then nút `Continue` bị disabled,
And inline validation error hiển thị ngay dưới field: "Prompt is required."

**AC.02. Prompt max length 8,000 characters**
Given user đang nhập vào field `Prompt`,
When length vượt quá 8,000 ký tự,
Then nút `Continue` bị disabled,
And inline validation error hiển thị ngay dưới field: "Prompt must be 8,000 characters or less."

**AC.03. Prompt support manual input và expression**
Given field `Prompt`,
When user nhập text trực tiếp hoặc kéo expression từ upstream step (vd: `{{step_1.output.text}}`),
Then field nhận cả hai loại input,
And expression được resolve khi node execute

**AC.04. Variable picker button (`+`) chèn expression vào Prompt**
Given user click button `+` bên cạnh field `Prompt`,
Then hệ thống mở variable picker hiển thị data có sẵn từ các upstream steps,
And khi user chọn một biến, expression được chèn vào vị trí caret hiện tại trong Prompt

**Edge Cases:**
- Prompt chứa toàn expression và sau khi resolve trả về empty → runtime error `"Prompt resolved to an empty value. Check your variable mapping."`
- Prompt chứa expression resolve ra value > 8,000 ký tự → runtime error `"Prompt must be 8,000 characters or less."`

---

### Story 3: Add Instructions in Advanced step

**Story:** As an automation workflow builder, I want to thêm Instructions (system prompt) trong Advanced step, so that AI tuân theo tone, formatting, hoặc business rules nhất quán giữa các run.

**Priority:** Should-have

**Acceptance Criteria:**

**AC.01. Instructions là optional, default empty**
Given Advanced step đang mở,
When user view field `Instructions`,
Then field không có asterisk (*) và không block Continue khi để trống,
And nếu user để trống, node vẫn execute bình thường (không gửi system prompt)

**AC.02. Instructions max length 4,000 characters**
Given user đang nhập vào field `Instructions`,
When length vượt quá 4,000 ký tự,
Then nút `Continue` bị disabled,
And inline validation error hiển thị ngay dưới field: "Instructions must be 4,000 characters or less."

**AC.03. Instructions chỉ support manual input, không support expression**
Given field `Instructions`,
When user nhập text,
Then chỉ chấp nhận manual input trực tiếp

**AC.04. Instructions được dùng làm system-level guidance cho AI**
Given Instructions có giá trị,
When node execute,
Then nội dung Instructions định hướng tone/format/rules cho toàn bộ response của AI Agent,
And Prompt vẫn là phần task cụ thể của lần run đó

**Edge Cases:**
- Instructions để trống → AI chỉ nhận Prompt, không có system-level guidance.

---

### Story 4: Test AI Agent configuration

**Story:** As an automation workflow builder, I want to test AI Agent với current configuration trong Test step, so that tôi verify được response trước khi save và publish workflow.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Test result hiển thị response, model, latency, token usage**
Given test execution completes successfully,
When kết quả load,
Then panel Test hiển thị:
- `Response` (text AI trả)
- `Model used`
- `Latency` (millisecond)
- `Token usage` (input / output / total — hiển thị `—` nếu provider không trả về)

**AC.02. Test fail hiển thị error message**
Given test execution fail (auth error, runtime error, provider error...),
When error xảy ra,
Then panel Test hiển thị error message theo standard error catalog tại mục Error Handling

---

### Story 5: Handle authentication and runtime errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when issues occur, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. OAuth token expired or revoked**
Given connected account's OAuth token đã expired hoặc bị revoked,
When node attempts to execute,
Then action fails với: "Authentication failed. Please reconnect your {provider} account.",
And workflow execution được mark là failed

**AC.02. Model not available for the connected account**
Given model được chọn không khả dụng (account không có quyền dùng, hoặc model đã deprecated/removed ở provider),
When node runs,
Then action surfaces: "Model {model_name} is not available. Please select another model.",
And workflow execution được mark là failed

**AC.03. Content policy violation**
Given Prompt hoặc Instructions vi phạm content policy của provider,
When provider trả về policy violation error,
Then action surfaces: "Content violates {provider} usage policies. Please modify your input.",
And workflow execution được mark là failed

**AC.04. Token limit exceeded for model**
Given combined Prompt + Instructions vượt quá token limit của model,
When provider trả về token limit error,
Then action surfaces: "Input exceeds token limit for model {model_name}. Please shorten your input.",
And workflow execution được mark là failed

**AC.05. Provider unavailable hoặc timeout**
Given provider API down hoặc request timeout,
When error xảy ra,
Then action surfaces: "{Provider} is currently unavailable. Please try again later." (unavailable) hoặc "AI Agent timed out. Please try again or use shorter input." (timeout),
And workflow execution được mark là failed

**Note:** Không retry tại node-level. Chỉ dùng cơ chế retry node của hệ thống.

---

## Out of Scope

- **Tools / MCP integration** — sẽ làm ở Sprint 2 (Jira, Gmail, GitHub, MCP servers, custom tools).
- **Session ID / Memory** — node Sprint 1 stateless. Multi-turn conversation defer sang sprint sau theo standard MART-1885 nếu áp dụng.
- **Streaming response** — Sprint 1 chỉ trả về single response sau khi provider hoàn tất generation.
- **Custom provider / Bring-your-own model** — chỉ support 4 providers list trong AC.01 Story 1.
- **Function calling / Structured output schema** — sẽ cân nhắc khi làm Tools ở Sprint 2.
- **Multi-modal input** (image, audio, file attached vào prompt) — không nằm trong scope Sprint 1.
- **Cost / token quota tracking ở UI** — token usage trả về output để downstream tự xử lý; chưa có dashboard riêng.
- **Cancel running execution from UI** — defer.

---

## Inputs

| Field | Type | Required | Default | Manual Input | Expression | Description |
|---|---|---|---|---|---|---|
| AI Provider | Dropdown | Yes | `AI` | No | No | Options: xem AC.01 Story 1. |
| Account | Connected Account | Conditional | None | No | No | Required khi AI Provider ≠ `AI`. Ẩn khi AI Provider = `AI`. Dùng lại logic Connected Account hiện có. |
| Model | Dropdown | Yes | Default model của provider | No | No | Model list hard-code BE theo provider. Luôn có default value → không có validation `Model is required`. |
| Prompt | Textarea | Yes | None | Yes | Yes | Max 8,000 ký tự. Variable picker (`+`) chèn expression. |
| Instructions | Textarea | No | Empty | Yes | No | Max 4,000 ký tự. Optional. Không support expression. |

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| success | boolean | Whether action succeeded | No |
| action | string | `"run_ai_agent"` | No |
| response | string | Text response từ AI provider | No |
| model_used | string | Actual model được dùng để generate response | No |
| tokens | object | `{ input: number, output: number, total: number }`. Trả về `null` cho từng field nếu provider không cung cấp | Yes |
| timestamp | string | Thời điểm node hoàn tất execution theo ISO 8601 | No |

---

## Business Rules

1. AI Provider luôn có default `AI` — node hoạt động ngay cả khi user chưa cấu hình gì thêm (trừ Prompt).
2. Khi AI Provider = `AI`, không yêu cầu Account; quota tính vào workspace, không bill qua user's account.
3. Khi AI Provider ≠ `AI`, Account là required và dùng lại logic Connected Account hiện có của hệ thống.
4. Model list hard-code BE theo từng provider, mỗi provider có 1 default model.
5. Đổi AI Provider luôn reset Account về empty và load lại Model list của provider mới (kèm default model mới).
6. Model list load độc lập với Account state — user có thể xem Model list ngay khi chọn provider.
7. Prompt là required, max 8,000 ký tự, support cả manual input và expression.
8. Instructions là optional, max 4,000 ký tự, chỉ support manual input.
9. Khi Instructions để trống, request body gửi tới provider không include system role.
10. Node stateless — mỗi run độc lập, không lưu context giữa các run.
11. Test step gọi provider thật để verify config trước khi save.
12. Không retry tại node-level. Chỉ dùng cơ chế retry node của hệ thống.
13. Token usage field naming align với convention của existing AI nodes (Ask ChatGPT, Generate Content, Ask Claude). Confirm chính xác field names với Engineering trước implementation.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| Account empty (Provider ≠ AI) | Config-time, real-time | `Account is required.` |
| Prompt empty | Config-time, real-time | `Prompt is required.` |
| Prompt > 8,000 ký tự | Config-time, real-time | `Prompt must be 8,000 characters or less.` |
| Instructions > 4,000 ký tự | Config-time, real-time | `Instructions must be 4,000 characters or less.` |
| Model list empty từ BE | Config-time, khi load Model | `No models available. Please check your connection or permissions.` |
| Prompt resolve về empty | Runtime | `Prompt resolved to an empty value. Check your variable mapping.` |
| Prompt resolve vượt 8,000 ký tự | Runtime | `Prompt must be 8,000 characters or less.` |
| OAuth token expired hoặc revoked | Runtime | `Authentication failed. Please reconnect your {provider} account.` |
| Selected account no longer exists | Runtime | `Selected connection no longer exists. Please update your workflow configuration.` |
| Model không khả dụng (no permission hoặc deprecated/removed) | Runtime | `Model {model_name} is not available. Please select another model.` |
| Content policy violation | Runtime | `Content violates {provider} usage policies. Please modify your input.` |
| Token limit exceeded for model | Runtime | `Input exceeds token limit for model {model_name}. Please shorten your input.` |
| Provider unavailable | Runtime | `{Provider} is currently unavailable. Please try again later.` |
| Request timeout | Runtime | `AI Agent timed out. Please try again or use shorter input.` |
| Network connection failed | Runtime | `Failed to connect to {provider}. Please check your internet connection.` |

---

## Dependencies

**Between Stories:**
- Story 1 (Connect Provider/Account/Model) phải complete trước khi các stories khác có thể configure.
- Story 5 (Error Handling) là cross-cutting concern áp dụng cho tất cả stories.

**External Dependencies:**
- **Connected Account flow** — tái sử dụng cho ChatGPT, Claude, Gemini.
- **OpenAI API**, **Anthropic Claude API**, **Google Gemini API** — chat/completion endpoint của từng provider.
- **AI Internal service** — internal service của công ty (default model: Qwen 3.5).
