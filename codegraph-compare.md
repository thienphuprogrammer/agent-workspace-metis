# CodeGraph vs Metis + MCP CodeIQ — Phân tích So sánh

  > Ngày: 2026-06-09

  ---
  
  ## 1. Tổng quan Kiến trúc

  | Chiều | CodeGraph (repo này) | Metis + MCP CodeIQ |
  |---|---|---|
  | Triết lý | Intelligence substrate — đẩy logic vào index | Workflow orchestration — đẩy logic vào prompts/agents |
  | Lớp AI | Không có LLM trong indexing (deterministic AST parse) | MCP CodeIQ = substrate; Metis = LLM-orchestrated workflows |
  | Persistence | SQLite local `.codegraph/` | Không rõ backend |
  | Benchmark | A/B đo được: -35% cost, -57% tokens, -46% time, -71% tool calls | Chưa có benchmark công bố |

  ---

  ## 2. Layer 1: Intelligence Substrate

  ### CodeGraph tools

  | Tool | Mô tả |
  |---|---|
  | `codegraph_explore` | **PRIMARY** — symbol bag → verbatim source + call path + dynamic-dispatch hops, 1 call |
  | `codegraph_search` | Tìm symbol theo tên, trả về kind + location + signature |
  | `codegraph_node` | Source + caller/callee trail; file-path mode = thay thế `Read` tool |
  | `codegraph_callers` | Reverse trace |
  | `codegraph_callees` | Forward trace |
  | `codegraph_impact` | Blast radius — ai bị ảnh hưởng nếu thay đổi symbol này |
  | `codegraph_files` | Liệt kê files trong directory |
  | `codegraph_status` | Index health + stats |

  ### MCP CodeIQ tools

  | Tool | Mô tả |
  |---|---|
  | `search_code` | Semantic search theo ý nghĩa/hành vi |
  | `search_literal_string` | Exact/fuzzy string search (⚠️  có fuzzy behavior không kiểm soát) |
  | `trace_callers_of_node` | Reverse trace (static only) |
  | `trace_calls_from_node` | Forward trace (static only) |
  | `search_comment` | ❌ Dead tool — không ổn định |
  | `get_repo_instruction` | ❌ Không dùng được |
  | `push_instruction` | ❌ Không dùng được |

  ### Điểm CodeGraph vượt trội

  **Dynamic dispatch synthesis**
  - MCP CodeIQ chỉ theo static edges.
  - CodeGraph synthesizes edges qua: React `setState→render`, JSX children, EventEmitter, callbacks.
  - Validated trên Excalidraw: flow 6 hops cross 3 React boundaries — 0 Read, 0 Grep.
  - MCP CodeIQ sẽ break ở đây.

  **`codegraph_explore` là "super-tool"**
  - Một call trả về verbatim source + call path + dynamic hops.
  - MCP CodeIQ cần 2–3 round-trips (`search_code` + `trace_*`) để đạt kết quả tương tự — và vẫn không có dynamic hops.

  **`codegraph_impact`**
  - MCP CodeIQ không có tương đương.
  - Critical cho refactoring/review: cho thấy blast radius trước khi edit.

  **`codegraph_node` thay thế `Read`**
  - Trả về source bytes (byte-for-byte identical với Read) + caller/callee trail + overload awareness.
  - MCP CodeIQ vẫn cần agent mở file riêng.

  **Adaptive output budgets**
  - Tự scale `maxCharsPerFile`, `gapThreshold`, `includeRelationships` theo repo size.
  - Ngăn context bloat trên large repos (excalidraw 643 files, vscode 10K+ files).

  ---
  
  ## 3. Layer 2: Workflow Orchestration

  ### Metis assets (CodeGraph không có)

  | Asset | Files | Vai trò |
  |---|---|---|
  | **Skills** | `.claude/skills/metis-bugfix.md`, `metis-review.md`, `metis-test.md`, `metis-security.md`, `metis-design.md`, ... | Biến 
  MCP primitives thành playbook nghiệp vụ |
  | **Agents** | `.claude/agents/metis.md`, `metis-search.md`, `metis-solver.md`, `metis-reviewer.md`, `metis-tester.md` | Phân vai rõ ràng 
  theo tác vụ |
  | **Prompt templates** | `.github/prompts/metis-bugfix.prompt.md`, `metis-review.prompt.md`, `metis-security.prompt.md`, ... | Chuẩn hóa 
  output format (Context/Analysis/Action/Verification) |
  | **Slash commands** | `.claude/commands/metis-bugfix`, `/metis-review`, `/metis-test`, ... | Tăng tốc UX người dùng cuối |

  ### CodeGraph guidance layer (hạn chế)

  `src/mcp/server-instructions.ts` — gửi qua MCP `initialize` response. Là **general tool guidance**, không phải task-specific playbook. Nói
  "dùng tool nào cho câu hỏi nào", không nói "khi fix bug thì làm theo flow cụ thể nào".

  ---
  
  ## 4. Use Case Coverage Matrix

  | Use case | CodeGraph | Metis + MCP CodeIQ | Ghi chú |
  |---|---|---|---|
  | Code search (semantic) | ✅ `codegraph_explore` / `codegraph_search` | ✅ `search_code` | CodeGraph chính xác hơn (AST-based) |
  | Code search (exact) | ✅ `codegraph_search` | ⚠️  `search_literal_string` (fuzzy) | CodeGraph không bị fuzzy leak |
  | Caller/callee trace (static) | ✅ | ✅ | Tương đương |
  | Dynamic dispatch trace | ✅ React, JSX, callback, EventEmitter | ❌ Static only | CodeGraph hơn rõ rệt |
  | Blast radius / impact | ✅ `codegraph_impact` | ❌ Không có | CodeGraph hơn |
  | Overload resolution | ✅ Tất cả overloads trong 1 call | ❌ | CodeGraph hơn |
  | Bug fix workflow | ⚠️  Primitives có, orchestration thiếu | ✅ `metis-solver` + bugfix skill/prompt/command | Metis hơn |
  | Code review workflow | ⚠️  `codegraph_impact` primitive | ✅ `metis-reviewer` + review policy | Metis hơn |
  | Test generation | ❌ | ✅ `metis-tester` | Metis hơn |
  | Security scan | ❌ | ✅ OWASP checklist trong `metis-security` | Metis hơn |
  | Design documentation | ❌ | ✅ `detail-design.prompt.md` | Metis hơn |
  | Feature implementation | ⚠️  Context tốt, không orchestrate | ✅ `metis-feature` + `metis-solver` | Metis hơn |
  | Copilot integration | ❌ | ✅ `.github/prompts` | Metis hơn |

  ---
  
  ## 5. Pros & Cons

  ### CodeGraph

  **Pros**
  - Intelligence chính xác hơn: dynamic dispatch, overload-aware, blast-radius
  - Benchmark thực đo: -57% tokens, -46% time, -71% tool calls (trên 7 repos)
  - Local-first, privacy: toàn bộ data trong `.codegraph/` SQLite, không ra ngoài
  - Multi-language deep support: TypeScript, Python, Go, Rust, Swift, C#, PHP, Delphi, Svelte, Vue...
  - Framework detectors tự động: React, Express, FastAPI, Django, Rails, Spring, Gin, Axum, ASP.NET, Vapor
  - Installer ecosystem: tự cấu hình Claude Code, Cursor, Codex, opencode
  - `codegraph_node` thay thế `Read` hoàn toàn cho indexed files
  - Deterministic — không phụ thuộc LLM để build index

  **Cons**
  - **Không có workflow orchestration layer** — agent tự quyết định flow, không có playbook
  - **Không có specialized agents** — không phân vai solver/reviewer/tester
  - **Không có slash commands** nghiệp vụ
  - **Không có prompt templates** chuẩn hóa output format
  - **Không có Copilot integration** (`.github/prompts`)
  - Startup latency ~2-3s (cần pre-warm daemon cho eval chính xác)
  - Index lag ~1s sau khi edit file

  ---

  ### Metis + MCP CodeIQ

  **Pros**
  - Workflow orchestration đầy đủ: search → trace → edit → verify đóng gói thành playbook
  - Specialized agents giảm nhiễu, tăng focus theo từng tác vụ
  - Slash commands tăng tốc UX người dùng cuối
  - Prompt templates standardize output (Context / Analysis / Action / Verification)
  - Copilot integration qua `.github/prompts`
  - Security workflow với OWASP checklist built-in
  - Test generation workflow chuyên biệt

  **Cons**
  - **Intelligence substrate yếu hơn**: chỉ 4 tools, static-only traces
  - **Không có dynamic dispatch coverage**: flow break tại React re-render, JSX children
  - **Không có blast-radius tool** (`codegraph_impact`)
  - **`search_literal_string` fuzzy behavior** — cần fallback grep cho exact match
  - **`search_comment` dead tool** — mất một loại query
  - **Chưa có A/B benchmark** công bố
  - Phụ thuộc vào orchestration layer: nếu Metis skills viết sai, toàn bộ flow sai
  - `get_repo_instruction` và `push_instruction` không dùng được

  ---

  ## 6. Kết luận

  CodeGraph  = tốt hơn ở tầng intelligence  (what the graph knows)
  Metis      = tốt hơn ở tầng orchestration (how to use the graph)

  ### Định vị khuyến nghị

  | Mục tiêu | Lựa chọn tốt nhất |
  |---|---|
  | Code search + system analysis thuần túy | **CodeGraph** — substrate mạnh hơn, benchmark có thực |
  | Bug fix / review / test / security end-to-end | **Metis** — có workflow layer hoàn chỉnh |
  | Trải nghiệm tốt nhất toàn diện | **CodeGraph + custom workflow layer** (skills/agents/commands tự build) |

  Điều Metis README kết luận áp dụng ngược lại: dùng **CodeGraph + custom workflow layer** sẽ cho trải nghiệm tốt hơn Metis + MCP CodeIQ, vì
  substrate mạnh hơn — nhưng phải tự build lớp orchestration.