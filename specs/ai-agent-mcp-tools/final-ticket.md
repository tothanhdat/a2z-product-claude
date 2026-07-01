# Ticket Title: Action - AI Agent - MCP Tools Integration (Sprint 2)

## Epic Summary

**Epic Name:** AI – AI Agent Tools (MCP)

**Goal:**
Cho phép automation workflow builder gắn một hoặc nhiều MCP server connections vào AI Agent node, để AI tự quyết định gọi Tool nào dựa trên Prompt — mở rộng AI Agent từ text-only response thành agent có khả năng tương tác với external services thông qua MCP protocol.

**Context:**
Tiếp nối ticket AI Agent Core (Sprint trước) — node đã có Connect step (AI Provider, Account, Model), Configure step (Prompt), và Advanced step (Instructions). Ticket này bổ sung section `Tools` trong Configure step, cho phép user add các tool types vào AI Agent thông qua nút Add → dialog chọn tool type. Sprint này chỉ hỗ trợ MCP Client Tool — user chọn MCP Client Tool từ dialog, đi qua Connect tab của MCP Client Tool để chọn account có sẵn hoặc tạo connection mới. Hệ thống load tất cả Tools từ các MCP servers đã connect và cung cấp cho AI provider. AI tự quyết định gọi Tool nào, có thể gọi nhiều Tools trong một run (multi-step reasoning).

**Expected UI Fields:**
- Tools (Configure) — với nút Add
- Connection Name * (New Connection — trong MCP Client Tool Connect tab)
- MCP Server URL * (New Connection — trong MCP Client Tool Connect tab)
- Authentication * (New Connection — trong MCP Client Tool Connect tab)

---

## User Stories

### Story 1: Add and manage Tools in AI Agent

**Story:** As an automation workflow builder, I want to add one or more Tools to AI Agent through a tool selection dialog, so that AI can access and use those Tools during execution.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Tools section hiển thị trong Configure step**
Given user mở Configure step của AI Agent,
When UI render,
Then section `Tools` hiển thị sau field `Prompt` với nút `Add`,
And section là optional — user có thể Continue mà không add tool nào

**AC.02. Add button mở dialog chọn tool type**
Given user click nút `Add` trong section `Tools`,
When dialog mở,
Then hệ thống hiển thị danh sách các tool types được hỗ trợ,
And hiện tại chỉ có `MCP Client Tool` trong danh sách

**AC.03. Chọn MCP Client Tool chuyển đến Connect tab**
Given user chọn `MCP Client Tool` từ dialog,
When hệ thống mở MCP Client Tool,
Then user thấy Connect tab với field `Account *`,
And user có thể chọn từ danh sách MCP accounts đã connect hoặc tạo connection mới (theo Story 2)

**AC.04. Tool entry hiển thị sau khi connect thành công**
Given user đã connect thành công một MCP account,
When hệ thống quay về Configure step của AI Agent,
Then tool entry hiển thị trong danh sách `Tools` với dòng 1: `MCP Client`, dòng 2: `{Connection Name}`

**AC.05. User có thể add nhiều tools**
Given user đã add một hoặc nhiều tools,
When user click nút `Add` lần nữa,
Then hệ thống cho phép add thêm tools,
And mỗi tool đại diện cho một MCP server riêng biệt

**AC.06. Xóa tool khỏi danh sách**
Given user đã add tools,
When user xóa một tool khỏi danh sách (qua menu ⋮),
Then tool entry bị loại bỏ khỏi danh sách `Tools` và khỏi canvas

**AC.07. Danh sách Tools và canvas Tool nodes đồng bộ**
Given user add hoặc xóa tool trong Configure step,
When danh sách `Tools` thay đổi,
Then canvas Tool nodes cập nhật tương ứng — thêm hoặc xóa Tool node nối với AI Agent,
And ngược lại, xóa Tool node trên canvas cũng cập nhật danh sách `Tools`

**AC.08. Không có tool nào — AI Agent hoạt động như Sprint trước**
Given section `Tools` trống,
When AI Agent execute,
Then node hoạt động như Sprint trước — chỉ trả text response dựa trên Prompt và Instructions,
And output `tool_calls` là empty array

**AC.09. Prevent duplicate tool connection**
Given user đã add một MCP Client Tool với account cụ thể,
When user cố add cùng account đó lần nữa,
Then account đã add không hiển thị trong danh sách chọn

**AC.10. Toast thông báo khi connect tool**
Given user hoàn tất flow connect MCP Client Tool,
When connection saved thành công,
Then hệ thống hiển thị toast: "Connection saved successfully.",
And nếu connection thất bại không rõ nguyên nhân, hệ thống hiển thị toast: "Failed to connect. Please try again."

**AC.11. Status icon sau execution**
Given user đã run test hoặc workflow execution,
When kết quả function calling trả về,
Then tool entry hiển thị status icon: ✅ xanh nếu tool call thành công, ⚠ vàng nếu tool call thất bại,
And trước khi execution lần đầu, tool entry không hiển thị status icon

**AC.12. Click Tool node trên canvas mở Configure step**
Given user nhìn thấy Tool node trên canvas,
When user click vào Tool node đó,
Then hệ thống mở Configure step của AI Agent với tool đó được highlight trong danh sách `Tools`

**Edge Cases:**
- MCP server không reachable khi load Tools → hiển thị warning inline cho tool entry đó: "Failed to load tools from {Connection Name}. Please check the connection." User vẫn có thể Continue với các tools khác.
- Tất cả tools đều fail load → section hiển thị warning nhưng không block Continue — AI Agent vẫn có thể chạy without tools (behavior Sprint trước).
- User đổi AI Provider (Connect step) → danh sách Tools giữ nguyên (Tools độc lập với AI provider).
- Chỉ khi connect thành công tới MCP Server thì tool mới hiển thị trong danh sách Tools.

---

### Story 2: Connect MCP Client Tool via Add flow

**Story:** As an automation workflow builder, I want to connect a new MCP server through MCP Client Tool's Connect tab within AI Agent, so that I can quickly add new MCP servers without leaving the node configuration.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Form tạo connection hiển thị các fields cần thiết**
Given user chọn tạo connection mới từ Account picker trong MCP Client Tool Connect tab,
When form "Connect a new account" mở,
Then hệ thống hiển thị các fields: `Connection Name *`, `MCP Server URL *`, `Authentication *`

**AC.02. Connection Name is required**
Given user đang tạo connection mới,
When `Connection Name *` để trống,
Then nút Connect bị disabled,
And inline validation error hiển thị ngay dưới field: "Connection Name is required."

**AC.03. Connection Name must not exceed 256 characters**
Given user đang tạo connection mới,
When `Connection Name` vượt quá 256 ký tự,
Then inline validation error hiển thị ngay dưới field: "Connection Name must be 256 characters or less."

**AC.04. MCP Server URL is required**
Given user đang tạo connection mới,
When `MCP Server URL *` để trống,
Then nút Connect bị disabled,
And inline validation error hiển thị ngay dưới field: "MCP Server URL is required."

**AC.05. MCP Server URL must be valid**
Given user nhập MCP Server URL,
When giá trị không bắt đầu bằng `http://` hoặc `https://`,
Then inline validation error hiển thị ngay dưới field: "Please enter a valid URL starting with http:// or https://."

**AC.06. Authentication defaults to None and supports three options**
Given form tạo connection hiển thị,
When field `Authentication *` được render,
Then default value là `None`,
And dropdown hiển thị:

| UI Label | Notes |
|---|---|
| None | Default option |
| OAuth MCP | |
| Bearer Token | Requires Bearer Token field |

**AC.07. Bearer Token field appears and is required only when Authentication = Bearer Token**
Given user chọn `Bearer Token` ở `Authentication *`,
When UI updates,
Then field `Bearer Token *` hiển thị và là required,
And khi `Bearer Token *` để trống, inline validation error hiển thị ngay: "Bearer Token is required.",
And khi Authentication là `None` hoặc `OAuth MCP`, field `Bearer Token` bị ẩn và không validate

**AC.08. OAuth MCP redirects user to authorization page**
Given user chọn `OAuth MCP` ở `Authentication *`,
When user tiến hành lưu connection,
Then user được redirect tới trang authorization của MCP server để approve access,
And sau khi user approve, hệ thống tiếp tục server validation (AC.09) trước khi lưu connection

**AC.09. Connection saved only after server validation succeeds**
Given user đã điền đầy đủ thông tin connection (hoặc đã hoàn tất OAuth approve),
When hệ thống thực hiện server validation,
Then hệ thống kiểm tra server reachable và có ít nhất một Tool available,
And connection được lưu chỉ khi cả hai điều kiện đạt,
And nếu OAuth succeed nhưng server validation fail → connection không lưu, hiển thị error tương ứng

**AC.10. New connection auto-added to Tools list**
Given connection mới được lưu thành công,
When MCP Client Tool Connect tab đóng lại,
Then tool entry mới tự động xuất hiện trong danh sách `Tools` của AI Agent (dòng 1: `MCP Client`, dòng 2: `{Connection Name}`),
And tool node tương ứng hiển thị trên canvas

**Edge Cases:**
- MCP Server URL reachable nhưng không expose valid MCP server → connection không được lưu, user thấy: "MCP server not found. Please check the MCP Server URL."
- Server timeout during connection test → user thấy: "MCP Client timed out. Please try again."
- OAuth MCP authorization bị user reject → connection không được lưu.
- Connection Name đã tồn tại trong danh sách → khi user nhấn nút Connect, hệ thống báo lỗi: "Connection Name already exists. Please use a different name."

---

### Story 3: AI Agent executes with MCP Tools and returns output

**Story:** As an automation workflow builder, I want AI Agent to automatically decide which MCP Tools to call based on Prompt, execute them in a multi-step loop, and return structured output including tool execution details, so that downstream steps can use both AI response and tool results.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. AI Agent sử dụng function calling khi có Tools**
Given AI Agent có một hoặc nhiều tools trong danh sách `Tools`,
When node execute,
Then hệ thống cung cấp tất cả available Tools từ các attached MCP servers cho AI provider,
And AI tự quyết định gọi Tool nào dựa trên Prompt và Instructions

**AC.02. AI Agent có thể gọi nhiều Tools trong một run**
Given AI Agent đang execute,
When AI quyết định cần gọi Tool,
Then hệ thống execute Tool call tới MCP server tương ứng,
And trả kết quả về cho AI để tiếp tục reasoning,
And AI có thể gọi thêm Tools khác nếu cần cho đến khi có final answer

**AC.03. Execution dừng khi AI trả final response**
Given AI Agent đang execute với tool calls,
When AI trả về final text response (không yêu cầu gọi thêm tool),
Then execution hoàn tất,
And action trả Output Payload gồm các fields tại mục Output

**AC.04. Tool với unsupported required nested input không available cho AI**
Given một Tool từ MCP server có required nested object hoặc array input fields,
When hệ thống load Tools cho AI Agent,
Then Tool đó không available cho AI Agent — AI không thể gọi tool đó

**AC.05. Tool với optional nested input vẫn available**
Given một Tool từ MCP server chỉ có optional nested object hoặc array input fields,
When hệ thống load Tools cho AI Agent,
Then Tool vẫn available nhưng các optional nested fields không được hỗ trợ

**AC.06. AI quyết định không gọi tool — hoạt động như Sprint trước**
Given AI Agent có tools trong danh sách `Tools`,
When AI nhận Prompt nhưng quyết định không cần gọi tool nào,
Then node trả text response bình thường,
And output `tool_calls` là empty array

**AC.07. Test step chỉ hiển thị AI response**
Given test execution completes,
When kết quả load trong Test panel,
Then panel hiển thị text response AI trả về, Model used, Latency, Token usage (giữ nguyên Sprint trước),
And Test step không hiển thị chi tiết từng tool call — chỉ Logs panel mới có view này

**AC.08. Logs hiển thị input/output từng Tool node**
Given workflow execution hoàn tất có tool calls,
When user mở Logs panel,
Then AI Agent hiển thị dưới dạng parent node với các child Tool nodes (mỗi tool đã gọi là một child node),
And user click vào từng child Tool node để xem INPUT (dữ liệu gửi tới tool) và OUTPUT (kết quả trả về từ tool)

**Edge Cases:**
- AI gọi Tool nhưng MCP server trả error → AI nhận error context và có thể quyết định gọi tool khác, hoặc trả response dựa trên thông tin có sẵn. Tool call được ghi nhận trong `tool_calls` với status `failed`.
- AI gọi cùng một Tool nhiều lần với input khác nhau → mỗi lần gọi đều được ghi nhận riêng trong `tool_calls`.
- Execution kéo dài do AI gọi nhiều tools → execution chịu system-level workflow execution timeout.

---

### Story 4: Handle tool-related authentication and runtime errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when MCP tool-related issues occur during AI Agent execution, so that I can quickly identify and resolve the root cause.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. MCP connection authentication fails during execution**
Given một MCP connection có authentication token expired hoặc invalid,
When AI Agent cố gọi Tool từ connection đó,
Then tool call được ghi nhận trong `tool_calls` với status `failed` và error: "Authentication failed for {Connection Name}.",
And AI nhận error context và tiếp tục reasoning với tools từ connections khác

**AC.02. MCP server unavailable hoặc timeout during execution**
Given MCP server không response hoặc timeout,
When AI Agent cố gọi Tool từ server đó,
Then tool call được ghi nhận trong `tool_calls` với status `failed` và error: "{Connection Name} timed out.",
And AI nhận error context và tiếp tục reasoning

**AC.03. Tool not found at runtime**
Given Tool đã tồn tại khi load nhưng bị xóa hoặc thay đổi trên MCP server,
When AI Agent cố gọi Tool đó,
Then tool call được ghi nhận với status `failed` và error: "Tool not found. It may have been deleted or moved.",
And AI tiếp tục reasoning

**AC.04. MCP Tool execution failed**
Given MCP server trả về Tool-level error result,
When AI Agent nhận response,
Then tool call được ghi nhận với status `failed` và error: "MCP Tool execution failed.",
And AI nhận error message và tiếp tục reasoning

**AC.05. All tool calls fail — AI still returns response**
Given tất cả tool calls trong execution đều fail,
When AI nhận lại tất cả error contexts,
Then AI vẫn trả final response dựa trên Prompt, Instructions, và thông tin có sẵn (không có tool results),
And output `success` vẫn là `true` nếu AI trả được response

**AC.06. AI Provider error (non-tool-related) follows Sprint trước**
Given lỗi xảy ra ở AI provider level (auth expired, timeout, content policy, token limit),
When error không liên quan đến MCP tools,
Then error handling theo Sprint trước AI Agent ticket — action fails với tương ứng error message

**Edge Cases:**
- Một trong nhiều MCP connections fail authentication, các connections khác vẫn hoạt động → AI vẫn dùng tools từ connections còn lại, `tool_calls` ghi nhận từng call riêng biệt.

---

## Out of Scope

- **Force/pin Tool cụ thể** — Sprint này AI hoàn toàn tự quyết. Defer sang sprint sau.
- **Streaming response khi có tool calls** — trả về single response sau khi hoàn tất toàn bộ reasoning loop.
- **Nested object/array input cho MCP Tools** — chỉ hỗ trợ primitive input types trong sprint này.
- **Tool execution history/log UI** — chưa có dashboard riêng cho tool call history.
- **MCP Resources và MCP Prompts** — chỉ hỗ trợ MCP Tools.
- **Custom tool definitions (non-MCP)** — chỉ hỗ trợ Tools từ MCP servers.
- **Parallel tool execution** — tool calls execute sequentially trong sprint này.
- **Tool result caching** — mỗi tool call luôn execute fresh.
- **Session ID / Memory** — node vẫn stateless, follow Sprint trước.
- **AI-based tool selection UI** (hiển thị AI đang "nghĩ" tool nào) — defer.
- **Max Iterations / tool call limit** — hệ thống không giới hạn số lần gọi tool. Execution chịu system-level workflow timeout.

---

## Outputs

| Field | Type | Description | Nullable |
|---|---|---|---|
| success | boolean | Whether action succeeded | No |
| action | string | Fixed value: `run_ai_agent` | No |
| ai_response | string | Final text response từ AI provider sau khi hoàn tất reasoning (có hoặc không có tool calls) | No |
| model_used | string | Actual model được dùng để generate response | No |
| tokens | number | Tổng tokens bao gồm tất cả reasoning turns + tool call context. Trả về `null` nếu provider không cung cấp. **Note:** cần align token counting với AI provider billing model. | Yes |
| tool_calls | array | Danh sách tool calls đã thực hiện. Mỗi item: `{ tool_name: string, server_url: string, connection_name: string, input: object, result: any, status: "success" \| "failed", error: string \| null, executed_at: string }`. Empty array nếu AI không gọi tool nào | No |
| timestamp | string | Execution timestamp ISO 8601 | No |

---

## Business Rules

1. Tools là optional — AI Agent hoạt động bình thường khi không có tool nào (behavior Sprint trước).
2. User có thể add nhiều tools. Mỗi tool (MCP connection) load Tools riêng, AI Agent nhìn thấy tất cả Tools từ mọi connections.
3. User add tool qua nút Add → chọn tool type (hiện tại chỉ có MCP Client Tool) → connect qua MCP Client Tool Connect tab.
4. Danh sách Tools trong Configure step và Tool nodes trên canvas phải đồng bộ.
5. Tools độc lập với AI Provider — đổi AI Provider không ảnh hưởng đến danh sách Tools.
6. AI tự quyết định gọi Tool nào dựa trên Prompt và Instructions — không có option force/pin Tool.
7. AI có thể gọi nhiều Tools trong một run (multi-step reasoning). Hệ thống không giới hạn số lần gọi tool — execution chịu system-level workflow execution timeout.
8. Tool calls execute sequentially — không parallel.
9. Nested object/array input schema từ MCP Tools không được hỗ trợ. Tools có required nested fields không available cho AI Agent.
10. Khi một tool call fail, AI nhận error context và tự quyết định hành động tiếp theo. Execution chỉ fail nếu AI provider-level error xảy ra.
11. Output `tool_calls` luôn present — empty array nếu không có tool call nào.
12. Connect step (AI Provider, Account, Model) và Prompt, Instructions giữ nguyên theo Sprint trước.
13. Không retry tool call tại node-level. Chỉ dùng cơ chế retry node của hệ thống cho toàn bộ execution.
14. Authentication None được phép sử dụng ở production (một số MCP servers public hoặc được bảo vệ bởi network layer ngoài platform).
15. Chỉ khi connect thành công tới MCP Server thì tool mới hiển thị trong danh sách Tools.
16. Status icon trên tool entry cập nhật sau mỗi lần test/execution: ✅ xanh = thành công, ⚠ vàng = thất bại.

---

## Error Handling

| Error Scenario | When Detected | User-facing Message |
|---|---|---|
| MCP server unreachable khi load tools | Config-time | `Failed to load tools from {Connection Name}. Please check the connection.` |
| Connection Name trống | Config-time | `Connection Name is required.` |
| Connection Name quá dài | Config-time | `Connection Name must be 256 characters or less.` |
| MCP Server URL trống | Config-time | `MCP Server URL is required.` |
| MCP Server URL không hợp lệ | Config-time | `Please enter a valid URL starting with http:// or https://.` |
| Bearer Token trống khi Auth = Bearer Token | Config-time | `Bearer Token is required.` |
| MCP server not found | Config-time | `MCP server not found. Please check the MCP Server URL.` |
| Connection Name đã tồn tại | Config-time | `Connection Name already exists. Please use a different name.` |
| Connection thất bại không rõ nguyên nhân | Config-time | Toast: `Failed to connect. Please try again.` |
| Connection test timeout | Config-time | `MCP Client timed out. Please try again.` |
| MCP connection auth failed (runtime tool call) | Runtime | Ghi trong `tool_calls`: `Authentication failed for {Connection Name}.` |
| MCP server timeout (runtime tool call) | Runtime | Ghi trong `tool_calls`: `{Connection Name} timed out.` |
| Tool not found (runtime) | Runtime | Ghi trong `tool_calls`: `Tool not found. It may have been deleted or moved.` |
| Tool execution failed (runtime) | Runtime | Ghi trong `tool_calls`: `MCP Tool execution failed.` |
| AI Provider auth failed | Runtime | Follow Sprint trước: `Authentication failed. Please reconnect your {provider} account.` |
| AI Provider timeout | Runtime | Follow Sprint trước: `AI Agent timed out. Please try again or use shorter input.` |
| Content policy violation | Runtime | Follow Sprint trước: `Content violates {provider} usage policies. Please modify your input.` |
| Token limit exceeded | Runtime | Follow Sprint trước: `Input exceeds token limit for model {model_name}. Please shorten your input.` |

