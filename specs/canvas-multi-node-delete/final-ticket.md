# User Story - Canvas - Add Delete to Multi-node Context Menu

## Epic Summary

**Epic Name:** Canvas – Add Delete to Multi-node Context Menu

**Goal:**
Bổ sung option "Delete" vào context menu khi user bôi đen nhiều node (≥ 2) trên canvas, giúp workflow builder xóa nhiều node cùng lúc mà không cần xóa từng node riêng lẻ.

**Context:**
Standalone UX improvement cho Canvas. Hiện tại, context menu khi chọn một node đơn đã có option "Delete" (phím tắt `Del`). Tuy nhiên, khi user chọn nhiều node và chuột phải, option này không xuất hiện — khiến user phải xóa từng node một. Ticket này bổ sung behavior còn thiếu để đồng nhất UX giữa single-node và multi-node selection. Hệ thống đã có cơ chế undo nên không cần confirmation dialog.

**Expected UI Fields:**
- Context menu item: "Delete" (kèm shortcut `Del`) — xuất hiện khi ≥ 2 node đang được chọn

---

## User Stories

### Story 1: Delete multiple selected nodes via context menu

**Story:** As an automation workflow builder, I want to delete all selected nodes at once via the right-click context menu, so that I can clean up my canvas faster without removing nodes one by one.

**Priority:** Must-have

**Acceptance Criteria:**

**AC.01. Delete option appears in multi-node context menu**
Given user đang ở canvas,
When user chọn ≥ 2 nodes và chuột phải,
Then context menu hiển thị option "Delete" với phím tắt `Del`,
And option "Delete" nằm ở vị trí cuối cùng của context menu (consistent với single-node context menu)

**AC.02. Delete removes all selected nodes immediately**
Given user đã chọn ≥ 2 nodes,
When user click "Delete" trong context menu hoặc nhấn phím `Del`,
Then tất cả các nodes đang được chọn bị xóa khỏi canvas ngay lập tức,
And các connections (edges) liên quan đến những nodes đó cũng bị xóa theo

**AC.03. Delete action is undoable**
Given user vừa xóa nhiều nodes,
When user thực hiện Undo (Cmd+Z / Ctrl+Z),
Then tất cả các nodes và connections vừa bị xóa được khôi phục về đúng vị trí ban đầu

---

## Out of Scope

- **Confirmation dialog** — hệ thống đã có undo nên không cần xác nhận trước khi xóa.
- **Locked/active node behavior** — không có concept locked node; không cần xử lý riêng.
- **Bulk delete từ sidebar / list view** — chỉ áp dụng cho canvas context menu.
