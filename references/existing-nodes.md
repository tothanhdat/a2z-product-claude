---
inclusion: manual
---

# Existing Nodes & Reusable Logic

## Mục đích

File này liệt kê tất cả nodes đã phát triển hoặc đang phát triển trên platform. Khi viết ticket mới, AI tham chiếu file này để:
- Biết logic nào đã tồn tại và có thể reuse (reference thay vì mô tả lại).
- Biết node nào cùng provider để reference patterns tương tự.
- Tránh viết ticket cho node đã Done.

**Status legend:** Done = đã ship | In progress = đang dev | Planning = đã plan chưa dev | Canceled = huỷ | Pending = chưa có kế hoạch cụ thể

---

## Schedule

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | Every X Minutes | Done | Recurring interval mode |
| Trigger | Every Day | Done | Recurring interval, Day of week/month/daily/specific time modes |
| Trigger | Every Month | — | — |
| Trigger | Every Hour | Done | Recurring interval mode |
| Trigger | Every Week | — | — |
| Trigger | Cron Expression | — | — |

---

## Webhook

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | Catch Webhook | Done | Instant webhook receive mechanism |
| Action | Return Response | Done | — |
| Action | Respond and Wait for Next Webhook | — | — |

---

## HTTP

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Action | Send Request | Done | HTTP request pattern |
| Action | Send an OAuth2 Request | — | OAuth2 request flow |

---

## Google Drive

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New File | Done | Scheduled polling, file detection |
| Trigger | New Folder | Done | Scheduled polling, folder detection |
| Action | Create New Folder | Done | Folder selector, Drive picker |
| Action | Upload File | Done | Binary file upload, folder selector |
| Action | Get File Information | Done | File lookup by ID |
| Action | Search | Done | File/folder search, query builder |
| Action | Save Document as PDF | Done | File conversion, folder selector |
| Action | Delete Permissions | Done | Permission management |
| Action | Move File | Done | Source/destination folder selector |
| Action | Trash File | Done | File ID input pattern |
| Action | Create New File | Done | Text file creation, folder selector |
| Action | Read File Content | Done | File content extraction |
| Action | List Files | Done | Folder listing, pagination |
| Action | Duplicate File | Done | File copy, new name input |
| Action | Update Permissions | Done | Permission role assignment |
| Action | Set Public Access | Done | Access level configuration |
| Action | Delete File | Done | File ID/URL input pattern, Add search file step |

---

## Google Sheets

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New or Updated Row | Done | Instant, spreadsheet/worksheet selector |
| Trigger | Updated Row | Done | Spreadsheet/worksheet selector |
| Trigger | New Row Added | Done | Instant, spreadsheet/worksheet selector |
| Trigger | New Spreadsheet | Done | Scheduled polling |
| Trigger | New Worksheet | Done | Scheduled polling, spreadsheet selector |
| Action | Insert Row | Done | Spreadsheet/worksheet selector, column mapping |
| Action | Insert Multiple Rows | Done | Expression-based dynamic value support |
| Action | Delete Row | Done | Row number input |
| Action | Find Rows | Done | Column name + search value pattern |
| Action | Create Worksheet | Done | Spreadsheet selector, title input |
| Action | Get Row | Done | Row number input |
| Action | Get Many Rows | Done | Sheet selection |
| Action | Find Worksheet(s) | Done | Title search |
| Action | Update Multiple Rows | Done | Batch row update |
| Action | Export Sheet | Done | CSV/TSV format selection |
| Action | Update Row | Done | Row overwrite pattern |
| Action | Create Spreadsheet | Done | Blank spreadsheet creation |
| Action | Clear Sheet | Done | Sheet clearing |
| Action | Get Next Row(s) | Done | Pagination/cursor pattern |
| Action | Find Spreadsheet(s) | Done | Name search |
| Action | Copy Worksheet | Done | Source/destination worksheet |
| Action | Create Spreadsheet Column | Done | Column addition |
| Action | Get Data Range | — | Range-based data retrieval |
| Action | Clear Row | Done | Row data clearing |

---

## Google Calendar

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New or Updated Event | Done | Scheduled polling, calendar selector, Expand Recurring Events logic |
| Trigger | Event Ends | Planning | Scheduled polling |
| Trigger | New Event Matching Search | — | Scheduled polling, search term filter |
| Trigger | New Calendar | Planning | Scheduled polling |
| Trigger | New Event | Planning | Instant webhook, calendar selector, push notification mechanism |
| Trigger | Event Start (Time Before) | — | Scheduled polling, time offset |
| Trigger | Event Cancelled | Planning | Scheduled polling |
| Action | Create Event | Done | Calendar selector, event fields (summary, start/end, attendees) |
| Action | Update Event | Done | Event ID input, event field update |
| Action | Create Quick Event | Done | Quick text-based event creation |
| Action | Get All Events | Done | Calendar selector, date range filter |
| Action | Delete Event | Done | Event ID input |
| Action | Get Event by ID | — | Event ID lookup |
| Action | Add Attendees to Event | Planning | Event ID + attendee list |
| Action | Find Busy/Free Periods | — | Calendar availability check |

---

## Google Docs

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New Document | Planning | Scheduled polling, folder selector |
| Action | Create Document | Planning | Document creation |
| Action | Read Document | Planning | Document content extraction |
| Action | Edit Template File | Planning | Template variable replacement |
| Action | Search for Document | Planning | Name-based search |
| Action | Append Text | Planning | Document content append |

---

## Google Gemini

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Action | Generate Content | In progress | Model selector, prompt input |
| Action | Generate Content from Image | In progress | Model selector, image input |
| Action | Text to Speech | In progress | Model/voice selector, file storage + URL generation |
| Action | Generate Content with File Search | Planning | File search integration |
| Action | Chat with Gemini | Planning | Session-based conversation |
| Action | Conversation | Done | Model selector, conversation context |
| Action | Summarize | Done | Model selector, text input |

---

## OpenAI

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Action | Ask ChatGPT | In progress | Model selector, prompt input |
| Action | Summarize | Done | Model selector, text summarization |
| Action | Conversation | Done | Session-based conversation, message history |
| Action | Generate Image | In progress | Text-to-image, model selector |
| Action | Text-to-Speech | In progress | Voice selector, audio generation |
| Action | Translate Audio | In progress | Audio file input, language detection |
| Action | Ask Assistant | In progress | Assistant ID, thread management |
| Action | Transcribe Audio | In progress | Audio file input, transcription |
| Action | Extract Structured Data from Text | Planning | Schema definition, text extraction |

---

## Anthropic Claude

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Action | Ask Claude | Planning | Model selector, prompt input |
| Action | Summarize | Done | Model selector (Haiku), text summarization |
| Action | Conversation | Done | Session-based conversation |
| Action | Extract Structured Data | Planning | Schema definition, multi-format input (text, image, PDF) |

---

## AI (Generic)

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Action | Ask AI | Done | Flexible AI step, data analysis/draft/decide |
| Action | AI Chat | Done | Session ID-based context memory |
| Action | Generate Image | In progress | Text-to-image |
| Action | Extract Structured Data | — | Schema-based extraction |
| Action | Summarize Text | Done | Text summarization |
| Action | Classify Text | In progress | Custom label classification |
| Action | Run Agent | Planning | Multi-step reasoning agent |

---

## Gmail

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New Email | Done | Scheduled polling, inbox monitoring |
| Trigger | New Labeled Email | Done | Label-based filtering |
| Action | Send Email | Done | Email composition (to, subject, body, attachments) |

---

## Telegram Bot

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New Message | Done | Instant webhook |
| Trigger | Edit Message | — | Proposed addition |
| Action | Send Text Message | Done | Chat ID, message text |
| Action | Get Chat Member | — | — |
| Action | Create Invite Link | Pending | Invite link generation |
| Action | Send Media | Pending | Binary file send |
| Action | Get File | — | File download |

---

## Facebook Messenger

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New Message | Done | Page webhook |
| Action | Send Message | Done | Reply to sender pattern |

---

## Viptalk

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New Message | Done | Message receive |
| Action | Send Message to Email | Done | Email-based send |
| Action | Send Message to RoomID | Done | Room ID-based send |

---

## Human Input

| Type | Name | Status | Priority | Reusable Logic |
|---|---|---|---|---|
| Trigger | Web Form | Planning | High | Instant, form submission |
| Trigger | Chat UI | Planning | High | Instant, chat message |
| Action | Respond on UI | Planning | High | Text/file response |

---

## Internal Table

| Type | Name | Status | Reusable Logic |
|---|---|---|---|
| Trigger | New Record Created | — | Instant, table selector |
| Trigger | Record Deleted | — | Instant, table selector |
| Trigger | Record Updated | — | Instant, table selector |
| Action | Create Record(s) | — | Table selector, field mapping |
| Action | Update Record | — | Record ID, field update |
| Action | Find Records | — | Filter-based search |
| Action | Delete Record(s) | — | Record ID input |
| Action | Get Record | — | Record ID lookup |
| Action | Clear Table | — | Table selector |

---

## Platform Utilities

| Node | Type | Name | Status | Reusable Logic |
|---|---|---|---|---|
| Error Handle | Action | — | Done | Error branching |
| Condition | Action | True/False (IF) | Done | Boolean condition routing |
| Filter | Action | — | Done | Data filtering |
| Loop | Action | — | Done | Iteration over arrays |
| Delay | Action | Delay For / Delay Until | Done | Duration or timestamp-based delay |
| JSON | Action | Convert Json↔Text | Done | JSON parse/stringify |
| CSV | Action | Convert CSV↔JSON | Done | CSV/JSON conversion |
| Text Helper | Action | Concatenate, Split, Replace, Find, Slugify, etc. | Done | String manipulation |
| Date Helper | Action | Get Current Date, Format, Difference, Add/Subtract, etc. | Done | Date manipulation |
| Math Helper | Action | Addition, Subtraction, Multiplication, Division, Modulo, Random | Done | Numeric operations |
| Image Helper | Action | Crop, Resize, Rotate, Compress, Base64, Metadata | Done | Image processing |
| Data Summarizer | Action | Average, Sum, Count Uniques, Min/Max | Done | Aggregation |
| Storage | Action | Get, Put, Append, Remove, List operations | Done | Key-value storage |
| Files Helper | Action | Read, Create, Zip, Unzip, Check type, Change encoding | Done | File operations |

---

## Other Integrations

| Node | Type | Name | Status | Reusable Logic |
|---|---|---|---|---|
| Google Forms | Trigger | New Response | — | Scheduled polling |
| Google Chat | Trigger | New Message / New Mention | — | Instant webhook |
| Google Chat | Action | Send Message, Add Member, Search, Get Details | — | Space/conversation targeting |
| RSS Feed | Trigger | New Item In Feed / Multiple Feeds | — | Scheduled polling |
| RSS Feed | Action | Pull Items | Done | Feed URL input |
| PDF | Action | Extract Text, Text to PDF, Page Count, Merge, Convert, Extract Pages | — | Binary file input/output |
| SMTP | Action | Send Email | — | SMTP server config |
| IMAP | Trigger | New Email | — | Scheduled polling |
| IMAP | Action | Mark Read/Unread, Move, Copy, Delete | — | Email operations |
| YouTube | Trigger | New Video In Channel | — | Scheduled polling |
| Instagram (Business) | Action | Upload Photo / Upload Reel | — | Media upload |
| Facebook Pages | Action | Create Post / Video / Photo | — | Page content creation |
| Jira Cloud | Trigger | New Issue / Updated Issue Status / Updated Issue | — | Scheduled polling |
| Jira Cloud | Action | Create/Update/Find/Get/Search/Comment/Attach/Link/Watcher | — | Issue management |
| Confluence | Trigger | New Page | — | Scheduled polling |
| Confluence | Action | Get Page Content, Create from Template | — | Page operations |
| GitHub | Trigger | PR, Issue, Discussion, Branch, Label, Release, Star, Push, Comment, Collaborator, Milestone | — | Instant webhook |
| GitHub | Action | Create/Update Issue, Branch, Comment, PR Review, Labels, etc. | — | Repository operations |
| FTP/SFTP | Trigger | New File | — | Scheduled polling |
| FTP/SFTP | Action | Upload, Read, Create, Delete, List, Rename | — | File system operations |
| Figma | Trigger | New Comment | — | Instant webhook |
| Figma | Action | Get File, Post Comment, Get Comments | — | Design file operations |
| Amazon S3 | Trigger | New or Updated File | — | Scheduled polling |
| Amazon S3 | Action | Upload, Read, Delete, Move, Generate URL | — | Object storage operations |
| Amazon S3 | Action | List Files | In progress | Folder/prefix listing |
| ElevenLabs | Action | Text to Speech | — | Voice selector, audio generation |
| X (Twitter) | Action | Create Tweet / Reply | — | Tweet composition |
