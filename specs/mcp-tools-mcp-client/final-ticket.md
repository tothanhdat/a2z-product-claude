# Ticket Title: Action - MCP Tools - MCP Client

## Epic Summary

**Epic Name:** MCP Tools – MCP Client

**Goal:**
Cho phép automation workflow builder kết nối tới một remote MCP server và execute một Tool được server hỗ trợ, để tích hợp workflow với các platform có hỗ trợ MCP mà không cần xây integration riêng cho từng platform trong sprint này.

**Context:**
User cấu hình connection bằng Server URL và Authentication mode, sau đó hệ thống load danh sách Tools từ MCP server để user chọn và execute trong Workflow. Authentication mặc định là OAuth; user có thể đổi sang Bearer Token hoặc None. Sprint này chỉ hỗ trợ MCP Tools; không hỗ trợ MCP Resources, Prompts, local MCP server, nested object/array input schema, raw result output, Refresh Tools, hoặc destructive warning.

**Expected UI Fields:**
* Account *
* Connection Name * — when creating new connection
* Server URL * — when creating new connection
* Authentication * — when creating new connection
* Bearer Token * — conditional
* Tool *
* Dynamic Tool Inputs

---

## User Stories

### Story 1: Connect accountt

**Story:** As an automation workflow builder, I want to connect an MCP Tools account, so that the MCP Client action runs with the correct MCP server connection.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Account * is required — cannot continue if not selected**
Given the Connect step is open,
When the "Account *" field has no account selected,
Then the "Continue" button is disabled,
And if the user clicks "Continue", a validation error is shown: "Account is required."

**AC.02. User selects an existing account**
Given the user clicks the "Select" button,
When existing MCP Tools accounts are available,
Then the system displays the available accounts,
And the user can select one account for this action.

**AC.03. User can create a new MCP Tools account**
Given the user clicks the "Select" button,
When the user chooses to create a new account,
Then the system displays the connection setup fields: "Connection Name *", "Server URL *", and "Authentication *".

**AC.04. Selected account displays connection name for confirmation**
Given the user has selected an MCP Tools account,
Then the "Account" field displays the selected connection name,
And the user can switch accounts by clicking "Select" again.

**Edge Cases:**
* User changes account after selecting a Tool → the selected Tool and all Dynamic Tool Inputs are cleared.

---

### Story 2: Create MCP server connection

**Story:** As an automation workflow builder, I want to create a connection to a remote MCP server, so that I can use Tools exposed by that server in my workflow.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Connection Name is required**
Given the user is creating a new MCP Tools account,
When "Connection Name *" is empty and user clicks Continue or Save,
Then the system shows validation error: "Connection Name is required."

**AC.02. Connection Name must not exceed 256 characters**
Given the user is creating or editing a connection,
When "Connection Name" exceeds 256 characters,
Then the system shows validation error: "Connection Name must be 256 characters or less."

**AC.03. Server URL is required**
Given the user is creating a new MCP Tools account,
When "Server URL *" is empty and user clicks Continue or Save,
Then the system shows validation error: "Server URL is required."

**AC.04. Server URL must be valid**
Given the user enters Server URL manually,
When the value does not start with `http://` or `https://`,
Then the system shows validation error: "Please enter a valid URL starting with http:// or https://."

**AC.05. Authentication defaults to OAuth**
Given the user is creating a new MCP Tools account,
When the connection setup form is displayed,
Then "Authentication *" defaults to "OAuth".

**AC.06. Authentication supports three options**
Given the user opens "Authentication *" dropdown,
When options are displayed,
Then the system shows:

| UI Label     | Notes                       |
| ------------ | --------------------------- |
| OAuth        | Default option              |
| Bearer Token | Requires Bearer Token field |
| None         | Allowed in production       |

**AC.07. Bearer Token field appears only when Authentication = Bearer Token**
Given the user selects "Bearer Token" in "Authentication *",
When the UI updates,
Then the system displays "Bearer Token *",
And the field is required.

**AC.08. Bearer Token is required when Authentication is Bearer Token**
Given "Authentication *" = "Bearer Token",
And "Bearer Token *" is empty,
When the user clicks Continue or Save,
Then the system shows validation error: "Bearer Token is required when Authentication is Bearer Token."

**AC.09. Bearer Token field is hidden and ignored for other authentication modes**
Given "Authentication *" is "OAuth" or "None",
When the connection is saved,
Then "Bearer Token" is not validated,
And any previously entered Bearer Token value is not included in the connection payload.

**AC.10. OAuth redirects user to authorization page**
Given the user selects "OAuth" in "Authentication *",
When the user continues connection setup,
Then the user is redirected to the MCP server's authorization page to approve access,
And after the user approves, the system saves the connection automatically and closes the setup form.

**AC.11. Connection is saved only after server validation succeeds**
Given the user provides Server URL and Authentication configuration,
When the user saves the connection,
Then the system confirms the server is reachable and exposes valid Tools,
And the connection is saved only if validation succeeds.

**Edge Cases:**
* Server URL is reachable but does not expose a valid MCP server → connection is not saved and user sees: "MCP server not found. Please check the Server URL."
* Server timeout during connection test → user sees: "MCP Client timed out. Please try again."
* OAuth authorization is rejected by user → connection is not saved.

---

### Story 3: Select MCP Tool

**Story:** As an automation workflow builder, I want to select a Tool exposed by the connected MCP server, so that the workflow can execute the correct operation.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Tool is required**
Given the Configure step is open,
When "Tool *" is empty and user clicks Continue,
Then the system shows validation error: "Tool is required."

**AC.02. Tool dropdown loads from selected MCP server**
Given the user has selected a valid MCP Tools account,
When the Configure step is opened,
Then the system loads available Tools from the connected MCP server,
And displays them in the "Tool *" dropdown.

**AC.03. Tool dropdown displays business-readable labels**
Given Tools are loaded from the MCP server,
When options are displayed,
Then the dropdown displays the Tool name.
If the Tool description is available, it is displayed as subtext below the Tool name. If not, only the Tool name is shown.

**AC.04. Empty Tool list displays empty state**
Given the selected MCP server returns no available Tools,
When the user opens "Tool *",
Then the system shows an empty state message: "No tools found for this MCP server.",
And the user cannot continue until a Tool is available.

**AC.05. Changing Account clears selected Tool**
Given the user has selected a Tool,
When the user changes the Account in the Connect step,
Then the system clears the selected Tool,
And clears all Dynamic Tool Inputs.

**AC.06. Changing Tool clears Dynamic Tool Inputs**
Given the user has selected a Tool and entered values into Dynamic Tool Inputs,
When the user selects a different Tool,
Then the system clears all Dynamic Tool Inputs from the previous Tool,
And renders the input fields for the newly selected Tool.

**AC.07. Tool does not support expression**
Given "Tool *" is a dropdown,
When the user configures the field,
Then the field does not support expression or Mapped Variable.

**Edge Cases:**
* Tool list cannot be loaded because authentication fails → user sees: "Authentication failed. Please reconnect your MCP Tools account."
* Tool list cannot be loaded because server is unavailable → user sees an error message and must reload manually.

---

### Story 4: Configure dynamic Tool Inputs

**Story:** As an automation workflow builder, I want the Configure step to show fields required by the selected Tool, so that I can provide the right input values before execution.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Dynamic fields render from selected Tool**
Given the user selects a Tool,
Then the system displays the input fields supported by that Tool.

**AC.02. Required Tool Inputs are marked as required**
Given the selected Tool marks an input as required,
When the input field is displayed,
Then the field label shows an asterisk `*`,
And the field must be filled before the user can continue.

**AC.03. Optional Tool Inputs are not required**
Given the selected Tool marks an input as optional,
When the input field is empty,
Then the user can continue without entering a value.

**AC.04. Dynamic Tool Inputs support manual input and expression**
Given a Dynamic Tool Input is displayed,
Then the user can enter a static value or map a value from an upstream step.

**AC.05. Required Dynamic Tool Input validates empty value at config-time**
Given a required Dynamic Tool Input is empty,
When the user clicks Continue,
Then the system shows validation error: "{Tool Input Label} is required."

**AC.06. Expression passes config-time required validation**
Given a required Dynamic Tool Input contains an expression,
When the user clicks Continue,
Then the field passes config-time validation,
And the value is validated at runtime after the expression resolves.

**AC.07. Tools with unsupported input types are handled gracefully**
Given the selected Tool has nested object or array input fields,
When the user selects that Tool,
Then optional nested fields are silently skipped and the user can still configure supported fields,
And if the Tool has required nested fields, the Tool cannot be used — an error is shown inline below the Tool field: "This tool contains unsupported required fields and cannot be used in this version.", and the user must select a different Tool.
If all Tools on the server have required nested fields, the user cannot use MCP Client with this server in sprint này.

---

### Story 5: Execute MCP Tool and return output

**Story:** As an automation workflow builder, I want the action to execute the selected MCP Tool and return a structured Output Payload, so that downstream steps can use the result.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Empty resolved value fails runtime validation**
Given a required Dynamic Tool Input uses expression,
When the resolved value is empty, null, or whitespace only,
Then the action fails with: "{Tool Input Label} resolved to an empty value. Check your variable mapping."

**AC.02. Invalid resolved value is handled by server error**
Given a Dynamic Tool Input resolves to a value the MCP server rejects,
When the action runs,
Then the action fails and surfaces the server error message to the user.

**AC.03. Output Payload is returned on success**
Given the MCP Tool executes successfully,
When the execution completes,
Then the action returns Output Payload gồm các fields tại mục Output.

**AC.04. Tool-level error result is treated as failed execution**
Given the MCP server returns a Tool-level error result,
When the action receives the response,
Then the action fails,
And the user sees: "MCP Tool execution failed. Check the tool input values and try again."

**Edge Cases:**
* MCP server returns empty result → action succeeds with `result = null`.

---

### Story 6: Handle authentication and permission errors

**Story:** As an automation workflow builder, I want clear and actionable error messages when authentication, permission, or server issues occur, so that I can identify and fix the problem.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Authentication fails**
Given the authentication token (OAuth or Bearer Token) has expired, been revoked, or is invalid,
When the action attempts to run,
Then the action fails with: "Authentication failed. Please reconnect your MCP Tools account."

**AC.02. MCP server unavailable or timeout**
Given the MCP server is unavailable or does not respond in time,
When the action attempts to run,
Then the action follows system retry logic,
And if retries fail, the action fails with: "MCP Client timed out. Please try again."

**AC.03. Tool not found at runtime**
Given the selected Tool no longer exists on the MCP server,
When the action runs,
Then the action fails with: "Tool not found. It may have been deleted or moved."

---

## Out of Scope

* Dedicated Jira MCP action.
* Dedicated GitHub MCP action.
* MCP Resources.
* MCP Prompts.
* Local MCP server.
* Nested object or array input rendering.
* Refresh Tools button.
* raw_result output.
* Destructive Tool warning for delete/remove/archive tools.
* Connected Account sharing behavior.
* Tool-level permission configuration in A2Z.
* Multi-tool execution in one action.
* AI-based Tool selection.
* Server registry or MCP marketplace.
* Custom headers beyond Bearer Token.

---

## Inputs

### Connect Step

| Field           | Type              | Required    | Default | Manual Input | Expression | Description                                                   |
| --------------- | ----------------- | ----------- | ------- | ------------ | ---------- | ------------------------------------------------------------- |
| Account         | Connected Account | Yes         | None    | No           | No         | Dùng lại logic Connected Account hiện có của hệ thống.        |
| Connection Name | Text input        | Conditional | None    | Yes          | No         | Required khi tạo connection mới. Max 256 characters.          |
| Server URL      | Text input        | Conditional | None    | Yes          | No         | Required khi tạo connection mới. URL của remote MCP server.   |
| Authentication  | Dropdown          | Conditional | OAuth   | No           | No         | Required khi tạo connection mới. Options: see Story 2, AC.06. |
| Bearer Token    | Secret text input | Conditional | None    | Yes          | No         | Required khi Authentication = Bearer Token. Hidden otherwise. |

> Note: Action field không hiển thị trong sprint này vì chỉ có một action (MCP Client). Field sẽ được bổ sung khi có thêm action trong sprint sau.

### Configure Step

| Field               | Type           | Required               | Default | Manual Input | Expression | Description                                                                                 |
| ------------------- | -------------- | ---------------------- | ------- | ------------ | ---------- | ------------------------------------------------------------------------------------------- |
| Tool                | Dropdown       | Yes                    | None    | No           | No         | Tool list loaded from selected MCP server.                                                  |
| Dynamic Tool Inputs | Dynamic fields | Depends on Tool schema | None    | Yes          | Yes        | Rendered from selected Tool schema. Nested object/array fields are skipped in sprint này.   |

---

## Outputs

| Field             | Type                                              | Description                                                                                                           | Nullable |
| ----------------- | ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------- |
| success           | boolean                                           | Whether the MCP Tool execution succeeded.                                                                             | No       |
| action            | string                                            | Fixed value: `mcp_client`.                                                                                            | No       |
| server_url        | string                                            | MCP server URL used by the selected connection.                                                                       | No       |
| tool_name         | string                                            | MCP Tool name executed.                                                                                               | No       |
| tool_display_name | string                                            | Business-readable Tool label if available.                                                                            | Yes      |
| result            | array / object / string / number / boolean / null | Normalized result returned by the MCP Tool. Nullable nếu Tool không trả data. Downstream steps có thể map field này. | Yes      |
| executed_at       | string                                            | Execution timestamp in ISO 8601.                                                                                      | No       |

---

## Business Rules

**1. Action field không hiển thị trong sprint này.**
Reason: Sprint này chỉ có một action (MCP Client). Field sẽ được thêm khi triển khai action khác.

**2. Authentication supports OAuth, Bearer Token, and None.**
Reason: MCP servers có thể dùng nhiều authentication modes khác nhau. Xem Story 2, AC.06 cho danh sách options.

**3. OAuth là default Authentication mode.**
Reason: Product decision, phù hợp với MCP servers cần user authorization.
Exception: User có thể đổi sang Bearer Token hoặc None.

**4. Authentication None được phép sử dụng ở production.**
Reason: Một số MCP servers public hoặc được bảo vệ bởi network layer ngoài platform.

**5. Bearer Token chỉ hiển thị khi Authentication = Bearer Token.**
Reason: Tránh user nhập secret không cần thiết.

**6. Tool list được load từ MCP server sau khi Account được chọn.**
Reason: Mỗi MCP server có danh sách Tools khác nhau.

**7. User phải chọn Tool tại config-time.**
Reason: Input fields của Tool cần được xác định trước để user có thể điền giá trị.

**8. Khi đổi Account, hệ thống clear selected Tool và Dynamic Tool Inputs.**
Reason: Tool list phụ thuộc vào MCP server của selected Account.

**9. Khi đổi Tool, hệ thống clear Dynamic Tool Inputs cũ.**
Reason: Mỗi Tool có input schema khác nhau; giữ value cũ có thể gây execution sai.

**10. Dynamic Tool Inputs support manual input và expression.**
Reason: Workflow cần map dữ liệu từ upstream step vào Tool execution.

**11. Nested object và array input schema bị skip trong sprint này.**
Reason: Product decision để giảm complexity. Nếu nested field là required, Tool không thể dùng trong sprint này.

**12. Refresh Tools button không nằm trong scope.**
Reason: Product decision cho sprint này.

**13. raw_result không được include trong Output Payload.**
Reason: Product decision cho sprint này. Có thể mở rộng ở phase sau.

**14. Destructive Tool warning không nằm trong scope.**
Reason: Product decision defer sang phase sau.

**15. Output Payload phải là structured output phục vụ downstream steps.**
Reason: Downstream steps cần stable fields để map expression.
Exception: `result` có thể nullable nếu MCP Tool execute thành công nhưng không trả data.

**16. Server unavailable và timeout dùng chung một error message.**
Reason: Từ góc độ user, cả hai cases đều yêu cầu cùng hành động (thử lại sau).

---

## Error Handling

| Error Scenario                      | When Detected | User-facing Message                                                                  |
| ----------------------------------- | ------------- | ------------------------------------------------------------------------------------ |
| Missing Account                     | Config-time   | `Account is required.`                                                               |
| Missing Connection Name             | Config-time   | `Connection Name is required.`                                                       |
| Connection Name too long            | Config-time   | `Connection Name must be 256 characters or less.`                                    |
| Missing Server URL                  | Config-time   | `Server URL is required.`                                                            |
| Invalid Server URL                  | Config-time   | `Please enter a valid URL starting with http:// or https://.`                        |
| Missing Authentication              | Config-time   | `Authentication is required.`                                                        |
| Missing Bearer Token                | Config-time   | `Bearer Token is required when Authentication is Bearer Token.`                      |
| Missing Tool                        | Config-time   | `Tool is required.`                                                                  |
| Unsupported required nested field   | Config-time   | `This tool contains unsupported required fields and cannot be used in this version.` |
| Authentication failed               | Runtime       | `Authentication failed. Please reconnect your MCP Tools account.`                    |
| Tool not found                      | Runtime       | `Tool not found. It may have been deleted or moved.`                                 |
| Empty resolved Dynamic Tool Input   | Runtime       | `{Tool Input Label} resolved to an empty value. Check your variable mapping.`        |
| MCP server not found or invalid     | Runtime       | `MCP server not found. Please check the Server URL.`                                 |
| MCP server timeout or unavailable   | Runtime       | `MCP Client timed out. Please try again.`                                            |
| Tool execution failed               | Runtime       | `MCP Tool execution failed. Check the tool input values and try again.`              |

---

## Assumptions

A1. MCP Client là Action thuộc App `MCP Tools`.
A2. Sprint này chỉ có một Action là `MCP Client`; Action field không hiển thị trên UI cho đến khi có thêm action trong sprint sau.
A3. Authentication None được phép dùng ở production.
A4. raw_result không được include trong Output Payload của sprint này.

---

## Open Questions

None.

---

## Dependencies

**Between Stories:**
* Story 1 và Story 2 phải hoàn tất trước khi user có thể Configure Tool.
* Story 3 phải hoàn tất trước Story 4 vì Dynamic Tool Inputs phụ thuộc selected Tool.
* Story 5 phụ thuộc Story 3 và Story 4.
* Story 6 là cross-cutting concern áp dụng cho toàn bộ action.

**External Dependencies:**
* Backend MCP OAuth support.
* Existing Connected Account behavior.
* Existing runtime expression resolution.
* Existing system retry logic.
