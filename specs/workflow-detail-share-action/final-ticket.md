# User Story – Workflow Detail – Add Share Action to Kebab Menu

## Epic Summary

**Epic Name:** Workflow Detail – Add Share Action to Kebab Menu

**Goal:**
Cho phép Owner/Admin share Workflow trực tiếp từ trang Workflow Detail mà không cần quay về Dashboard.

**Context:**
Tiếp nối MART-1534 (Share Workflow from Dashboard). Delta này bổ sung entry point Share vào kebab menu (⋮) trên Workflow Detail page header. Toàn bộ logic phân quyền, Share Settings popup, invitation, và remove user hoàn toàn theo MART-1534 — không có thay đổi.

**Expected UI Components:**
- Share action trong kebab menu (⋮) trên Workflow Detail page header (cùng vị trí với Edit, Duplicate, Create template, Delete)

---

## User Stories

### Story 1: Access Share Settings from Workflow Detail page

**Story:** As an Owner or Admin, I want to access Share Settings directly from the Workflow Detail page, so that I can manage sharing without returning to the Dashboard.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Owner/Admin sees Share action in kebab menu**
Given user có role Owner hoặc Admin,
When user mở kebab menu (⋮) trên Workflow Detail page header,
Then hệ thống hiển thị action Share trong menu, cùng với các action Edit, Duplicate, Create template, Delete.

**AC.02. Member does not see Share action in kebab menu**
Given user có role Member,
When user mở kebab menu (⋮) trên Workflow Detail page header,
Then hệ thống không hiển thị action Share.

**AC.03. Share opens Share Settings popup**
Given Owner hoặc Admin click action Share trong kebab menu,
When hệ thống xử lý,
Then Share Settings popup mở ra và hoạt động theo đúng behavior đã định nghĩa trong MART-1534.

---

## Out of Scope

- Thay đổi bất kỳ behavior nào của Share Settings popup — covered by MART-1534.
- Logic phân quyền theo role cho Share action — covered by MART-1534.
- Share entry point tại các vị trí khác trên Workflow Detail page (ngoài kebab menu).
