# Evonic — Ringkasan Dokumentasi Menyeluruh (berdasarkan evonic.dev)

> Scope utama: pembacaan dokumentasi Evonic mulai dari `https://evonic.dev/system/`, lalu diperluas ke bagian arsitektur, agent lifecycle, tools, skills, plugin, knowledge base, channels, evaluasi, dan deployment/operasional.

## 1) Gambaran Besar: Evonic itu apa?

Evonic adalah **agentic AI framework/platform** untuk membangun dan mengoperasikan agent dari tahap desain sampai produksi. Secara praktik, Evonic menggabungkan:

- **Agent runtime** (loop eksekusi inti)
- **Model abstraction** (dukungan provider/model berbeda)
- **Tooling** (built-in + extensible via skills)
- **Knowledge base per agent**
- **Channels** (interface masuk/keluar agent)
- **Plugins** (extension berbasis event)
- **Persistence** (SQLite untuk state & history)

Referensi:
- https://evonic.dev/system/
- https://evonic.dev/development/architecture/

---

## 2) Arsitektur Inti (System Design)

### 2.1 Runtime loop agent
Alur umum runtime:

1. Pesan/user input masuk via channel/API.
2. Runtime load konfigurasi agent + session history.
3. Runtime panggil model LLM.
4. Jika model meminta tool call, runtime jalankan tool lalu ulangi loop.
5. Saat jawaban final siap, hasil dipersist + dikirim balik.

Implikasi: agent bersifat **stateful** dan bisa menyelesaikan task multi-step.

Referensi:
- https://evonic.dev/system/agents/
- https://evonic.dev/agents/overview/

### 2.2 Event-driven backbone
Evonic memakai event stream/pub-sub untuk lifecycle events; misalnya:

- `message_received`
- `processing_started`
- `tool_executed`
- `final_answer`
- `turn_complete`

Manfaat: plugin dan komponen lain bisa subscribe event tanpa coupling ketat.

Referensi:
- https://evonic.dev/system/events/
- https://evonic.dev/system/plugins/

### 2.3 Persistensi
State penting (agent, channel, sessions/messages, evaluasi) disimpan di SQLite.

Referensi:
- https://evonic.dev/reference/database-schema/

---

## 3) Agent Model di Evonic

### 3.1 Unit kerja utama = Agent
Setiap agent biasanya punya:

- model default
- system prompt/behavior
- daftar tools
- knowledge base
- channels
- skills terpasang
- workplace (lokal/remote/cloud)

Referensi:
- https://evonic.dev/system/agents/
- https://evonic.dev/agents/creating-agents/

### 3.2 Session dan mode
- Session dipisah per konteks user-agent-channel.
- Ada konsep mode stateful seperti **Plan/Execute**.

Ini penting untuk orkestrasi yang lebih deliberate (planning dulu, lalu eksekusi).

Referensi:
- https://evonic.dev/agents/overview/

---

## 4) Tools: Eksekusi & Safety

### 4.1 Tool execution
Evonic mendukung tool call agent untuk menjalankan aksi (contoh: shell, Python, read/write, dsb tergantung konfigurasi skill/toolset).

### 4.2 Safety controls
Untuk tool yang eksekutif (mis. bash/runpy), dokumentasi menekankan:

- preferensi sandbox (Docker)
- heuristic filtering/pattern blocking
- path validation/traversal prevention
- command/flag restrictions

Referensi:
- https://evonic.dev/agents/tools/
- https://evonic.dev/security/heuristic-code-safety/

---

## 5) Skills: Paket Kapabilitas

Skills adalah mekanisme extension untuk nambah kemampuan agent secara modular.

Umumnya skill berisi:

- metadata (`skill.json`)
- tool definitions (schema)
- backend implementasi tool
- lifecycle install/uninstall

Ini bikin kapabilitas bisa reusable antar agent/proyek.

Referensi:
- https://evonic.dev/system/skills/
- https://evonic.dev/skills/skills/

---

## 6) Plugins: Extension Berbasis Event

Plugin dipakai untuk fitur cross-cutting (integrasi, observability, automation hook).

Karakteristik:
- subscribe ke event runtime
- dapat expose routes tertentu
- dapat di-enable/disable/reload
- punya best practices operasional sendiri

Referensi:
- https://evonic.dev/system/plugins/
- https://evonic.dev/plugins/setup/
- https://evonic.dev/plugins/best-practices/

---

## 7) Knowledge Base (KB) per Agent

KB agent bersifat per-agent (isolated), dibaca saat dibutuhkan via tooling/read path terbatas.

Manfaat:
- grounding knowledge spesifik agent
- mengurangi prompt bloat dibanding inject semua dokumen tiap request

Referensi:
- https://evonic.dev/agents/knowledge-base/

---

## 8) Channels & Integrasi Interface

Channel = layer interaksi agent dengan dunia luar (chat platform/API integration).

Poin penting:
- satu agent bisa multi-channel
- session history tetap terisolasi per channel
- ada manajemen lifecycle channel (start/stop/update/primary)

Referensi:
- https://evonic.dev/agents/channels/

---

## 9) Workplaces (Execution Environment)

Evonic memisahkan *agent logic* dari *execution environment* lewat konsep workplace:

- local
- remote (SSH)
- cloud-connected (Evonet)

Ini relevan untuk use case distributed execution dan deployment fleksibel.

Referensi:
- https://evonic.dev/agents/workplaces/

---

## 10) Multi-agent & Orkestrasi Antar Agent

Dokumentasi menegaskan dukungan komunikasi agent-to-agent dengan guardrails (rate/limit/depth semantics) untuk menghindari loop liar/ping-pong.

Referensi:
- https://evonic.dev/agents/agent-messaging/

---

## 11) Getting Started & Workflow Praktis Developer

### 11.1 Instalasi cepat
Contoh dari dokumentasi:

```bash
curl -fsSL https://evonic.dev/install.sh | bash
```

Referensi:
- https://evonic.dev/getting-started/installation/

### 11.2 Konfigurasi
Dokumen konfigurasi membahas env seperti base URL model, API key, model name, host/port/debug, dll.

Referensi:
- https://evonic.dev/getting-started/configuration/

### 11.3 Menjalankan platform
Contoh command:

```bash
evonic start
evonic status
evonic stop
```

Referensi:
- https://evonic.dev/getting-started/quickstart/

### 11.4 Buat dan kelola agent
Ada alur via UI/CLI/API untuk create, update, enable/disable, dan remove agent.

Referensi:
- https://evonic.dev/agents/creating-agents/
- https://evonic.dev/reference/api-agents/

---

## 12) Evaluasi & Quality Workflow

Evonic punya workflow evaluasi (test definitions, evaluators, run/status/log) untuk ngukur performa model/agent secara terstruktur.

Referensi:
- https://evonic.dev/evaluation/evaluation-workflow/
- https://evonic.dev/reference/api-evaluation/
- https://evonic.dev/reference/api-test-management/

---

## 13) Operasional & Produksi

### Hal yang terlihat kuat dari dokumentasi
- Arsitektur event-driven yang extensible.
- Konsep per-agent isolation (workspace/KB/session).
- Safety posture untuk tool execution.
- Plugin dan skill sebagai extension points yang jelas.
- Dukungan environment deployment yang fleksibel (workplaces).

### Hal yang perlu dicatat (gap/uncertainty)
- Detail batas skala/high-throughput event bus tidak terlalu dalam di halaman yang dibaca.
- Beberapa guidance produksi tersebar lintas halaman, belum sepenuhnya “single runbook”.
- Ada indikasi sebagian fitur/channel masih bertahap matang.

---

## 14) Kesimpulan Singkat

Secara menyeluruh, Evonic tampak sebagai framework agent yang **cukup lengkap untuk production-minded development**:

- bisa define agent secara granular,
- bisa nambah kemampuan via skills/plugins,
- punya event-driven orchestration,
- punya lapisan evaluasi/testing,
- dan meng-address aspek keamanan eksekusi tool.

Untuk tahap lanjut, langkah paling bernilai biasanya:
1. tentukan 1 use case agent nyata,
2. implement skill/tool minimal,
3. pasang evaluasi regression,
4. deploy di workplace yang sesuai,
5. aktifkan observability/plugin berbasis event.

---

## Sumber yang ditelaah (utama)

- https://evonic.dev/system/
- https://evonic.dev/system/agents/
- https://evonic.dev/system/skills/
- https://evonic.dev/system/plugins/
- https://evonic.dev/system/events/
- https://evonic.dev/development/architecture/
- https://evonic.dev/agents/overview/
- https://evonic.dev/agents/tools/
- https://evonic.dev/agents/knowledge-base/
- https://evonic.dev/agents/channels/
- https://evonic.dev/agents/workplaces/
- https://evonic.dev/agents/agent-messaging/
- https://evonic.dev/getting-started/installation/
- https://evonic.dev/getting-started/configuration/
- https://evonic.dev/getting-started/quickstart/
- https://evonic.dev/agents/creating-agents/
- https://evonic.dev/reference/api-agents/
- https://evonic.dev/reference/database-schema/
- https://evonic.dev/evaluation/evaluation-workflow/
- https://evonic.dev/reference/api-evaluation/
- https://evonic.dev/reference/api-test-management/
- https://evonic.dev/security/heuristic-code-safety/
- https://evonic.dev/plugins/setup/
- https://evonic.dev/plugins/best-practices/
