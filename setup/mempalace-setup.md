# Setup MemPalace: "Bộ Não Thứ Hai" Cho AI Agent

*Biến AI từ thằng "hay quên" thành dev có trí nhớ dài hạn.*

---

## Nó là gì?

**MemPalace** là MCP memory server local-first. Lưu verbatim code, conversation, và context của từng project. Agent nào connect được MCP (opencode, Claude Code, Cursor, Antigravity) đều có thể dùng nó như "bộ não" riêng.

Setup xong, thay vì mỗi lần chat lại phải giải thích lại từ đầu, agent tự động query memory cũ ra đọc trước khi code.

---

## 1. Cài MemPalace (global)

```bash
# Dùng uv (recommended)
uv tool install mempalace

# Hoặc pipx
pipx install mempalace
```

Kiểm tra:
```bash
mempalace --version
# MemPalace 3.4.1 (hoặc mới hơn)

mempalace-mcp --version
```

> **Note:** `mempalace-mcp` là MCP server - cái mà agent kết nối tới. Nếu không thấy, chạy `uv tool install mempalace` lại.

---

## 2. Init Palace Cho Từng Dự Án

Mỗi dự án cần một "palace" riêng (dữ liệu không bị lẫn giữa các dự án):

```bash
# Di chuyển vào project và init
cd /path/to/project
mempalace init .

# Nếu muốn init + mine luôn
mempalace init . --auto-mine .
```

Hoặc chỉ định đường dẫn:
```bash
mempalace init C:\Repository\crm-be
```

**Các project trong repository này:**

| Dự án | Status | Drawers |
|-------|--------|---------|
| `crm-batch` | ✅ | 251 |
| `crm-be` | ✅ | 1322 |
| `crm-fe` | ✅ | 404 |
| `ECQS` | ✅ | 1181 |
| `ECQS-FE` | ✅ | 141 |

Kiểm tra toàn bộ:
```bash
mempalace status
```

---

## 3. Cấu Hình MCP Cho Agent

### 3.1 opencode / Claude Code / MCP-compatible agents

Tạo file `.mcp.json` trong thư mục gốc của project:

```json
{
  "mcpServers": {
    "mempalace": {
      "command": "mempalace-mcp"
    }
  }
}
```

Agent nào support MCP sẽ tự động discover file này và kết nối tới MemPalace khi làm việc trong project đó.

### 3.2 Antigravity (Google Gemini)

Antigravity dùng **MCP Store** qua global config. Không cần plugin folder.

Sửa file `~/.gemini/config/mcp_config.json`:

```json
{
  "mcpServers": {
    "mempalace": {
      "command": "mempalace-mcp"
    }
  }
}
```

Antigravity tự động start `mempalace-mcp` khi mở. Agent trong Antigravity sẽ có 33 MCP tools để search/read/write memory.

> ⚠️ **Windows users:** Antigravity plugin (hooks `.sh`) không dùng được trên Windows — bash script mở Git Bash mỗi lần chạy. **Không cài plugin, chỉ cần MCP config là đủ.** Mất auto-save hooks, nhưng query/gọi tools vẫn ngon.

---

## 4. Hooks Chi Tiết (Linux/Mac)

> ⚠️ **Windows không support hooks.** Hooks là bash script (`.sh`), Windows mở Git Bash mỗi lần chạy. Bỏ qua section này nếu dùng Windows.

### Save Hook (Stop event)

Hook chạy mỗi khi agent kết thúc một response. Đếm số message, tới ngưỡng (default 15) thì:

1. Auto-mine toàn bộ conversation transcript vào palace (bao gồm tool output - code, bash results, errors)
2. Block agent bảo nó save lại key decisions, quotes, topics

Cấu hình qua env vars:
- `MEMPAL_SAVE_INTERVAL`: số message giữa các lần save (default 15)
- `MEMPALACE_HOOKS_AUTO_SAVE=false`: tắt auto-save
- `MEMPAL_PYTHON`: path tới Python interpreter (auto-detect nếu không set)
- **`.sh` hooks không chạy trên Windows.** Chỉ dùng MCP tools cho Windows: `mempalace_search`, `mempalace_diary_write`, `mempalace_wake_up`. Không auto-save, nhưng vẫn query memory được.

### Wake Hook (PreInvocation)

Chạy trước mỗi response của agent. Inject `additional_context` với memory recall scope theo đúng project (wing) đang làm việc.

---

## 5. Các Wing (Phân Vùng Dữ Liệu)

Mỗi dự án là một **wing** riêng. Bên trong wing có:
- **Rooms**: phân loại theo thời gian (general, sessions, ...)
- **Drawers**: verbatim content - code, conversation, documentation

Mặc định, `mempalace init` lấy tên thư mục làm tên wing. Ví dụ:
- `crm-batch/` → wing `crm_batch`
- `ECQS/` → wing `ecqs`

Search scope theo wing:
```bash
mempalace search "GraphQL" --wing crm_be
```

---

## 6. Daily Workflow

### Check dung lượng memory
```bash
mempalace status
```

### Mine thêm dữ liệu
```bash
# Project files
mempalace mine /path/to/project

# Conversation transcripts
mempalace mine ~/.claude/projects/ --mode convos
mempalace mine ~/.codex/sessions/ --mode convos

# Extract office docs (PDF, DOCX, XLSX, PPT, EPUB)
pip install mempalace[extract]
mempalace mine /path/to/docs --mode extract
```

### Search memory từ CLI
```bash
mempalace search "từ khóa"
mempalace search "từ khóa" --wing crm_be --top-k 10
```

### Recall cho agent
Agent tự động dùng MCP tool `mempalace_wake_up` và `mempalace_search`. Không cần gõ tay.

---

## 7. Xử Lý Sự Cố

### "mempalace-mcp not found"
```bash
uv tool install mempalace
# hoặc
pipx install mempalace
```

### Hook không chạy
```bash
# Check log
cat ~/.mempalace/hook_state/hook.log

# Set Python path nếu auto-detect fail
export MEMPAL_PYTHON="C:\Users\Admin\AppData\Local\Programs\Python\Python312\python.exe"
```

### Palace bị lỗi index
```bash
mempalace repair --palace ~/.mempalace/palace
mempalace repair rebuild-index --palace ~/.mempalace/palace
```

### Muốn reset toàn bộ
```bash
# Xoá palace (sẽ mất hết dữ liệu đã mine)
rm -rf ~/.mempalace
mempalace init .
```

---

## 8. Tổng Quan Kiến Trúc

```
                     ┌──────────────────────┐
                     │   Antigravity / IDE   │
                     │  (opencode, Cursor,   │
                     │   Claude Code, Codex) │
                     └──────────┬───────────┘
                                │ MCP Protocol
                     ┌──────────▼───────────┐
                     │    mempalace-mcp     │
                     │   (MCP Server, 33    │
                     │    tools: search,    │
                     │    diary, kg, etc.)  │
                     └──────────┬───────────┘
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                  ▼
     ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
     │   Palace     │  │ Knowledge    │  │ Entity       │
     │  (ChromaDB)  │  │ Graph (KG)   │  │ Registry     │
     │  - wings     │  │ (SQLite)     │  │ (JSON)       │
     │  - rooms     │  │ - temporal   │  │ - people     │
     │  - drawers   │  │ - triples    │  │ - projects   │
     └──────────────┘  └──────────────┘  └──────────────┘

     Hook scripts chạy background (Linux/Mac):
     ┌─────────────────────────────────────────┐
     │  Stop hook: auto-mine transcript + save │
     │  PreInvocation hook: inject recall      │
     │  PreCompact hook: save trước khi nén    │
     └─────────────────────────────────────────┘

     Windows: không có hooks, dùng MCP tools thủ công
     mempalace search | mempalace_diary_write | mempalace_wake_up
```

---

*"Mỗi project có một bộ não riêng. Agent vào project nào, nó tự động load memory của project đó. Không còn cảnh 'mất não' giữa chừng nữa."*

Link tham khảo:
- [MemPalace GitHub](https://github.com/mempalace/mempalace)
- [MemPalace Docs](https://mempalaceofficial.com/)
- [MCP Protocol](https://modelcontextprotocol.io/)