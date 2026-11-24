# SlipGuard: เอกสารอธิบายโค้ดส่วนของแอ๊น (`onctoberant`)

> **ที่มาของโค้ด:** อ่านจากเครื่องของแอ๊น `D:\slipguard` เมื่อ 21 พ.ย. 2569 ประมาณ 15:45 น.
> **เลขบรรทัด** ในกรอบโค้ดทุกกรอบ = เลขบรรทัดจริงในไฟล์ (VS Code กด `Ctrl+G` แล้วพิมพ์เลขบรรทัดเพื่อไปที่บรรทัดนั้นได้)
> **โค้ดในเอกสารนี้ดึงจากไฟล์จริงด้วยสคริปต์** ไม่ได้พิมพ์ใหม่ และตรวจแล้วว่าทุกบรรทัดที่ไม่ใช่บรรทัดว่างของทุกไฟล์ในข้อ 2 ถูกแสดงครบ
> ถ้าเครื่องแอ๊น `git pull` แล้วไฟล์เปลี่ยน เลขบรรทัดอาจเลื่อนได้

---

## สารบัญ (Table of Contents)

- **1** [Project Overview — ภาพรวม](#s1)
- **2** [Code Structure — โครงสร้างโค้ด](#s2)
- **3–4** [Detailed + Line-by-Line Explanation — อธิบายโค้ดทีละส่วนและทีละบรรทัด](#s3)
   - [3.1 `backend/app/api/__init__.py`](#f-api-init)
   - [3.2 `backend/app/api/summary.py` — คำนวณยอด](#f-summary)
   - [3.3 `backend/app/api/store.py` — อ่านฐานข้อมูล](#f-store)
   - [3.4 `backend/app/api/reports.py` — API](#f-reports)
   - [3.5 `backend/app/api/sample_data.py` — สลิปสุ่ม](#f-sample)
   - [3.6 `backend/app/api/seed_demo.py` — ใส่/ลบสลิปตัวอย่าง](#f-seed)
   - [3.7 `backend/app/metrics/__init__.py` + `timing.py` — จับเวลา](#f-timing)
   - [3.8 `backend/app/metrics/report.py` — สรุป p50/p95](#f-report)
   - [3.9 จุดที่โค้ดของแอ๊นต่อกับไฟล์ของคนอื่น](#f-touch)
   - [3.10 `frontend/app/layout.tsx`](#f-layout)
   - [3.11 `frontend/app/AutoRefresh.tsx`](#f-autorefresh)
   - [3.12 `frontend/app/page.tsx` — หน้าสรุปยอด](#f-page)
   - [3.13 `frontend/app/globals.css` — สไตล์](#f-css)
   - [3.14 เทสต์ 3 ไฟล์](#f-tests)
- **5** [Workflow / Logic Flow — ลำดับการทำงาน](#s5)
- **6** [Important Concepts — แนวคิดที่ใช้ในโค้ด](#s6)
- **7** [My Contribution — ผลงานของแอ๊น](#s7)
- **8** [ข้อสังเกตและจุดที่ยังไม่สามารถยืนยันได้](#s8)

> **หมายเหตุเรื่องการจัดหัวข้อ:** ข้อ 3 (Detailed) กับข้อ 4 (Line-by-Line) ตามที่ขอ ถูกรวมไว้ในหัวข้อเดียวต่อไฟล์ เพื่อแสดงโค้ดแต่ละช่วง **ครั้งเดียว** แล้วตามด้วย (ก) กล่องสรุปว่าส่วนนี้ทำอะไร/ทำไม/รับอะไร/คืนอะไร/ต่อกับใคร และ (ข) คำอธิบายทีละบรรทัด ถ้าแยกเป็น 2 หัวข้อ โค้ดทุกบรรทัดจะถูกพิมพ์ซ้ำ 2 รอบ เอกสารจะยาวเป็นเท่าตัวโดยไม่ได้เนื้อหาเพิ่ม

**วิธีอ่านคำอธิบายทีละบรรทัด**

- `L14` = บรรทัดที่ 14 · `L14–16` = บรรทัด 14 ถึง 16 ที่ทำงานร่วมกัน
- **ถ้าลบ/เปลี่ยน:** = ผลที่จะเกิดถ้าลบหรือแก้บรรทัดนั้น
- บรรทัดที่เป็นคอมเมนต์ (`#` ใน Python, `//` หรือ `{/* */}` ใน TSX, `/* */` ใน CSS) และ docstring (`"""..."""`) ไม่มีผลต่อการทำงาน คอมเมนต์ของแอ๊นเขียนเป็นภาษาไทยอยู่แล้ว จึงอธิบายรวบเป็นกลุ่ม ไม่ได้แปลซ้ำทีละบรรทัด
- บรรทัดว่างไม่อธิบาย (ใช้เว้นให้อ่านง่าย ตามมาตรฐาน PEP 8 ของ Python คือเว้น 2 บรรทัดระหว่างฟังก์ชันระดับบนสุด)

---

<a id="s1"></a>
## 1. Project Overview — ภาพรวม

### 1.1 SlipGuard คืออะไร

ระบบตรวจสลิปโอนเงินใน LINE OA ของร้านค้าออนไลน์: ลูกค้าส่งรูปสลิป → ระบบตรวจหลายชั้น (L1–L7) → ตอบ ✅ ผ่าน / ⚠️ รอร้านตรวจ / ❌ ไม่ผ่าน · ทีม 5 คนแบ่งงานตามชั้น

### 1.2 ส่วนที่แอ๊นรับผิดชอบ (ตามแผนทีม `claude/slipguard-team-plan.md`)

| งาน | อธิบาย |
|---|---|
| **สรุปยอด (API)** | อ่านสลิปที่วีบันทึกในตาราง `slips` แล้วรวมยอด วัน / เดือน / ปี / ช่วงที่เลือก |
| **`summarize()`** | ฟังก์ชันที่ตกลงกับแพรวไว้ ให้ webhook เรียกตอนแม่ค้าพิมพ์ "ยอดวันนี้ / ยอดเดือนนี้ / ยอดปีนี้" ใน LINE |
| **หน้าเว็บสรุปยอด** | หน้า `http://localhost:3000` (แทน LIFF ที่ตัดไปตามแผนเร่ง) แสดงยอด การ์ดสถานะ กราฟ/ปฏิทิน ตารางสลิปล่าสุด |
| **วัดเวลา (A6)** | `timed()` จับเวลาแต่ละชั้นลง log + `report.py` สรุปเป็นตาราง p50/p95 |
| **หน้าที่ร่วม** | ดูแล CI ให้ผ่าน · รวม slide · ตารางต้นทุน A6 |

### 1.3 วัตถุประสงค์ของแต่ละไฟล์

| ไฟล์ | วัตถุประสงค์ | แตะฐานข้อมูล? | ใครเรียกใช้ |
|---|---|---|---|
| `backend/app/api/__init__.py` | ทำให้โฟลเดอร์ `api` เป็น package + สรุปว่ามีไฟล์อะไร | ไม่ | Python (ตอน import) |
| `backend/app/api/summary.py` | **คำนวณ** ยอดรวม/ช่องกราฟ/นับสถานะ + จัดข้อความ LINE (ฟังก์ชันล้วน) | ไม่ | store.py, reports.py, webhook ของแพรว |
| `backend/app/api/store.py` | **อ่าน** ตาราง `slips` ของวี + `summarize()` + สลิปล่าสุด | อ่าน | reports.py, webhook ของแพรว |
| `backend/app/api/reports.py` | **API HTTP** `/api/v1/reports/summary` และ `/recent` | ผ่าน store.py | หน้าเว็บ (page.tsx) |
| `backend/app/api/sample_data.py` | สร้างสลิปสุ่ม (seed คงที่) | ไม่ | เทสต์, seed_demo.py |
| `backend/app/api/seed_demo.py` | ใส่/ลบสลิปตัวอย่าง `DEMO-` ลงตารางจริง | เขียน/ลบ | คนรันคำสั่งเอง |
| `backend/app/metrics/__init__.py` | ทำให้ `metrics` เป็น package | ไม่ | Python |
| `backend/app/metrics/timing.py` | `timed()` จับเวลาแล้วพิมพ์ log | ไม่ | ชั้น L1–L7 (ถ้าครอบ — ดูข้อ 8) |
| `backend/app/metrics/report.py` | อ่าน log → ตาราง markdown p50/p95 | ไม่ | คนรันคำสั่งเอง |
| `frontend/app/layout.tsx` | โครง HTML ที่ทุกหน้าใช้ | – | Next.js |
| `frontend/app/page.tsx` | หน้าสรุปยอดทั้งหน้า | เรียก API | เบราว์เซอร์ |
| `frontend/app/AutoRefresh.tsx` | รีเฟรชข้อมูลทุก 15 วินาที | – | page.tsx |
| `frontend/app/globals.css` | สีและเลย์เอาต์ทั้งเว็บ | – | layout.tsx |
| `backend/tests/test_reports.py` | เทสต์การคำนวณ + API (ไม่ใช้ฐานข้อมูล) | ไม่ | pytest / CI |
| `backend/tests/test_reports_db.py` | เทสต์ที่อ่าน/เขียนตารางจริง (SQLite) | SQLite ชั่วคราว | pytest / CI |
| `backend/tests/test_metrics.py` | เทสต์ timing + report | ไม่ | pytest / CI |

### 1.4 โค้ดเหล่านี้ทำงานร่วมกันอย่างไร (ภาพรวม)

```
ลูกค้าส่งสลิปใน LINE
  → webhook (แพรว) → run_pipeline L1–L5 (บีม/ชิชา) → verify_slip L6–L7 (วี)
  → INSERT 1 แถวลงตาราง slips          ← จุดที่ข้อมูลของแอ๊นเริ่มต้น

                ┌──────────── ตาราง slips (Postgres) ────────────┐
                │                                                 │
     store.load_slips()                                  store.load_recent()
     (WHERE shop_id + ช่วงวัน)                            (ORDER BY ใหม่→เก่า LIMIT)
                │                                                 │
     summary.build_summary()                                      │
     (รวมยอด / ช่องกราฟ / นับสถานะ)                                 │
        │                 │                                       │
 reports.summary()   store.summarize()  ──→ summary_message()     │
 GET /api/v1/        (แพรวเรียก)            → webhook ตอบ LINE      │
 reports/summary                                           reports.recent()
        │                                                  GET /api/v1/reports/recent
        └──────────────── JSON ─────────────┬──────────────────────┘
                                            ↓
                          frontend/app/page.tsx (Server Component)
                          → HTML ที่มีตัวเลขครบ → เบราว์เซอร์
                          ← AutoRefresh ขอ render ใหม่ทุก 15 วินาที

วัดเวลา:  with timed("L2_qr"): ...  → log "TIMING layer=L2_qr ms=12.4"
          webhook ของแพรว         → log "slip_timing ... total_ms=1840"
          docker compose logs | python -m app.metrics.report → ตาราง p50/p95 → slide A6
```

**หลักการแบ่งไฟล์ (แยกตามหน้าที่ — separation of concerns)**

- `summary.py` **ไม่รู้จักฐานข้อมูล** รับ list ของสลิปเข้า คืน dict ออก → เทสต์ได้โดยไม่ต้องเปิด docker
- `store.py` เป็น **ที่เดียว** ที่คุยกับตาราง `slips` ถ้าวีเปลี่ยนชื่อคอลัมน์ แก้ไฟล์เดียว
- `reports.py` ดูแลเรื่อง **HTTP** (อ่านพารามิเตอร์ URL, ตอบ status code) ไม่มีสูตรคำนวณ
- ผลพลอยได้: ตัวเลขใน LINE (ผ่าน `summarize()`) กับตัวเลขบนเว็บ (ผ่าน API) ใช้ `build_summary()` ตัวเดียวกัน จึงตรงกันแน่นอน

---

<a id="s2"></a>
## 2. Code Structure — โครงสร้างโค้ด

แบ่งตามหน้าที่ที่มีอยู่จริงในโค้ด (หัวข้อที่ไม่มีในโค้ดไม่ได้ใส่)

### 2.1 Import / Dependencies

| ไลบรารี | ใช้ที่ไฟล์ | ใช้ทำอะไร |
|---|---|---|
| `datetime`, `decimal`, `typing`, `dataclasses`, `collections.abc` (Python มาตรฐาน) | summary, store, reports, sample_data | วันเวลา, เงิน, type |
| `random`, `functools.lru_cache` | sample_data, reports | สุ่มแบบมี seed, จำผลฟังก์ชัน |
| `argparse`, `os` | seed_demo, reports | อ่านคำสั่ง command line, อ่าน env |
| `logging`, `time`, `contextlib` | timing | log, นาฬิกา, ทำ `with` |
| `re`, `math`, `sys` | report | regex, ปัดขึ้น, stdin/argv |
| **FastAPI** | reports | ทำ API (`APIRouter`, `Depends`, `HTTPException`, `Query`) |
| **SQLAlchemy 2** | store, reports, seed_demo | คุยกับฐานข้อมูล (`select`, `delete`, `Session`, `Engine`) |
| `app.core.contracts.Verdict` (ของทีม) | summary, store, sample_data | ค่า passed / suspicious / failed |
| `app.verify.models.Slip`, `app.verify.db` (ของวี) | store, reports, seed_demo | ตาราง slips, สร้าง engine/ตาราง |
| **Next.js 14 / React 18** | frontend | หน้าเว็บ (`Link`, `useRouter`, `useEffect`) |
| **pytest**, `fastapi.testclient` | tests | เทสต์ |

### 2.2 Variables / Constants

| ชื่อ | ไฟล์ | ค่า/ความหมาย |
|---|---|---|
| `BKK` | summary.py L23 | เขตเวลาไทย +07:00 |
| `Mode` | summary.py L26 | `"daily" \| "monthly" \| "yearly" \| "custom"` |
| `MAX_CUSTOM_DAYS` | summary.py L29 | 366 |
| `Period`, `PERIOD_TO_MODE` | store.py L33–34 | `"day"` → `"daily"` ฯลฯ |
| `router`, `SlipSource`, `RecentSource`, `DB_ERROR_DETAIL` | reports.py L37–50 | router ของ API, type ฟังก์ชัน, ข้อความ error |
| `BANKS`, `AMOUNTS`, `SAMPLE_REASONS`, `SAMPLE_SCORES` | sample_data.py L18–29 | ข้อมูลสำหรับสุ่ม |
| `DEMO_PREFIX` | seed_demo.py L29 | `"DEMO-"` |
| `logger` | timing.py L28 | logger ชื่อ `slipguard.timing` |
| `LINE_RE`, `TOTAL_RE` | report.py L29, L32 | regex อ่าน log |
| `API`, `MODES`, `VERDICT`, `FILTERS`, `WEEKDAYS`, `SHOW_ROWS`, `FETCH_ROWS` | page.tsx L21–78 | ค่าตั้งของหน้าเว็บ |
| ตัวแปรสี `--mint` ฯลฯ | globals.css L6–20 | สีทั้งเว็บ |

### 2.3 Components / Classes

- **Class:** `SlipRecord` (summary.py L32–43) — dataclass แบบแก้ค่าไม่ได้
- **React Components:** `RootLayout` (layout.tsx), `Home` (page.tsx, Server Component), `AutoRefresh` (Client Component)

### 2.4 Functions (เรียงตามไฟล์)

| ไฟล์ | ฟังก์ชัน |
|---|---|
| summary.py | `period_range`, `bucket_labels`, `bucket_of`, `build_summary`, `_thai_date`, `summary_message` |
| store.py | `_utc_midnight`, `_as_utc`, `_final_verdict`, `to_record`, `load_slips`, `summarize`, `recent_item`, `load_recent` |
| reports.py | `_engine`, `_shop_id`, `db_source`, `get_source`, `db_recent_source`, `get_recent_source`, `summary` (endpoint), `recent` (endpoint) |
| sample_data.py | `sample_slips`, `_generate` |
| seed_demo.py | `seed`, `main` |
| timing.py | `timed` (context manager) |
| report.py | `parse`, `percentile`, `markdown_table`, `main` |
| page.tsx | `getJson`, `todayInBangkok`, `toDate`, `toIso`, `shiftPeriod`, `currentPeriodStart`, `periodTitle`, `axisLabel`, `showTick`, `baht`, `shortBaht`, `heat`, `slipTime`, `first`, `href` |

### 2.5 API / Database

- **API:** `GET /api/v1/reports/summary`, `GET /api/v1/reports/recent` (reports.py)
- **Database (อ่าน):** `load_slips`, `load_recent` (store.py) · **(เขียน/ลบ):** `seed` (seed_demo.py) · ตารางที่ใช้: `slips` ของวีเท่านั้น

### 2.6 State Management / Event Handlers

- หน้าเว็บ **ไม่มี React state** (`useState`) เลย สถานะทั้งหมด (โหมด, วันที่, ตัวกรอง) เก็บใน **URL** เช่น `/?mode=monthly&date=2026-10-01&filter=failed` แล้วแต่ละปุ่มเป็น `Link` ที่เปลี่ยน URL
- **Event:** มีตัวเดียวคือตัวจับเวลา `setInterval` ใน `AutoRefresh` (ทุก 15 วินาที) ไม่มี `onClick`

### 2.7 Conditional Logic / Loops / Error Handling / Return Values (จุดสำคัญ)

- **Conditional:** เลือกช่วงตามโหมด (`period_range`), นับเงินเฉพาะ passed (`build_summary` L126), ผลแม่ค้าก่อนผลระบบ (`_final_verdict`), ปฏิทินหรือกราฟ (page.tsx L292)
- **Loops:** วนสลิป (`build_summary` L120), วนสร้างสลิปสุ่ม (`_generate` L50–53), วนบรรทัด log (`parse` L38), `.map()` วาดเมนู/การ์ด/แท่ง/แถว ใน page.tsx
- **Error Handling:** `ValueError` → 422 และ `SQLAlchemyError`/`RuntimeError` → 503 (reports.py), `try/finally` (timing.py), `try/catch` ใน `getJson` (page.tsx), `except Exception` ใน webhook ของแพรว
- **Return Values:** `build_summary` คืน dict รูปแบบตายตัว (ดู 3.2) ซึ่งเป็น "สัญญา" ระหว่าง backend กับ frontend (type `Summary` ใน page.tsx ต้องตรงกัน)

---

<a id="s3"></a>
## 3–4. Detailed + Line-by-Line Explanation

ลำดับ: backend (คำนวณ → อ่านฐานข้อมูล → API → ข้อมูลตัวอย่าง → วัดเวลา) → จุดต่อกับไฟล์คนอื่น → frontend → เทสต์

<a id="f-api-init"></a>
### 3.1 `backend/app/api/__init__.py`

`backend/app/api/__init__.py` · บรรทัด 1–8

```python
1  """API สรุปยอดสำหรับหน้าเว็บของแม่ค้า (แอ๊น)
2  
3  summary.py     คำนวณยอด (ฟังก์ชันล้วน ไม่แตะฐานข้อมูล)
4  store.py       อ่านสลิปจากตาราง slips + summarize() ให้แพรวเรียก
5  reports.py     endpoint /api/v1/reports/summary และ /recent
6  sample_data.py ข้อมูลสลิปตัวอย่าง
7  seed_demo.py   ใส่/ลบสลิปตัวอย่างในฐานข้อมูล (python -m app.api.seed_demo)
8  """
```

> **ทำอะไร:** ไฟล์ชื่อ `__init__.py` ทำให้ Python มองโฟลเดอร์ `api` เป็น **package** จึงเขียน `from app.api.store import summarize` ได้ · ในไฟล์มีแค่ docstring สรุปว่าแต่ละไฟล์ทำอะไร ไม่มีโค้ดทำงาน

- `L1–8` docstring (ข้อความในเครื่องหมาย `"""`) ใช้เป็นสารบัญของโฟลเดอร์ ไม่มีผลต่อการทำงาน · **ถ้าลบเนื้อหา:** ไม่มีผล (แต่ถ้าลบ *ไฟล์ทิ้ง* Python 3 ยังมองเป็น namespace package ได้ แต่เครื่องมือบางตัว เช่น pytest บางโหมด จะหาโมดูลยากขึ้น จึงควรมีไว้)

---

<a id="f-summary"></a>
### 3.2 `backend/app/api/summary.py` — หัวใจของการคำนวณยอด

> **ภาพรวมไฟล์**
> - **ทำอะไร:** รับรายการสลิป (list ของ `SlipRecord`) + ช่วงวัน → คืนยอดรวม, จำนวนสลิปที่ผ่าน, จำนวนแต่ละสถานะ, ข้อมูลกราฟ · และจัดข้อความสรุปยอดสำหรับ LINE
> - **ทำไมแยกไฟล์นี้:** เป็น **pure function** (ฟังก์ชันล้วน: input เดิมได้ output เดิมเสมอ ไม่แตะฐานข้อมูล/ไฟล์/เวลาจริง) จึงเทสต์ได้ด้วยข้อมูลในโค้ด ไม่ต้องเปิด docker
> - **ใครเรียก:** `store.summarize()`, `reports.summary()`, และ `webhook.summary_text()` ของแพรว (เรียก `summary_message`)

#### 3.2.1 Docstring อธิบายกติกา

`backend/app/api/summary.py` · บรรทัด 1–12

```python
 1  """คำนวณยอดสรุป วัน/เดือน/ปี/ช่วงที่เลือก (แอ๊น)
 2  
 3  ไฟล์นี้เป็น "ฟังก์ชันล้วน" (pure function) คือรับข้อมูลเข้า คืนผลออก ไม่แตะฐานข้อมูล
 4  → เทสต์ได้ง่าย ไม่ต้องเปิด docker (ดู tests/test_reports.py)
 5  ส่วนที่อ่านจากฐานข้อมูลอยู่ใน store.py แล้วส่งรายการสลิปมาให้ไฟล์นี้คำนวณ
 6  
 7  กติกา:
 8  - total_amount และ slip_count นับเฉพาะสลิปที่ผ่าน (✅) เพราะเป็นเงินที่ร้านได้รับจริง
 9  - status_counts นับทุกสถานะในช่วงนั้น (✅ ⚠️ ❌) ให้แม่ค้าเห็นว่ามีสลิปรอตรวจกี่ใบ
10  - ตัดวันตามเวลาไทย (+07:00) ไม่ใช่ UTC  ไม่งั้นสลิปตอนตี 0–7 จะไปอยู่ผิดวัน
11  - series มีทุกช่อง แม้ช่องนั้นยอดเป็น 0  กราฟจะได้ไม่ขาดช่วง
12  """
```

- `L1–12` docstring ของไฟล์ บอก 4 กติกาหลัก: (1) ยอดเงินนับเฉพาะ ✅ (2) นับจำนวนทุกสถานะ (3) ตัดวันตามเวลาไทย (4) กราฟมีทุกช่องแม้ยอด 0 · ไม่มีผลต่อการทำงาน แต่ใช้ตอบอาจารย์ได้ว่า "กติกาการนับเขียนไว้ตรงไหน"

#### 3.2.2 Import และค่าคงที่

`backend/app/api/summary.py` · บรรทัด 14–29

```python
14  from collections.abc import Iterable
15  from dataclasses import dataclass
16  from datetime import date, datetime, timedelta, timezone
17  from decimal import Decimal
18  from typing import Literal
19  
20  from app.core.contracts import Verdict
21  
22  # เวลาไทย: ไทยไม่มี daylight saving จึงใช้ +07:00 คงที่ได้ตลอดปี (ไม่ต้องลงไลบรารี timezone เพิ่ม)
23  BKK = timezone(timedelta(hours=7))
24  
25  # โหมดที่ API รับ: daily = วัน, monthly = เดือน, yearly = ปี, custom = เลือกช่วงเอง
26  Mode = Literal["daily", "monthly", "yearly", "custom"]
27  
28  # จำกัดช่วง custom ไม่เกิน 1 ปี (366 วันเผื่อปีอธิกสุรทิน) กันคนขอข้อมูลยาวจน API ช้า
29  MAX_CUSTOM_DAYS = 366
```

> **ทำอะไร:** ดึงเครื่องมือที่ต้องใช้ + กำหนดค่าที่ใช้ทั้งไฟล์ · **ต่อกับ:** `BKK` และ `Mode` ถูก import ต่อโดย store.py และ reports.py

- `L14` `from collections.abc import Iterable` — `Iterable` คือ type "อะไรก็ได้ที่วน `for` ได้" (list, tuple, generator) ใช้เป็น type ของพารามิเตอร์ `slips` ใน `build_summary` ให้รับได้หลายแบบ · **ถ้าลบ:** L99 จะ `NameError: name 'Iterable' is not defined` ตอน import ไฟล์
- `L15` `dataclass` — decorator สร้างคลาสเก็บข้อมูลโดย Python เขียน `__init__`, `__repr__`, `__eq__` ให้ · **ถ้าลบ:** L32 error
- `L16` `date` = วันที่อย่างเดียว · `datetime` = วัน+เวลา · `timedelta` = ระยะเวลา (เช่น `timedelta(days=1)`) · `timezone` = เขตเวลาแบบออฟเซ็ตคงที่
- `L17` `Decimal` = ทศนิยมฐาน 10 แบบแม่นยำ สำหรับเงิน (เหตุผลใน L41)
- `L18` `Literal` = type ที่ยอมรับเฉพาะค่าที่ระบุ
- `L20` `Verdict` = enum ของทีมใน `app/core/contracts.py` มี `PASSED="passed"`, `SUSPICIOUS="suspicious"`, `FAILED="failed"` · ใช้ร่วมกับทุกคนในทีม ผลตรวจจึงสะกดตรงกันทั้งระบบ
- `L22` คอมเมนต์อธิบาย L23
- `L23` `BKK = timezone(timedelta(hours=7))` — สร้างเขตเวลา +07:00 ไทยไม่มี daylight saving (ไม่ปรับเวลาฤดูร้อน) จึงเป็นค่าคงที่ได้ทั้งปี ไม่ต้องลงไลบรารี `zoneinfo`/`pytz` · **ถ้าเปลี่ยนเป็น `timezone.utc`:** สลิปช่วง 00:00–06:59 เวลาไทยจะถูกนับเป็นวันก่อนหน้า และกราฟรายชั่วโมงจะเลื่อนไป 7 ชม.
- `L25–26` `Mode = Literal["daily", "monthly", "yearly", "custom"]` — ตั้งชื่อ type ไว้ใช้ซ้ำ · FastAPI อ่าน type นี้แล้ว **ตรวจค่าให้อัตโนมัติ** ถ้า URL ส่ง `mode=weekly` จะตอบ 422 ทันที ไม่เข้าฟังก์ชัน · **ถ้าเปลี่ยนเป็น `str`:** ค่าแปลกจะหลุดเข้า `period_range` แล้วตกไปทาง custom (บรรทัด 69) ได้ error ข้อความไม่ตรงปัญหา
- `L28–29` `MAX_CUSTOM_DAYS = 366` — ชื่อตัวพิมพ์ใหญ่ = ธรรมเนียมของค่าคงที่ · 366 เผื่อปีอธิกสุรทิน · **ถ้าลบการจำกัด:** ผู้ใช้ขอข้อมูล 10 ปีได้ API จะดึงทุกแถว + สร้างช่องกราฟ 3,650 ช่อง ช้าและหน้าเว็บอ่านไม่ออก

#### 3.2.3 Class `SlipRecord`

`backend/app/api/summary.py` · บรรทัด 32–43

```python
32  @dataclass(frozen=True)
33  class SlipRecord:
34      """สลิปหนึ่งใบในรูปที่การสรุปยอดต้องใช้ (มาจากตาราง slips หรือข้อมูลตัวอย่าง)
35  
36      frozen=True = แก้ค่าหลังสร้างไม่ได้ กันโค้ดส่วนอื่นเผลอแก้ข้อมูลระหว่างคำนวณ
37      """
38  
39      trans_ref: str       # เลขอ้างอิงรายการ (ไม่ได้ใช้คำนวณ เก็บไว้ debug)
40      bank: str            # ธนาคารผู้โอน เช่น KBANK
41      amount: Decimal      # ยอดเงิน ใช้ Decimal ไม่ใช้ float เพราะ float ปัดเศษสตางค์เพี้ยน
42      verdict: Verdict     # ผลตัดสิน passed / suspicious / failed
43      created_at: datetime  # เวลาที่ระบบได้รับสลิป ต้องมี timezone เสมอ (ไม่งั้นแปลงเป็นเวลาไทยไม่ได้)
```

> **ทำอะไร:** กำหนด "รูปร่าง" ของสลิป 1 ใบที่การสรุปยอดใช้ · **ทำไม:** แยกจากคลาส `Slip` ของฐานข้อมูล ทำให้ summary.py ไม่ต้องรู้จัก SQLAlchemy และเทสต์สร้างสลิปเองได้ง่าย · **ใครสร้าง:** `store.to_record()` (จากฐานข้อมูล), `sample_data._generate()` (สุ่ม), เทสต์

- `L32` `@dataclass(frozen=True)` — `@` = decorator (ฟังก์ชันที่ห่อ/แก้ไขคลาสข้างล่าง) · `frozen=True` = สร้างแล้วแก้ไม่ได้ ถ้าเขียน `slip.amount = 0` จะได้ `FrozenInstanceError` · ผลพลอยได้: hash ได้ จึงเก็บใน `lru_cache`/tuple ได้ปลอดภัย · **ถ้าเอา `frozen=True` ออก:** ยังทำงานได้ แต่โค้ดส่วนไหนเผลอแก้ค่าระหว่างคำนวณจะไม่มีใครรู้ และของใน cache ของ sample_data อาจถูกแก้
- `L33` `class SlipRecord:` ประกาศคลาส
- `L34–37` docstring ของคลาส
- `L39` `trans_ref: str` — รูปแบบ `ชื่อ: type` ใน dataclass = 1 ช่องข้อมูล · เลขอ้างอิงไม่ได้ใช้คำนวณ เก็บไว้ debug
- `L40` `bank: str` ธนาคารผู้โอน
- `L41` `amount: Decimal` — **ทำไมไม่ใช้ float:** float เก็บเป็นฐาน 2 ทศนิยมหลายตัวเก็บไม่ตรง เช่น `0.1 + 0.2 == 0.30000000000000004` รวมเงินหลายร้อยใบสตางค์เพี้ยน · Decimal เก็บฐาน 10 ตรงตามที่เห็น และคอลัมน์ `amount` ของวีเป็น `Numeric(12, 2)` ซึ่ง SQLAlchemy คืนเป็น Decimal อยู่แล้ว
- `L42` `verdict: Verdict` ผลตัดสิน (enum ไม่ใช่ str ธรรมดา → พิมพ์ผิดจะ error ทันที)
- `L43` `created_at: datetime` — **ต้องมี timezone** เพราะ L91, L122 เรียก `.astimezone(BKK)` ถ้าเวลาไม่มี timezone Python จะเดาว่าเป็นเวลาของเครื่อง (ใน docker = UTC, บน Windows แอ๊น = เวลาไทย) ผลจะไม่เหมือนกันในแต่ละเครื่อง

#### 3.2.4 `period_range()` — หาช่วงวันที่ต้องสรุป

`backend/app/api/summary.py` · บรรทัด 46–76

```python
46  def period_range(
47      mode: Mode, ref: date, start: date | None = None, end: date | None = None
48  ) -> tuple[date, date]:
49      """คืน (วันเริ่ม, วันถัดจากวันสุดท้าย) ของช่วงที่ต้องสรุป
50  
51      ใช้แบบ "ครึ่งเปิด" [start, end) คือรวมวันเริ่ม ไม่รวมวัน end
52      ข้อดี: ไม่ต้องคิดว่าเดือนนี้มี 28/30/31 วัน แค่ใช้วันที่ 1 ของเดือนถัดไปเป็น end
53  
54      ref = วันอ้างอิง เช่น ขอ monthly + ref 18 พ.ย. → ได้ 1 พ.ย. ถึง 1 ธ.ค.
55      """
56      if mode == "daily":
57          # วันเดียว: ตั้งแต่ ref ถึงก่อนวันพรุ่งนี้
58          return ref, ref + timedelta(days=1)
59      if mode == "monthly":
60          first = ref.replace(day=1)  # วันที่ 1 ของเดือนนั้น
61          # เดือนธันวาคม เดือนถัดไปคือมกราคมของปีหน้า (replace(month=13) จะ error)
62          nxt = date(first.year + 1, 1, 1) if first.month == 12 else first.replace(month=first.month + 1)
63          return first, nxt
64      if mode == "yearly":
65          # 1 ม.ค. ปีนี้ ถึงก่อน 1 ม.ค. ปีหน้า
66          return date(ref.year, 1, 1), date(ref.year + 1, 1, 1)
67  
68      # mode == "custom": ผู้ใช้เลือกช่วงเอง ต้องตรวจค่าที่ส่งมาก่อน
69      if start is None or end is None:
70          raise ValueError("โหมด custom ต้องส่ง start และ end (YYYY-MM-DD)")
71      if start > end:
72          raise ValueError("start ต้องไม่เกิน end")
73      if (end - start).days + 1 > MAX_CUSTOM_DAYS:  # +1 เพราะนับรวมวันสุดท้ายด้วย
74          raise ValueError(f"เลือกช่วงได้ไม่เกิน {MAX_CUSTOM_DAYS} วัน")
75      # end ที่ผู้ใช้ส่งมา "รวม" วันนั้น จึง +1 วันให้เป็นแบบครึ่งเปิดเหมือนโหมดอื่น
76      return start, end + timedelta(days=1)
```

> - **ทำอะไร:** แปลง "โหมด + วันอ้างอิง" เป็นช่วงวัน `(start, end)`
> - **รับ:** `mode` (Mode), `ref` (date) และสำหรับ custom: `start`, `end` (date หรือ None)
> - **คืน:** `tuple[date, date]` แบบ **ครึ่งเปิด** `[start, end)` = รวม start ไม่รวม end · หรือโยน `ValueError` ถ้าค่าผิด
> - **ทำไมครึ่งเปิด:** ไม่ต้องรู้ว่าเดือนมี 28/29/30/31 วัน (ใช้วันที่ 1 ของเดือนถัดไปเป็น end) และช่วงที่ต่อกันไม่ทับกัน สลิปเที่ยงคืนพอดีจะอยู่วันเดียวเท่านั้น
> - **ใครเรียก:** `store.summarize()` L94, `reports.summary()` L109 · **คืนให้:** `build_summary()` และ `store.load_slips()`

- `L46–48` ประกาศฟังก์ชัน · `start: date | None = None` = รับ date หรือ None (`|` = "หรือ" ใน type) ค่าเริ่มต้น None (ไม่ต้องส่งก็ได้) · `-> tuple[date, date]` = บอกว่าคืน tuple ของ date 2 ตัว (type hint ไม่บังคับตอนรัน แต่ช่วย editor และคนอ่าน)
- `L49–55` docstring อธิบายแบบครึ่งเปิดพร้อมตัวอย่าง
- `L56` `if mode == "daily":`
- `L57–58` คืน `(ref, ref + 1 วัน)` เช่น ref = 18 พ.ย. → `(18 พ.ย., 19 พ.ย.)` = ทั้งวันที่ 18 · `return` จบฟังก์ชันทันที บรรทัดล่างไม่ทำ
- `L59` `if mode == "monthly":` (ไม่ต้องใช้ `elif` เพราะบรรทัดก่อนหน้า `return` ไปแล้ว)
- `L60` `ref.replace(day=1)` — date แก้ค่าในตัวไม่ได้ (immutable) `replace` สร้าง date ใหม่ที่เปลี่ยนเฉพาะช่องที่ระบุ: 18 พ.ย. → 1 พ.ย.
- `L61` คอมเมนต์
- `L62` `nxt = A if เงื่อนไข else B` = conditional expression (if บรรทัดเดียว) · ถ้าเป็นเดือน 12 → 1 ม.ค. ปีหน้า ไม่งั้นเดือน +1 · **ถ้าเขียนแค่ `first.replace(month=first.month + 1)`:** เดือนธันวาคมจะ `ValueError: month must be in 1..12` (มีเทสต์ `test_december_rolls_into_next_year` จับไว้) · ตั้งชื่อ `nxt` ไม่ใช้ `next` เพราะ `next` เป็นฟังก์ชันในตัวของ Python
- `L63` คืน `(1 พ.ย., 1 ธ.ค.)`
- `L64–66` yearly: `date(ปี, 1, 1)` ถึง `date(ปี+1, 1, 1)`
- `L68` ถึงบรรทัดนี้ได้แปลว่าไม่ใช่ 3 โหมดแรก = custom (Literal รับประกันว่าไม่มีค่าอื่น)
- `L69–70` `if start is None or end is None:` ใช้ `is None` (ตรวจว่าเป็น None จริงๆ) ไม่ใช้ `not start` · `raise ValueError(...)` = โยน error พร้อมข้อความ หยุดฟังก์ชัน · reports.py จะจับแล้วตอบ 422
- `L71–72` start ต้องไม่เกิน end (date เทียบ `>` ได้ตรงๆ)
- `L73` `(end - start)` ลบ date กันได้ `timedelta` · `.days` = จำนวนวัน · `+ 1` เพราะผู้ใช้ส่ง end แบบ **รวมวันนั้น** เช่น 1–3 ม.ค. ลบกันได้ 2 แต่จริงคือ 3 วัน
- `L74` `f"...{MAX_CUSTOM_DAYS}..."` = f-string ใส่ค่าตัวแปรในข้อความ
- `L75–76` แปลง end ของผู้ใช้ (รวมวันนั้น) เป็นครึ่งเปิด (+1 วัน) โค้ดถัดไปทุกตัวจึงใช้กติกาเดียวกัน · **ถ้าลืม +1:** วันสุดท้ายที่ผู้ใช้เลือกจะหายไปจากยอด

#### 3.2.5 `bucket_labels()` และ `bucket_of()` — ช่องของกราฟ

`backend/app/api/summary.py` · บรรทัด 79–96

```python
79  def bucket_labels(mode: Mode, start: date, end: date) -> list[str]:
80      """ชื่อช่องของกราฟ: รายวัน = ชั่วโมง, รายเดือน/ช่วง = วัน, รายปี = เดือน"""
81      if mode == "daily":
82          return [f"{h:02d}:00" for h in range(24)]  # "00:00" ... "23:00"
83      if mode == "yearly":
84          return [f"{start.year}-{m:02d}" for m in range(1, 13)]  # "2026-01" ... "2026-12"
85      # monthly / custom: หนึ่งช่องต่อหนึ่งวัน รูปแบบ "2026-11-18"
86      return [(start + timedelta(days=i)).isoformat() for i in range((end - start).days)]
87  
88  
89  def bucket_of(mode: Mode, slip: SlipRecord) -> str:
90      """สลิปใบนี้ตกอยู่ช่องไหนของกราฟ (ต้องคืนชื่อแบบเดียวกับ bucket_labels)"""
91      local = slip.created_at.astimezone(BKK)  # แปลงเป็นเวลาไทยก่อนเสมอ
92      if mode == "daily":
93          return f"{local.hour:02d}:00"
94      if mode == "yearly":
95          return f"{local.year}-{local.month:02d}"
96      return local.date().isoformat()
```

> - **`bucket_labels`** รับ `(mode, start, end)` คืน **ชื่อช่องทั้งหมด** (list ของ str) เรียงตามลำดับ
> - **`bucket_of`** รับ `(mode, slip)` คืน **ชื่อช่องของสลิป 1 ใบ** (str)
> - **ความสัมพันธ์:** 2 ฟังก์ชันนี้ต้องสร้างชื่อ **รูปแบบเดียวกันเป๊ะ** เพราะ `build_summary` ใช้ชื่อจาก `bucket_of` ไปหา key ใน dict ที่สร้างจาก `bucket_labels` ถ้ารูปแบบต่าง (เช่น `"9:00"` vs `"09:00"`) จะ `KeyError`

- `L79–80` ประกาศฟังก์ชัน + docstring
- `L81–82` daily → `[f"{h:02d}:00" for h in range(24)]` — list comprehension (สร้าง list ด้วยการวนในบรรทัดเดียว) · `range(24)` = 0..23 · `:02d` = จำนวนเต็มกว้าง 2 หลัก เติม 0 ข้างหน้า → `"00:00"`, `"01:00"` ... `"23:00"`
- `L83–84` yearly → 12 ช่อง `"2026-01"` ... `"2026-12"` (`range(1, 13)` = 1..12)
- `L85–86` monthly/custom → 1 ช่องต่อวัน · `(end - start).days` = จำนวนวันในช่วง · `.isoformat()` ของ date = `"2026-11-18"`
- `L89–90` ประกาศ `bucket_of` + docstring
- `L91` `slip.created_at.astimezone(BKK)` แปลงเวลาเป็นเวลาไทยก่อน เช่น 02:15 UTC → 09:15 ไทย · **ถ้าลบการแปลง:** ช่องกราฟจะเป็นชั่วโมง UTC (เลื่อน 7 ชม.)
- `L92–93` daily → ชั่วโมงไทย `"09:00"` (ทิ้งนาที)
- `L94–95` yearly → `"2026-11"`
- `L96` monthly/custom → `"2026-11-18"`

#### 3.2.6 `build_summary()` — รวมยอด

`backend/app/api/summary.py` · บรรทัด 99–145

```python
 99  def build_summary(slips: Iterable[SlipRecord], mode: Mode, start: date, end: date) -> dict:
100      """รวมยอดสลิปในช่วง [start, end) แล้วคืน dict ที่ API ส่งให้หน้าเว็บ
101  
102      รูปแบบที่คืน (หน้าเว็บและ summary_message ใช้รูปแบบนี้):
103          {
104            "mode": "daily",
105            "start": "2026-11-18", "end": "2026-11-18",   # end = วันสุดท้าย (รวมวันนั้น)
106            "total_amount": "350.00",                      # เป็นข้อความ กัน JSON ปัดเศษ
107            "slip_count": 2,                               # สลิปที่ผ่าน
108            "status_counts": {"passed": 2, "suspicious": 1, "failed": 1},
109            "series": [{"label": "09:00", "amount": "350.00", "slip_count": 2}, ...]
110          }
111      """
112      labels = bucket_labels(mode, start, end)
113      # เตรียมทุกช่องไว้เป็น 0 ก่อน ช่องที่ไม่มีสลิปจะได้แสดง 0 ไม่หายไปจากกราฟ
114      amounts = {label: Decimal("0") for label in labels}
115      counts = {label: 0 for label in labels}
116      status_counts = {v.value: 0 for v in Verdict}  # {"passed": 0, "suspicious": 0, "failed": 0}
117      total = Decimal("0")
118      passed = 0
119  
120      for slip in slips:
121          # กันข้อมูลนอกช่วงหลุดเข้ามา (เช่น ข้อมูลตัวอย่างที่คืนมาทั้งก้อน) ตัดด้วยวันตามเวลาไทย
122          local_day = slip.created_at.astimezone(BKK).date()
123          if not (start <= local_day < end):
124              continue
125          status_counts[slip.verdict.value] += 1  # นับทุกสถานะ
126          if slip.verdict != Verdict.PASSED:
127              continue  # ⚠️ / ❌ ไม่นับเป็นเงินที่ได้รับ
128          label = bucket_of(mode, slip)
129          amounts[label] += slip.amount
130          counts[label] += 1
131          total += slip.amount
132          passed += 1
133  
134      return {
135          "mode": mode,
136          "start": start.isoformat(),
137          "end": (end - timedelta(days=1)).isoformat(),  # แปลงกลับเป็นวันสุดท้าย (รวมวันนั้น) ให้คนอ่านง่าย
138          "total_amount": f"{total:.2f}",  # ทศนิยม 2 ตำแหน่งเสมอ เช่น "350.00"
139          "slip_count": passed,
140          "status_counts": status_counts,
141          "series": [
142              {"label": label, "amount": f"{amounts[label]:.2f}", "slip_count": counts[label]}
143              for label in labels
144          ],
145      }
```

> - **ทำอะไร:** วนสลิปทีละใบ นับสถานะทุกใบ แล้วบวกยอดเงินเฉพาะใบที่ผ่านเข้าช่องกราฟและยอดรวม
> - **รับ:** `slips` (Iterable ของ SlipRecord), `mode`, `start`, `end` (ครึ่งเปิดจาก `period_range`)
> - **คืน:** dict (รูปแบบใน docstring L102–110) ซึ่งเป็น **สัญญากับหน้าเว็บ** (type `Summary` ใน page.tsx L37–45) และกับ `summary_message()`
> - **ใครเรียก:** `store.summarize()` L95, `reports.summary()` L121

- `L99` ประกาศฟังก์ชัน คืน `dict`
- `L100–111` docstring แสดงรูปแบบผลลัพธ์จริง
- `L112` เอาชื่อช่องทั้งหมดมาก่อน
- `L113` คอมเมนต์
- `L114` `{label: Decimal("0") for label in labels}` — dict comprehension สร้าง dict ที่ทุกช่องเป็น 0 · ใช้ `Decimal("0")` (ข้อความ) ไม่ใช่ `Decimal(0.0)` เพื่อไม่ผ่าน float · **ทำไมเตรียมทุกช่องก่อน:** ชั่วโมงที่ไม่มีสลิปยังต้องมีช่อง (ยอด 0) กราฟจะไม่ขาดช่วง · **ถ้าลบ:** L129 `amounts[label] += ...` จะ `KeyError` และกราฟจะมีแค่ช่องที่มีสลิป
- `L115` ตัวนับจำนวนสลิปที่ผ่านต่อช่อง
- `L116` `{v.value: 0 for v in Verdict}` — วน enum ได้ทุกค่า · `.value` = ข้อความ `"passed"` · ได้ `{"passed": 0, "suspicious": 0, "failed": 0}` · ถ้าวันหลังเพิ่มสถานะใน contracts บรรทัดนี้รองรับเอง
- `L117` `total` ยอดรวม · `L118` `passed` จำนวนใบที่ผ่าน
- `L120` `for slip in slips:` วนทีละใบ
- `L121` คอมเมนต์
- `L122` วันตามเวลาไทยของสลิป
- `L123–124` `if not (start <= local_day < end): continue` — Python เขียนเทียบ 2 ด้านต่อกันได้ (`a <= x < b`) · `continue` = ข้ามไปใบถัดไป · **ทำไมต้องมี ทั้งที่ store.py กรองใน SQL แล้ว:** กันแหล่งข้อมูลที่คืนมาทั้งก้อน (เทสต์ส่ง SLIPS ทุกใบ, sample_data) และเป็นตาข่ายชั้นที่ 2 · **ถ้าลบ:** สลิปนอกช่วงจะถูกนับ หรือ `bucket_of` คืนชื่อช่องที่ไม่มีใน dict → `KeyError`
- `L125` `status_counts[slip.verdict.value] += 1` นับ **ทุกสถานะ** (`+=` = บวกเพิ่มเข้าไปในตัวเดิม)
- `L126–127` ถ้าไม่ใช่ PASSED → `continue` ไม่นับเงิน · **กติกาธุรกิจหลัก:** ยอดรับ = เงินที่ร้านได้จริง ⚠️ ยังไม่ยืนยัน ❌ คือปลอม · **ถ้าลบ 2 บรรทัดนี้:** ยอดบนเว็บจะรวมสลิปปลอมด้วย
- `L128` หาช่องของสลิป
- `L129–130` บวกยอด/จำนวนเข้าช่อง
- `L131–132` บวกยอดรวม/จำนวนรวม
- `L134` เริ่มสร้าง dict ผลลัพธ์
- `L135` `"mode"` ส่งโหมดกลับ (summary_message ใช้เลือกหัวข้อ)
- `L136` `start.isoformat()` แปลง date เป็นข้อความ (JSON ไม่รู้จัก date)
- `L137` `end - 1 วัน` แปลงกลับเป็น "วันสุดท้ายแบบรวม" ให้คนอ่านเข้าใจง่าย เช่น เดือน พ.ย. → `"2026-11-30"` (ไม่ใช่ `"2026-12-01"`)
- `L138` `f"{total:.2f}"` ทศนิยม 2 ตำแหน่ง และส่งเป็น **ข้อความ** เพราะ JSON แปลงตัวเลขเป็น float ในฝั่ง JavaScript (เพี้ยนได้) หน้าเว็บค่อยแปลงตอนแสดงผล
- `L139` จำนวนสลิปที่ผ่าน (int)
- `L140` dict นับสถานะ
- `L141–144` list comprehension สร้าง series เรียงตามลำดับช่อง แต่ละช่องมี label/amount (ข้อความ)/slip_count
- `L145` ปิด dict

**ตัวอย่างเดินโค้ด** (ข้อมูลเดียวกับ `tests/test_reports.py`): วันที่ 18 พ.ย. มี ✅100 (09:15), ✅250 (09:40), ⚠️999 (12:00), ❌500 (20:00)

```
L116 status_counts = {passed:0, suspicious:0, failed:0}
ใบ 1 ✅100 09:15 → L125 passed=1 → L128 "09:00" → amounts["09:00"]=100, total=100, passed=1
ใบ 2 ✅250 09:40 → passed=2 → "09:00" → amounts["09:00"]=350, total=350, passed=2
ใบ 3 ⚠️999      → suspicious=1 → L126 continue (ไม่นับเงิน)
ใบ 4 ❌500      → failed=1     → L126 continue
ผล: total_amount "350.00", slip_count 2, status_counts {2,1,1}, series ช่อง 09:00 = "350.00" 2 ใบ
```

#### 3.2.7 `_thai_date()` และ `summary_message()` — ข้อความ LINE

`backend/app/api/summary.py` · บรรทัด 148–179

```python
148  def _thai_date(d: date) -> str:
149      """วันที่แบบไทย พ.ศ. เช่น 18/11/2569 (พ.ศ. = ค.ศ. + 543)"""
150      return f"{d.day}/{d.month}/{d.year + 543}"
151  
152  
153  def summary_message(summary: dict) -> str:
154      """ข้อความสรุปยอดสำหรับตอบแม่ค้าใน LINE (แพรวเรียกใช้หลัง summarize())
155  
156      ตัวอย่างผลลัพธ์:
157          📊 ยอดวันที่ 18/11/2569
158          ฿350.00 จาก 2 สลิปที่ผ่าน
159          ⚠️ รอตรวจ 1 ใบ · ❌ ไม่ผ่าน 1 ใบ
160      """
161      start = date.fromisoformat(summary["start"])
162      mode = summary["mode"]
163      # หัวข้อเปลี่ยนตามโหมด
164      if mode == "daily":
165          title = f"ยอดวันที่ {_thai_date(start)}"
166      elif mode == "monthly":
167          title = f"ยอดเดือน {start.month}/{start.year + 543}"
168      elif mode == "yearly":
169          title = f"ยอดปี {start.year + 543}"
170      else:
171          title = f"ยอด {_thai_date(start)} – {_thai_date(date.fromisoformat(summary['end']))}"
172  
173      counts = summary["status_counts"]
174      return (
175          f"📊 {title}\n"
176          # :,.2f = ใส่ลูกน้ำหลักพัน + ทศนิยม 2 ตำแหน่ง เช่น 12,500.00
177          f"฿{Decimal(summary['total_amount']):,.2f} จาก {summary['slip_count']} สลิปที่ผ่าน\n"
178          f"⚠️ รอตรวจ {counts['suspicious']} ใบ · ❌ ไม่ผ่าน {counts['failed']} ใบ"
179      )
```

> - **`_thai_date`** รับ date คืน `"18/11/2569"` · ขึ้นต้นด้วย `_` = ธรรมเนียม Python ว่า "ใช้ภายในไฟล์" (ไม่ได้บังคับ)
> - **`summary_message`** รับ dict จาก `build_summary` คืนข้อความ 3 บรรทัดสำหรับตอบแม่ค้าใน LINE
> - **ทำไมรับ dict ไม่รับ session:** แยก "หาตัวเลข" ออกจาก "จัดข้อความ" เทสต์ข้อความได้โดยไม่ต้องมีฐานข้อมูล
> - **ใครเรียก:** `webhook.summary_text()` ของแพรว (L143–144 ใน webhook.py)

- `L148–150` `d.year + 543` = แปลง ค.ศ. เป็น พ.ศ. · `d.day` ไม่เติม 0 (ได้ `1/11/2569` ตามที่คนไทยเขียนทั่วไป)
- `L153–160` ประกาศฟังก์ชัน + docstring มีตัวอย่างผลลัพธ์
- `L161` `date.fromisoformat(...)` แปลงข้อความ `"2026-11-18"` กลับเป็น date
- `L162` อ่านโหมด
- `L163–171` เลือกหัวข้อตามโหมด (`if / elif / else`) · `L171` custom แสดงช่วง `ยอด 1/11/2569 – 30/11/2569` (f-string ข้างในใช้ `'end'` เครื่องหมายเดี่ยว เพราะข้างนอกใช้ `"` แล้ว)
- `L173` ดึง dict นับสถานะ
- `L174–179` วงเล็บครอบหลาย f-string = Python **ต่อข้อความให้เอง** (implicit concatenation) · `\n` = ขึ้นบรรทัดใหม่
- `L176` คอมเมนต์
- `L177` `Decimal(summary['total_amount'])` แปลงข้อความเป็นตัวเลขก่อน แล้ว `:,.2f` = ใส่ `,` คั่นหลักพัน + ทศนิยม 2 ตำแหน่ง → `12,500.00` · **ถ้าใส่ข้อความตรงๆ:** จะได้ `12500.00` ไม่มีลูกน้ำ (เทสต์ `test_summary_message_uses_thousands_separator` จะแดง)
- `L178` บรรทัดจำนวน ⚠️ และ ❌

<a id="f-store"></a>
### 3.3 `backend/app/api/store.py` — อ่านจากตาราง `slips` ของวี

> **ภาพรวมไฟล์**
> - **ทำอะไร:** ไฟล์เดียวใน `app/api/` ที่คุยกับฐานข้อมูล: อ่านสลิปของร้านในช่วงวัน แปลงเป็น `SlipRecord` ส่งให้ summary.py · อ่านสลิปล่าสุดสำหรับตารางบนเว็บ · มี `summarize()` ให้แพรวเรียก
> - **ข้อมูลเข้า:** `Session` (การเชื่อมต่อฐานข้อมูล), `shop_id`, ช่วงวัน
> - **ต่อกับ:** ตาราง `Slip` ของวี (`app/verify/models.py`) ← อ่าน · `summary.py` ← ส่งต่อให้คำนวณ · `reports.py`, `webhook.py` ← เรียกใช้

#### 3.3.1 Docstring

`backend/app/api/store.py` · บรรทัด 1–19

```python
 1  """อ่านสลิปจากตาราง slips (ของวี) มาสรุปยอด (แอ๊น)
 2  
 3  ไฟล์นี้เป็นส่วนเดียวของ app/api/ ที่คุยกับฐานข้อมูล ส่วนการคำนวณอยู่ใน summary.py
 4  
 5  summarize() คือฟังก์ชันที่แพรวเรียกตอนแม่ค้ากด "ยอดวันนี้ / ยอดเดือนนี้ / ยอดปีนี้" ใน LINE:
 6  
 7      from app.api.store import summarize
 8      from app.api.summary import summary_message
 9  
10      with Session(get_engine()) as session:
11          result = summarize(session, shop.id, "day")
12      reply(summary_message(result))
13  
14  กติกาที่ต่างจากข้อมูลตัวอย่าง:
15  - ถ้าแม่ค้ากดตรวจเองแล้ว (merchant_verdict) ใช้ผลของแม่ค้าแทนผลของระบบ
16  - สลิปที่อ่านยอดไม่ได้ (amount เป็น NULL) นับจำนวนสลิป แต่ยอดเงินนับเป็น 0
17  - เวลาใช้ received_at (เวลาที่ระบบได้รับสลิป) ไม่ใช่เวลาโอนในสลิป
18    เพราะเวลาในสลิปมาจาก OCR ซึ่งอาจอ่านผิด/ถูกแก้ได้ ส่วน received_at ระบบเป็นคนใส่เอง
19  """
```

- `L1–19` อธิบายหน้าที่ ตัวอย่างการเรียกของแพรว (L7–12) และกติกา 3 ข้อที่ต่างจากข้อมูลตัวอย่าง: ผลแม่ค้าชนะผลระบบ / ยอดอ่านไม่ได้นับ 0 / ใช้ `received_at` ไม่ใช่เวลาในสลิป · ไม่มีผลต่อการทำงาน

#### 3.3.2 Import และตัวแปลงคำ

`backend/app/api/store.py` · บรรทัด 21–34

```python
21  from datetime import date, datetime, time, timezone
22  from decimal import Decimal
23  from typing import Literal
24  
25  from sqlalchemy import select
26  from sqlalchemy.orm import Session
27  
28  from app.api.summary import BKK, SlipRecord, build_summary, period_range
29  from app.core.contracts import Verdict
30  from app.verify.models import Slip
31  
32  # คำที่แพรวส่งมา ("day") แปลงเป็นโหมดของ API ("daily") จะได้ใช้ฟังก์ชันชุดเดียวกัน
33  Period = Literal["day", "month", "year"]
34  PERIOD_TO_MODE = {"day": "daily", "month": "monthly", "year": "yearly"}
```

- `L21` `time` = เวลาของวัน (ไม่มีวันที่) ใช้ `time.min` = 00:00 ใน L42
- `L22` `Decimal` ใช้ตอนยอดเป็น NULL (L64)
- `L23` `Literal` ใช้สร้าง type `Period`
- `L25` `select` = สร้างคำสั่ง SELECT แบบ SQLAlchemy 2 · ข้อดี: ค่าตัวแปรถูกส่งแยกเป็นพารามิเตอร์ จึง **กัน SQL injection** (ไม่ได้เอาข้อความไปต่อเป็น SQL เอง)
- `L26` `Session` = การคุยกับฐานข้อมูล 1 รอบ (ถือ connection + transaction)
- `L28` ใช้ของจาก summary.py ซ้ำ ไม่เขียนสูตรซ้ำ (หลัก DRY: Don't Repeat Yourself)
- `L29` `Verdict` แปลงข้อความในฐานข้อมูลเป็น enum (L65)
- `L30` `Slip` = คลาสที่แทนตาราง `slips` ของวี (ORM: Object-Relational Mapping = แถวในตาราง ↔ object ใน Python)
- `L32` คอมเมนต์
- `L33` `Period = Literal["day", "month", "year"]` — คำที่ตกลงกับแพรวในแผนทีม (`summarize(session, shop_id, period: "day"|"month"|"year")`)
- `L34` dict แปลงคำของแพรว → โหมดของ API · **ทำไม:** ใช้ `period_range`/`build_summary` ชุดเดียวกับหน้าเว็บ ไม่ต้องเขียนสองชุด · **ถ้าแพรวส่งคำอื่น เช่น "week":** L92 `KeyError` → webhook จับไว้แล้วตอบ "ยังดูยอดไม่ได้" (ดู 3.9)

#### 3.3.3 ตัวช่วยเรื่องเวลาและผลตัดสิน

`backend/app/api/store.py` · บรรทัด 37–56

```python
37  def _utc_midnight(day: date) -> datetime:
38      """เที่ยงคืนเวลาไทยของวันนั้น แปลงเป็น UTC (ฐานข้อมูลเทียบเวลาแบบ UTC)
39  
40      เช่น 18 พ.ย. 00:00 เวลาไทย = 17 พ.ย. 17:00 UTC
41      """
42      return datetime.combine(day, time.min, BKK).astimezone(timezone.utc)
43  
44  
45  def _as_utc(value: datetime) -> datetime:
46      """ใส่ timezone ให้เวลาจากฐานข้อมูล
47  
48      Postgres คืนเวลาที่มี timezone อยู่แล้ว แต่ SQLite (ใช้ในเทสต์) คืนเวลาไม่มี timezone
49      ซึ่งจริงๆ เป็น UTC → ใส่ UTC ให้ ไม่งั้น astimezone() จะเข้าใจผิดว่าเป็นเวลาเครื่อง
50      """
51      return value if value.tzinfo is not None else value.replace(tzinfo=timezone.utc)
52  
53  
54  def _final_verdict(slip: Slip) -> str:
55      """ผลสุดท้ายของสลิป: แม่ค้ากดตรวจเองแล้วใช้ผลของแม่ค้า ไม่งั้นใช้ผลของระบบ"""
56      return slip.merchant_verdict or slip.verdict
```

> **`_utc_midnight(day)`** รับ date คืน datetime UTC ของเที่ยงคืนไทยวันนั้น · **`_as_utc(value)`** รับ datetime คืน datetime ที่มี timezone แน่นอน · **`_final_verdict(slip)`** รับแถว Slip คืนข้อความผลสุดท้าย

- `L37` ประกาศ รับ `date` คืน `datetime`
- `L38–41` docstring พร้อมตัวอย่าง 18 พ.ย. 00:00 ไทย = 17 พ.ย. 17:00 UTC
- `L42` `datetime.combine(day, time.min, BKK)` = เอาวันที่ + 00:00 + เขตเวลาไทย มารวมเป็น datetime · `.astimezone(timezone.utc)` = แปลงเป็นเวลาเดียวกันในเขต UTC · **ทำไม:** Postgres เก็บ `received_at` เป็น `timestamptz` เทียบแบบเวลาจริง (instant) การส่งขอบเขตเป็น UTC ชัดเจนที่สุด · **ถ้าใช้เที่ยงคืน UTC แทน:** สลิป 00:00–06:59 ไทยจะไปอยู่วันก่อน (เทสต์ `test_day_is_cut_by_thai_time` จะแดง)
- `L45–50` docstring: Postgres คืนเวลาที่มี timezone แต่ **SQLite** (ใช้ในเทสต์) คืนเวลาไม่มี timezone ซึ่งที่จริงเป็น UTC
- `L51` `value if value.tzinfo is not None else value.replace(tzinfo=timezone.utc)` — มี timezone แล้วคืนตัวเดิม ไม่มีก็ **ติดป้าย** UTC (`replace` ไม่เปลี่ยนตัวเลขเวลา ต่างจาก `astimezone` ที่แปลงเวลา) · **ถ้าลบ:** เทสต์ใน SQLite จะแปลงเวลาผิดตามเขตเวลาของเครื่องที่รันเทสต์
- `L54–55` ประกาศ + docstring
- `L56` `slip.merchant_verdict or slip.verdict` — ตัวดำเนินการ `or` ของ Python คืน **ค่าตัวแรกที่เป็นจริง**: ถ้า `merchant_verdict` เป็น None (แม่ค้ายังไม่กด) คืน `verdict` ของระบบ · **ทำไม:** ระบบให้ ⚠️ แต่แม่ค้าเช็กแอปธนาคารแล้วว่าเงินเข้า ควรนับยอด (เทสต์ `test_merchant_review_overrides_system_verdict`) · **ถ้าใช้แค่ `slip.verdict`:** ปุ่ม "ร้านตรวจแล้ว" (ถ้าทำในอนาคต) จะไม่มีผลกับยอด

#### 3.3.4 `to_record()` — แปลงแถว → SlipRecord

`backend/app/api/store.py` · บรรทัด 59–67

```python
59  def to_record(slip: Slip) -> SlipRecord:
60      """แปลงแถวในตาราง slips → SlipRecord ที่ summary.py ใช้คำนวณ"""
61      return SlipRecord(
62          trans_ref=slip.trans_ref or "",  # NULL → ข้อความว่าง
63          bank=slip.bank or "",
64          amount=slip.amount if slip.amount is not None else Decimal("0"),  # อ่านยอดไม่ได้ = 0
65          verdict=Verdict(_final_verdict(slip)),
66          created_at=_as_utc(slip.received_at),
67      )
```

> **รับ:** แถว `Slip` 1 แถว · **คืน:** `SlipRecord` · **ทำไม:** แปลงคอลัมน์ที่อาจเป็น NULL ให้เป็นค่าที่ summary.py ใช้ได้ และใส่กติกา "ผลแม่ค้าก่อน"

- `L59–60` ประกาศ + docstring
- `L61` สร้าง SlipRecord (ใช้ keyword argument `ชื่อ=ค่า` อ่านง่ายและสลับลำดับได้)
- `L62` `slip.trans_ref or ""` — NULL (None) → ข้อความว่าง เพราะช่อง `trans_ref` ของ SlipRecord เป็น `str`
- `L63` ธนาคารแบบเดียวกัน
- `L64` `slip.amount if slip.amount is not None else Decimal("0")` — ยอดอ่านไม่ได้ → 0 บาท แต่สลิปยังนับเป็น 1 ใบ (เทสต์ `test_passed_slip_without_amount_counts_as_zero`) · ใช้ `is not None` ไม่ใช้ `or` เพราะถ้ายอดเป็น `Decimal("0")` จริงๆ `or` ก็ให้ผลเดียวกัน แต่ `is not None` สื่อความหมายชัดกว่า · **ข้อจำกัด:** สลิปที่ผ่านแต่อ่านยอดไม่ได้ ทำให้ยอดบนเว็บต่ำกว่าเงินจริง
- `L65` `Verdict(_final_verdict(slip))` — เรียก enum ด้วยข้อความได้ enum ตัวนั้น ถ้าข้อความไม่ใช่ 3 ค่านี้จะ `ValueError` ทันที (ดีกว่าคำนวณผิดเงียบๆ)
- `L66` ใช้ `received_at` (เวลาที่ฐานข้อมูลบันทึก `server_default=func.now()`) **ไม่ใช่** `transferred_at` (เวลาในสลิปจาก OCR ซึ่งอ่านผิดหรือถูกแก้ได้) · ผ่าน `_as_utc` ให้มี timezone เสมอ
- `L67` ปิดวงเล็บ

#### 3.3.5 `load_slips()` — query ฐานข้อมูล

`backend/app/api/store.py` · บรรทัด 70–83

```python
70  def load_slips(session: Session, shop_id: str, start: date, end: date) -> list[SlipRecord]:
71      """สลิปของร้านนี้ที่ได้รับตั้งแต่ start ถึงก่อน end (ตัดวันตามเวลาไทย)
72  
73      กรองในฐานข้อมูลเลย (WHERE) ไม่ดึงทั้งตารางมากรองใน Python
74      และมี index ix_slips_shop_received (shop_id, received_at) ของวีช่วยให้ query เร็ว
75      """
76      rows = session.scalars(
77          select(Slip).where(
78              Slip.shop_id == shop_id,
79              Slip.received_at >= _utc_midnight(start),
80              Slip.received_at < _utc_midnight(end),
81          )
82      )
83      return [to_record(row) for row in rows]
```

> **รับ:** session, shop_id, start, end (ครึ่งเปิด) · **คืน:** list ของ SlipRecord · **ใครเรียก:** `summarize()` L95, `reports.db_source()` L71

- `L70` ประกาศ
- `L71–75` docstring: กรองใน SQL + มี index ช่วย
- `L76` `session.scalars(...)` รัน query แล้วคืน **object ของคอลัมน์แรก** ทีละแถว (ที่นี่คือ object `Slip` ทั้งแถว) ต่างจาก `session.execute` ที่คืนเป็น tuple
- `L77` `select(Slip)` = `SELECT slips.* FROM slips`
- `L78` `Slip.shop_id == shop_id` — ไม่ใช่การเทียบแบบ Python ธรรมดา SQLAlchemy แปลงเป็นเงื่อนไข SQL `shop_id = :param` · **ถ้าลบ:** ยอดของทุกร้านปนกัน (เทสต์ `test_day_counts_only_passed_slips_of_this_shop` จะแดง)
- `L79` `received_at >= เที่ยงคืนไทยของวันเริ่ม (เป็น UTC)`
- `L80` `received_at < เที่ยงคืนไทยของวัน end` (ใช้ `<` = ครึ่งเปิด)
- `L81–82` ปิดวงเล็บ · SQL ที่ได้ประมาณ: `SELECT * FROM slips WHERE shop_id = 'shop-1' AND received_at >= '2026-11-17 17:00+00' AND received_at < '2026-11-18 17:00+00'`
- `L83` แปลงทุกแถวด้วย list comprehension
- **ทำไมกรองใน SQL ไม่ดึงมากรองใน Python:** ถ้ามีหลายหมื่นแถว การดึงทั้งตารางช้าและกินหน่วยความจำ ตารางของวีมี index `ix_slips_shop_received (shop_id, received_at)` ตรงกับเงื่อนไขนี้พอดี ฐานข้อมูลจึงกระโดดไปหาแถวได้โดยไม่ต้องอ่านทั้งตาราง

#### 3.3.6 `summarize()` — ฟังก์ชันที่แพรวเรียก

`backend/app/api/store.py` · บรรทัด 86–95

```python
86  def summarize(session: Session, shop_id: str, period: Period, today: date | None = None) -> dict:
87      """ยอดวันนี้ / เดือนนี้ / ปีนี้ ของร้าน (ฟังก์ชันที่ตกลงกับแพรวไว้)
88  
89      คืน dict เดียวกับ API: total_amount (ข้อความ เช่น "3098.00"), slip_count (สลิปที่ผ่าน),
90      status_counts, series · today ใส่ได้ตอนเทสต์ ไม่ใส่ = วันนี้ตามเวลาไทย
91      """
92      mode = PERIOD_TO_MODE[period]
93      ref = today or datetime.now(BKK).date()
94      start, end = period_range(mode, ref)
95      return build_summary(load_slips(session, shop_id, start, end), mode, start, end)
```

> **รับ:** session (คนเรียกเปิดให้), shop_id, period (`"day"|"month"|"year"`), today (ไม่บังคับ) · **คืน:** dict เดียวกับ API · **ใครเรียก:** `webhook.summary_text()` ของแพรว และเทสต์

- `L86` ชื่อ/พารามิเตอร์ตรงกับที่ตกลงในแผนทีม · `today: date | None = None` ใส่ตอนเทสต์จะได้ล็อกวัน (ไม่งั้นผลเทสต์เปลี่ยนทุกวัน)
- `L87–91` docstring
- `L92` แปลง `"day"` → `"daily"`
- `L93` `today or datetime.now(BKK).date()` ไม่ส่งมา = วันนี้ตามเวลาไทย · **ถ้าใช้ `date.today()`:** จะได้วันของเขตเวลาเครื่อง ใน docker (UTC) ช่วง 00:00–06:59 ไทยจะได้ "เมื่อวาน"
- `L94` หาช่วง
- `L95` อ่านสลิปในช่วง → คำนวณ (เรียกต่อกัน 2 ฟังก์ชันในบรรทัดเดียว)
- ฟังก์ชันนี้ **ไม่เปิด/ปิด/commit session เอง** เพราะแค่อ่าน คนเรียก (webhook) เปิดด้วย `open_session()` ของวี

#### 3.3.7 `recent_item()` และ `load_recent()` — สลิปล่าสุด

`backend/app/api/store.py` · บรรทัด 98–124

```python
 98  def recent_item(slip: Slip) -> dict:
 99      """แปลงแถวในตาราง slips → 1 แถวในตาราง "สลิปล่าสุด" บนหน้าเว็บ
100  
101      ไม่ส่ง line_user_id / เลขอ้างอิงเต็มออกไป เพราะหน้าเว็บไม่ได้ใช้ (ส่งเท่าที่จำเป็น)
102      """
103      return {
104          # เวลาไทยแบบ ISO เช่น "2026-11-19T21:05+07:00" หน้าเว็บเอาไปจัดรูปแบบต่อ
105          "received_at": _as_utc(slip.received_at).astimezone(BKK).isoformat(timespec="minutes"),
106          "bank": slip.bank,  # อาจเป็น None ถ้า OCR อ่านไม่ได้
107          "amount": f"{slip.amount:.2f}" if slip.amount is not None else None,
108          "verdict": _final_verdict(slip),
109          "fraud_score": slip.fraud_score,
110          "reasons": list(slip.reasons or []),  # เหตุผลจาก CheckResult เช่น "สลิปนี้ถูกใช้ไปแล้ว"
111          "reviewed": slip.merchant_verdict is not None,  # แม่ค้ากดตรวจเองแล้วหรือยัง
112      }
113  
114  
115  def load_recent(session: Session, shop_id: str, limit: int = 10) -> list[dict]:
116      """สลิปล่าสุดของร้าน เรียงจากใหม่ไปเก่า (ใช้ตอน demo: ส่งสลิปใน LINE แล้วเห็นขึ้นบนเว็บ)"""
117      rows = session.scalars(
118          select(Slip)
119          .where(Slip.shop_id == shop_id)
120          # id ช่วยเรียงกรณีเวลาเท่ากันเป๊ะ ผลจะได้ออกมาลำดับเดิมทุกครั้ง
121          .order_by(Slip.received_at.desc(), Slip.id.desc())
122          .limit(limit)
123      )
124      return [recent_item(row) for row in rows]
```

> **`recent_item`** แปลงแถว → dict 1 แถวของตาราง "สลิปล่าสุด" (ต้องตรงกับ type `RecentSlip` ใน page.tsx L48–56) · **`load_recent`** อ่าน N แถวล่าสุดของร้าน · **ใครเรียก:** `reports.db_recent_source()` L85

- `L98–102` ประกาศ + docstring: **ไม่ส่ง `line_user_id` / เลขอ้างอิงเต็มออกไป** เพราะหน้าเว็บไม่ใช้ (หลัก data minimization: ส่งเท่าที่จำเป็น ข้อมูลส่วนตัวของลูกค้าไม่หลุดออก API)
- `L103` เริ่ม dict
- `L104` คอมเมนต์
- `L105` แปลงเป็นเวลาไทย แล้ว `.isoformat(timespec="minutes")` → `"2026-11-19T21:05+07:00"` (ตัดวินาที) หน้าเว็บตัดข้อความเอา `HH:MM` ได้เลย
- `L106` ธนาคาร (อาจเป็น None)
- `L107` `f"{slip.amount:.2f}" if ... else None` ยอดเป็นข้อความ 2 ตำแหน่ง หรือ None (หน้าเว็บแสดง "อ่านไม่ได้")
- `L108` ผลสุดท้าย (ผลแม่ค้าก่อน) เหมือนตอนรวมยอด
- `L109` คะแนนเสี่ยง 0–100
- `L110` `list(slip.reasons or [])` กัน NULL และสร้าง list ใหม่ (ไม่ส่ง object ภายในของ SQLAlchemy ออกไป)
- `L111` `slip.merchant_verdict is not None` → True/False ว่าแม่ค้ากดตรวจเองแล้วหรือยัง
- `L112` ปิด dict
- `L115–116` ประกาศ `load_recent` ค่าเริ่มต้น 10 แถว
- `L117–119` `select(Slip).where(Slip.shop_id == shop_id)` เฉพาะร้านนี้ · ขึ้นบรรทัดใหม่ในวงเล็บได้ (method chaining)
- `L120` คอมเมนต์
- `L121` `.order_by(Slip.received_at.desc(), Slip.id.desc())` = `ORDER BY received_at DESC, id DESC` เรียงใหม่ → เก่า ถ้าเวลาเท่ากันเป๊ะใช้ id ตัดสิน ผลจึงออกลำดับเดิมทุกครั้ง (ไม่สลับไปมาตอนรีเฟรช) · **ถ้าลบ order_by:** ฐานข้อมูลคืนลำดับใดก็ได้ ตาราง "ล่าสุด" จะไม่ใช่ล่าสุด
- `L122` `.limit(limit)` = `LIMIT 10` ดึงเท่าที่ต้องใช้
- `L123` ปิด
- `L124` แปลงทุกแถวเป็น dict

<a id="f-reports"></a>
### 3.4 `backend/app/api/reports.py` — API ที่หน้าเว็บเรียก

> **ภาพรวมไฟล์**
> - **ทำอะไร:** เปิด endpoint HTTP 2 ตัว · อ่านพารามิเตอร์จาก URL · เรียก store/summary · แปลง error เป็น status code ที่เหมาะสม
> - **ข้อมูลเข้า:** HTTP request เช่น `GET /api/v1/reports/summary?mode=monthly&date=2026-11-18`
> - **ข้อมูลออก:** JSON (FastAPI แปลง dict เป็น JSON ให้)
> - **ต่อกับ:** `main.py` (`include_router`) · `store.py` · `summary.py` · `app.verify.db.get_engine` ของวี · `page.tsx` (ผู้เรียก)

#### 3.4.1 Docstring (คู่มือ API)

`backend/app/api/reports.py` · บรรทัด 1–20

```python
 1  """API สรุปรายรับสำหรับหน้าเว็บของแม่ค้า (แอ๊น)
 2  
 3  GET /api/v1/reports/summary
 4      ?mode=daily                       ยอดวันนี้ (แยกรายชั่วโมง)
 5      ?mode=monthly                     ยอดเดือนนี้ (แยกรายวัน)
 6      ?mode=yearly                      ยอดปีนี้ (แยกรายเดือน)
 7      ?mode=custom&start=...&end=...    ช่วงที่เลือก (แยกรายวัน, ไม่เกิน 366 วัน)
 8      &date=YYYY-MM-DD                  ดูวัน/เดือน/ปีอื่น (ค่าเริ่มต้น = วันนี้ตามเวลาไทย)
 9      &shop_id=shop-1                   ร้านไหน (ค่าเริ่มต้น = SHOP_ID ใน .env)
10  
11  GET /api/v1/reports/recent
12      ?limit=10                         สลิปล่าสุดกี่ใบ (1–50) ใช้ทำตาราง "สลิปล่าสุด" บนหน้าเว็บ
13      &shop_id=shop-1
14  
15  แหล่งข้อมูล: ตาราง slips ของวีเท่านั้น (สลิปที่ verify_slip() บันทึกหลังลูกค้าส่งใน LINE)
16  ไม่มีโหมดข้อมูลปลอมใน API → ยอดบนเว็บคือยอดจากระบบของทีมจริงเสมอ
17  (ทดสอบบนเครื่องตัวเองก่อนต่อระบบครบ: python -m app.api.seed_demo ใส่แถว DEMO- ลงตาราง แล้ว --clear ก่อน demo)
18  
19  ข้อจำกัด: ยังไม่มีระบบล็อกอิน ใครเปิด URL นี้ได้ก็เห็นยอดร้าน ใช้แค่ในเครื่อง/ตอน demo
20  """
```

- `L1–20` คู่มือการใช้ API: พารามิเตอร์ทุกตัวของ `/summary` (L3–9) และ `/recent` (L11–13) · แหล่งข้อมูลคือตาราง slips เท่านั้น ไม่มีโหมดข้อมูลปลอม (L15–17) · **ข้อจำกัดที่เขียนไว้เอง: ยังไม่มีระบบล็อกอิน** (L19) · ไม่มีผลต่อการทำงาน แต่ FastAPI ไม่ได้เอา docstring ระดับไฟล์ไปแสดงใน `/docs` (แสดงเฉพาะ docstring ของฟังก์ชัน endpoint)

#### 3.4.2 Import และ router

`backend/app/api/reports.py` · บรรทัด 22–37

```python
22  import os
23  from collections.abc import Callable
24  from datetime import date, datetime
25  from functools import lru_cache
26  
27  from fastapi import APIRouter, Depends, HTTPException, Query
28  from sqlalchemy import Engine
29  from sqlalchemy.exc import SQLAlchemyError
30  from sqlalchemy.orm import Session
31  
32  from app.api.store import load_recent, load_slips
33  from app.api.summary import BKK, Mode, SlipRecord, build_summary, period_range
34  from app.verify.db import get_engine
35  
36  # prefix = ทุก endpoint ในไฟล์นี้ขึ้นต้นด้วย /api/v1/reports · tags ใช้จัดกลุ่มในหน้า /docs
37  router = APIRouter(prefix="/api/v1/reports", tags=["reports"])
```

- `L22` `os` ใช้อ่าน env `SHOP_ID` (L65)
- `L23` `Callable` = type ของ "ฟังก์ชัน"
- `L24` `date`, `datetime`
- `L25` `lru_cache` = decorator จำผลของฟังก์ชัน (L53)
- `L27` จาก FastAPI: `APIRouter` (กลุ่ม endpoint), `Depends` (ขอให้ FastAPI เรียกฟังก์ชันแล้วส่งผลเข้ามา = dependency injection), `HTTPException` (โยนเพื่อตอบ error), `Query` (ตั้งค่าเพิ่มให้พารามิเตอร์ใน URL)
- `L28` `Engine` = ตัวจัดการการเชื่อมต่อฐานข้อมูล (มี connection pool)
- `L29` `SQLAlchemyError` = คลาสแม่ของ error ฐานข้อมูลทุกแบบ (ต่อไม่ได้, ไม่มีตาราง, ...)
- `L30` `Session`
- `L32–34` ฟังก์ชันของแอ๊นเอง + `get_engine` ของวี
- `L36` คอมเมนต์
- `L37` `router = APIRouter(prefix="/api/v1/reports", tags=["reports"])` — ทุก endpoint ในไฟล์ขึ้นต้น `/api/v1/reports` · `v1` = เวอร์ชัน ถ้าวันหลังเปลี่ยนรูปแบบใหญ่ทำ v2 ได้โดยหน้าเว็บเก่าไม่พัง · `tags` จัดกลุ่มในหน้า `http://localhost:8000/docs` · **ถ้าลบ prefix:** URL จะกลายเป็น `/summary` หน้าเว็บที่เรียก `/api/v1/reports/summary` จะได้ 404

#### 3.4.3 Type ของแหล่งข้อมูล + ข้อความ error

`backend/app/api/reports.py` · บรรทัด 39–50

```python
39  # แหล่งข้อมูล = ฟังก์ชันที่รับ (shop_id, วันเริ่ม, วันถัดจากวันสุดท้าย) แล้วคืนรายการสลิป
40  # แยกเป็นฟังก์ชัน เพื่อให้เทสต์สลับเป็นของปลอมได้ (ตอนใช้งานจริงอ่านจากตาราง slips เสมอ)
41  SlipSource = Callable[[str, date, date], list[SlipRecord]]
42  # แหล่งข้อมูลของตารางสลิปล่าสุด: รับ (shop_id, limit) คืน list ของ dict
43  RecentSource = Callable[[str, int], list[dict]]
44  
45  # ข้อความที่ตอบเมื่ออ่านฐานข้อมูลไม่ได้ บอกวิธีแก้ไปด้วยเลย (ปัญหาที่เจอบ่อยที่สุดคือยังไม่ได้สร้างตาราง)
46  DB_ERROR_DETAIL = (
47      "อ่านฐานข้อมูลไม่ได้ (สร้างตารางแล้วหรือยัง? "
48      "docker compose run --rm api python -m app.verify.db "
49      "หรือใส่ข้อมูลตัวอย่าง: docker compose run --rm api python -m app.api.seed_demo)"
50  )
```

- `L39–40` คอมเมนต์อธิบายว่าทำไมแยกแหล่งข้อมูลเป็นฟังก์ชัน (เพื่อให้เทสต์สลับเป็นของปลอมได้)
- `L41` `SlipSource = Callable[[str, date, date], list[SlipRecord]]` = type alias: "ฟังก์ชันที่รับ (str, date, date) คืน list ของ SlipRecord" · ตั้งชื่อไว้อ่านง่าย ไม่มีผลตอนรัน
- `L42–43` แบบเดียวกันสำหรับ /recent: รับ (shop_id, limit) คืน list ของ dict
- `L45` คอมเมนต์
- `L46–50` ข้อความ error ที่ **บอกวิธีแก้ไปด้วย** · ข้อความในวงเล็บหลายบรรทัดต่อกันเป็นข้อความเดียว · หน้าเว็บเอาข้อความนี้ไปแสดงในกล่อง error (page.tsx L88) · **ทำไม:** ปัญหาที่เจอบ่อยสุดตอนทดสอบคือยังไม่ได้สร้างตาราง ผู้ใช้จะรู้ทันทีว่าต้องรันคำสั่งอะไร

#### 3.4.4 การเชื่อมต่อฐานข้อมูล + เลือกร้าน

`backend/app/api/reports.py` · บรรทัด 53–65

```python
53  @lru_cache
54  def _engine() -> Engine:
55      """สร้างการเชื่อมต่อฐานข้อมูลครั้งเดียวแล้วใช้ซ้ำ
56  
57      lru_cache ทำให้เรียกครั้งที่ 2 เป็นต้นไปได้ engine ตัวเดิม (engine มี connection pool ในตัว
58      ถ้าสร้างใหม่ทุก request จะเปิด connection ใหม่เรื่อยๆ จนฐานข้อมูลเต็ม)
59      """
60      return get_engine()
61  
62  
63  def _shop_id(shop_id: str | None) -> str:
64      """ร้านที่จะดู: ส่งมาใน URL ก่อน ไม่ส่งใช้ SHOP_ID จาก .env (ค่าเดียวกับที่วี seed ร้าน)"""
65      return shop_id or os.getenv("SHOP_ID", "shop-1")
```

- `L53` `@lru_cache` — ครั้งแรกที่เรียก `_engine()` จะรันจริงและจำผลไว้ ครั้งต่อไปคืนตัวเดิมทันที
- `L54` ประกาศ คืน `Engine`
- `L55–59` docstring: engine มี **connection pool** (เก็บสายต่อฐานข้อมูลไว้ใช้ซ้ำ) ถ้าสร้างใหม่ทุก request จะเปิดสายใหม่เรื่อยๆ จน Postgres รับไม่ไหว (ค่าเริ่มต้น Postgres รับ 100 connection)
- `L60` เรียก `get_engine()` ของวี (อ่าน `DATABASE_URL` จาก .env ถ้าไม่มีโยน `RuntimeError`) · **ข้อดีของการสร้างตอนเรียกครั้งแรก (lazy) ไม่ใช่ตอน import:** ถ้ายังไม่ตั้ง DATABASE_URL แอปยังเปิดได้ เฉพาะ endpoint นี้ตอบ 503 · **ถ้าลบ `@lru_cache`:** ทุก request สร้าง engine + pool ใหม่ ช้าลงและ connection รั่ว
- `L63` ประกาศ `_shop_id`
- `L64` docstring
- `L65` `shop_id or os.getenv("SHOP_ID", "shop-1")` — ส่งมาใน URL ใช้ค่านั้น · ไม่ส่งใช้ `SHOP_ID` จาก .env (ค่าเดียวกับที่วี seed ร้าน) · ไม่มี env อีกใช้ `"shop-1"` (`getenv(ชื่อ, ค่าเริ่มต้น)`)

#### 3.4.5 แหล่งข้อมูลจริง + ตัวเลือกแหล่งข้อมูล

`backend/app/api/reports.py` · บรรทัด 68–90

```python
68  def db_source(shop_id: str, start: date, end: date) -> list[SlipRecord]:
69      """อ่านสลิปจากตาราง slips (เปิด session ใหม่ทุก request แล้วปิดเมื่อจบ with)"""
70      with Session(_engine()) as session:
71          return load_slips(session, shop_id, start, end)
72  
73  
74  def get_source() -> SlipSource:
75      """แหล่งข้อมูลของ /summary = ตาราง slips
76  
77      ใช้ผ่าน Depends() ของ FastAPI → ในเทสต์สลับเป็นของปลอมได้ด้วย app.dependency_overrides
78      """
79      return db_source
80  
81  
82  def db_recent_source(shop_id: str, limit: int) -> list[dict]:
83      """สลิปล่าสุดจากตาราง slips"""
84      with Session(_engine()) as session:
85          return load_recent(session, shop_id, limit)
86  
87  
88  def get_recent_source() -> RecentSource:
89      """แหล่งข้อมูลของ /recent = ตาราง slips (หลักเดียวกับ get_source)"""
90      return db_recent_source
```

- `L68` `db_source` มี signature ตรงกับ `SlipSource`
- `L69` docstring
- `L70` `with Session(_engine()) as session:` — `with` = context manager: เปิด session ตอนเข้า และ **ปิดให้เสมอตอนออก แม้เกิด error** (คืน connection ให้ pool) · **ถ้าไม่ใช้ with และลืม `session.close()`:** connection ค้าง pool เต็มในที่สุด
- `L71` อ่านสลิปแล้ว `return` (ออกจาก with → ปิด session)
- `L74` `get_source()` คืน **ตัวฟังก์ชัน** `db_source` (สังเกตไม่มีวงเล็บหลังชื่อ = ไม่ได้เรียก แค่ส่งฟังก์ชันออกไป ฟังก์ชันใน Python เป็นค่าธรรมดาที่ส่งต่อได้ — first-class function)
- `L75–78` docstring: ใช้ผ่าน `Depends()` → เทสต์สลับได้ด้วย `app.dependency_overrides`
- `L79` `return db_source`
- `L82–85` `db_recent_source` แบบเดียวกันสำหรับ /recent
- `L88–90` `get_recent_source` คืน `db_recent_source`
- **ทำไมต้องอ้อมผ่าน `get_source` ไม่เรียก `db_source` ตรงๆ ใน endpoint:** เพราะ FastAPI ให้แทนที่ dependency ได้ตอนเทสต์ (`app.dependency_overrides[get_source] = lambda: fake_source`) เทสต์ API จึงไม่ต้องมีฐานข้อมูล ส่วนตอนใช้งานจริงไม่มีทางได้ของปลอม เพราะโค้ดจริงไม่มีที่ไหนตั้ง override

#### 3.4.6 Endpoint `GET /api/v1/reports/summary`

`backend/app/api/reports.py` · บรรทัด 93–121

```python
 93  @router.get("/summary")
 94  def summary(
 95      mode: Mode = "daily",
 96      # ชื่อพารามิเตอร์ใน URL คือ date แต่ในโค้ดชื่อ ref เพราะ date ชนกับชื่อ type date ของ Python
 97      ref: date | None = Query(None, alias="date"),
 98      start: date | None = None,
 99      end: date | None = None,
100      shop_id: str | None = None,
101      source: SlipSource = Depends(get_source),
102  ):
103      """ยอดรวม + จำนวนสลิปแต่ละสถานะ + ข้อมูลกราฟ ของช่วงที่เลือก"""
104      ref = ref or datetime.now(BKK).date()  # ไม่ส่ง date มา = วันนี้ตามเวลาไทย
105      shop_id = _shop_id(shop_id)
106  
107      # 1) หาช่วงวัน ถ้าค่าที่ส่งมาผิด (เช่น custom ไม่มี start) ตอบ 422 พร้อมเหตุผล
108      try:
109          period_start, period_end = period_range(mode, ref, start, end)
110      except ValueError as e:
111          raise HTTPException(status_code=422, detail=str(e)) from e
112  
113      # 2) อ่านสลิป ถ้าฐานข้อมูลมีปัญหา ตอบ 503 (service ยังไม่พร้อม) พร้อมวิธีแก้
114      #    RuntimeError = ยังไม่ได้ตั้ง DATABASE_URL (มาจาก get_engine ของวี)
115      try:
116          slips = source(shop_id, period_start, period_end)
117      except (SQLAlchemyError, RuntimeError) as e:
118          raise HTTPException(status_code=503, detail=DB_ERROR_DETAIL) from e
119  
120      # 3) คำนวณยอด (ฟังก์ชันล้วนใน summary.py)
121      return build_summary(slips, mode, period_start, period_end)
```

> **รับ (จาก URL):** `mode`, `date`, `start`, `end`, `shop_id` (ไม่บังคับทุกตัว) · **คืน:** dict ของ `build_summary` → JSON · **error:** 422 (ค่าผิด), 503 (ฐานข้อมูลไม่พร้อม)

- `L93` `@router.get("/summary")` decorator ลงทะเบียน: เมื่อมี `GET /api/v1/reports/summary` ให้เรียกฟังก์ชันข้างล่าง
- `L94` ประกาศฟังก์ชัน (ไม่ใช้ `async` เพราะ SQLAlchemy ที่ใช้เป็นแบบ sync · FastAPI จะรันฟังก์ชัน sync ใน thread pool ให้ ไม่บล็อก request อื่น)
- `L95` `mode: Mode = "daily"` — FastAPI เห็นว่าพารามิเตอร์นี้ไม่ได้อยู่ใน path จึงอ่านจาก query string · type `Mode` (Literal) → ตรวจค่าให้ ค่าไม่อยู่ในรายการตอบ 422
- `L96` คอมเมนต์
- `L97` `ref: date | None = Query(None, alias="date")` — ใน URL ชื่อ `date` แต่ในโค้ดชื่อ `ref` เพราะ `date` ชนกับชื่อ type `date` ของ Python (ถ้าตั้งชื่อพารามิเตอร์ว่า `date` จะบังชื่อ type ภายในฟังก์ชัน) · FastAPI แปลง `"2026-11-18"` เป็น date ให้ รูปแบบผิดตอบ 422
- `L98–100` `start`, `end`, `shop_id` ไม่บังคับ
- `L101` `source: SlipSource = Depends(get_source)` — FastAPI เรียก `get_source()` แล้วส่งผล (ฟังก์ชัน db_source) เข้ามาเป็น `source` · ผู้ใช้ส่งค่านี้ทาง URL ไม่ได้
- `L102` ปิดวงเล็บพารามิเตอร์
- `L103` docstring (FastAPI เอาไปแสดงใน `/docs`)
- `L104` ไม่ส่ง date = วันนี้ตามเวลาไทย (เซิร์ฟเวอร์ใน docker ตั้งเป็น UTC จึงต้องระบุ BKK)
- `L105` หาร้าน
- `L107` คอมเมนต์ขั้นที่ 1
- `L108–111` `try: ... except ValueError as e: raise HTTPException(status_code=422, detail=str(e)) from e` — ถ้า `period_range` โยน ValueError (custom ไม่ครบ/ช่วงกลับด้าน/ยาวเกิน) แปลงเป็น HTTP 422 พร้อมเหตุผลภาษาไทย · `from e` = ผูก error เดิมไว้ (exception chaining) เห็นใน log ว่ามาจากไหน · **ถ้าไม่จับ:** FastAPI ตอบ 500 Internal Server Error ผู้ใช้ไม่รู้ว่าผิดตรงไหน
- `L113–114` คอมเมนต์ขั้นที่ 2
- `L115–118` อ่านสลิป · จับ 2 แบบในวงเล็บ (tuple ของ exception) · `SQLAlchemyError` = ฐานข้อมูลพัง/ไม่มีตาราง · `RuntimeError` = ยังไม่ตั้ง DATABASE_URL (จาก get_engine ของวี) → ตอบ **503 Service Unavailable** พร้อมวิธีแก้
- `L120` คอมเมนต์ขั้นที่ 3
- `L121` คำนวณแล้วคืน dict → FastAPI แปลงเป็น JSON อัตโนมัติ
- **ทำไม 422 กับ 503 ต่างกัน:** 4xx = ผู้เรียกผิด (แก้ที่ URL) · 5xx = ฝั่งเซิร์ฟเวอร์ไม่พร้อม (แก้ที่ระบบ) คน debug รู้ทันทีว่าต้องไปแก้ที่ไหน

#### 3.4.7 Endpoint `GET /api/v1/reports/recent`

`backend/app/api/reports.py` · บรรทัด 124–137

```python
124  @router.get("/recent")
125  def recent(
126      # ge/le = ตรวจค่าให้อยู่ระหว่าง 1–50 อัตโนมัติ เกินช่วง FastAPI ตอบ 422 เอง
127      limit: int = Query(10, ge=1, le=50),
128      shop_id: str | None = None,
129      source: RecentSource = Depends(get_recent_source),
130  ):
131      """สลิปล่าสุดของร้าน เรียงใหม่ → เก่า พร้อมผลตรวจและเหตุผล"""
132      try:
133          items = source(_shop_id(shop_id), limit)
134      except (SQLAlchemyError, RuntimeError) as e:
135          raise HTTPException(status_code=503, detail=DB_ERROR_DETAIL) from e
136      # ห่อด้วย {"items": ...} ไม่คืน list ตรงๆ เผื่ออนาคตเพิ่มข้อมูลอื่น (เช่นจำนวนทั้งหมด) ได้โดยไม่พังหน้าเว็บ
137      return {"items": items}
```

- `L124` ลงทะเบียน `/recent`
- `L125` ประกาศ
- `L126` คอมเมนต์
- `L127` `limit: int = Query(10, ge=1, le=50)` — ค่าเริ่มต้น 10 · `ge` = greater than or equal (≥ 1) · `le` = less than or equal (≤ 50) · ส่ง 999 มา FastAPI ตอบ 422 เอง (เทสต์ `test_recent_endpoint` ตรวจ) · **ถ้าลบ `le=50`:** ใครก็ขอทีละล้านแถวได้
- `L128` `shop_id`
- `L129` แหล่งข้อมูลผ่าน Depends
- `L130–131` ปิด + docstring
- `L132–135` เรียกแหล่งข้อมูล ถ้าฐานข้อมูลพังตอบ 503 เหมือน /summary
- `L136` คอมเมนต์
- `L137` คืน `{"items": [...]}` **ไม่คืน list ตรงๆ** เผื่ออนาคตเพิ่มช่องอื่น (เช่น `total`) ได้โดยหน้าเว็บเดิมไม่พัง (ถ้าคืน list แล้ววันหลังเปลี่ยนเป็น object หน้าเว็บที่ใช้ `.filter()` จะพัง)

<a id="f-sample"></a>
### 3.5 `backend/app/api/sample_data.py` — สลิปสุ่มสำหรับเทสต์

> **ภาพรวมไฟล์**
> - **ทำอะไร:** สร้างสลิปปลอมย้อนหลังหลายวัน แบบสุ่มที่ **กำหนด seed** (รันกี่ครั้งได้ชุดเดิม)
> - **ใครใช้:** เทสต์ (`test_sample_data_is_repeatable_and_has_today`) และ `seed_demo.py` เท่านั้น · **API ไม่ได้อ่านไฟล์นี้** ตัวเลขบนเว็บจึงมาจากตาราง slips เสมอ

#### 3.5.1 Docstring

`backend/app/api/sample_data.py` · บรรทัด 1–7

```python
1  """ข้อมูลสลิปตัวอย่าง (สุ่ม) สำหรับเทสต์และทดสอบบนเครื่องตัวเอง (แอ๊น)
2  
3  API ไม่ได้อ่านจากไฟล์นี้ (API อ่านตาราง slips ของวีเท่านั้น)
4  ใช้แค่ 2 ที่: เทสต์ และ seed_demo.py (ใส่แถว DEMO- ลงตารางตอนระบบของเพื่อนยังไม่ต่อครบ)
5  
6  สุ่มแบบกำหนด seed ทุกครั้งจึงได้ข้อมูลชุดเดิมสำหรับวันเดียวกัน (เทสต์ซ้ำได้ผลเดิม)
7  """
```

- `L1–7` บอกว่าใช้ที่ไหนบ้าง และย้ำว่า API ไม่อ่านไฟล์นี้

#### 3.5.2 Import และข้อมูลสำหรับสุ่ม

`backend/app/api/sample_data.py` · บรรทัด 9–29

```python
 9  import random
10  from datetime import date, datetime, time, timedelta
11  from decimal import Decimal
12  from functools import lru_cache
13  
14  from app.api.summary import BKK, SlipRecord
15  from app.core.contracts import Verdict
16  
17  # ชื่อย่อธนาคารแบบเดียวกับที่ทีมใช้ใน labels.csv
18  BANKS = ["KBANK", "SCB", "BBL", "KTB", "BAY", "TTB"]
19  # ราคาสินค้าที่พบบ่อยในร้านออนไลน์ (บาท)
20  AMOUNTS = [59, 89, 120, 150, 199, 250, 299, 350, 490, 590, 890, 1200, 1590]
21  
22  # เหตุผลตัวอย่างของแต่ละผล (ข้อความเดียวกับที่ชั้นตรวจจริงใช้ จะได้เห็นหน้าตาตอน demo)
23  SAMPLE_REASONS = {
24      Verdict.PASSED: [],
25      Verdict.SUSPICIOUS: ["ยังยืนยันกับธนาคารไม่ได้ ให้ร้านเช็กในแอปธนาคาร"],
26      Verdict.FAILED: ["สลิปนี้ถูกใช้ไปแล้ว"],
27  }
28  # คะแนนเสี่ยงตัวอย่าง (ตามเกณฑ์ decide(): <30 ผ่าน, 30–69 รอตรวจ, ≥70 ไม่ผ่าน)
29  SAMPLE_SCORES = {Verdict.PASSED: 0, Verdict.SUSPICIOUS: 40, Verdict.FAILED: 100}
```

- `L9` `random` ไลบรารีสุ่มมาตรฐาน
- `L10` `time` ใช้สร้างเวลาของวัน (L55)
- `L11` `Decimal` ยอดเงิน
- `L12` `lru_cache` จำผล (L42)
- `L14–15` ใช้ `BKK`, `SlipRecord` จาก summary.py และ `Verdict` ของทีม
- `L17–18` `BANKS` ชื่อย่อธนาคารแบบเดียวกับ `labels.csv` ของทีม
- `L19–20` `AMOUNTS` ราคาที่พบบ่อยในร้านออนไลน์ จะได้ดูสมจริงตอน demo
- `L22–27` `SAMPLE_REASONS` dict: key เป็น enum → list เหตุผล · ใช้ข้อความเดียวกับชั้นตรวจจริง (เช่น "สลิปนี้ถูกใช้ไปแล้ว" ของ L7 วี) จะได้เห็นหน้าตาตอน demo · ใช้โดย seed_demo.py L61
- `L28–29` `SAMPLE_SCORES` คะแนนตามเกณฑ์ `decide()` ของวี: < 30 ผ่าน, 30–69 รอตรวจ, ≥ 70 ไม่ผ่าน · ใช้โดย seed_demo.py L60

#### 3.5.3 `sample_slips()` — ฟังก์ชันที่คนอื่นเรียก

`backend/app/api/sample_data.py` · บรรทัด 32–39

```python
32  def sample_slips(today: date | None = None, days: int = 400) -> list[SlipRecord]:
33      """สลิปตัวอย่างย้อนหลัง `days` วันนับจาก `today` (ค่าเริ่มต้น = วันนี้ตามเวลาไทย)
34  
35      400 วัน = มากกว่า 1 ปีนิดหน่อย โหมดรายปีจะได้มีข้อมูลครบทุกเดือน
36      """
37      if today is None:
38          today = datetime.now(BKK).date()
39      return list(_generate(today, days))
```

> **รับ:** `today` (ไม่บังคับ), `days` (ค่าเริ่มต้น 400) · **คืน:** list ของ SlipRecord

- `L32` ประกาศ
- `L33–36` docstring: 400 วัน = เกิน 1 ปีนิดหน่อย โหมดรายปีจะมีข้อมูลครบทุกเดือน
- `L37–38` ไม่ส่ง today ใช้วันนี้ตามเวลาไทย · **ทำไมไม่ตั้งค่าเริ่มต้นในวงเล็บเป็น `today=datetime.now(BKK).date()`:** ค่าเริ่มต้นของพารามิเตอร์ Python ถูกคำนวณ **ครั้งเดียวตอนนิยามฟังก์ชัน** ถ้าเซิร์ฟเวอร์เปิดค้างข้ามวัน today จะค้างเป็นวันเดิม จึงใช้ None แล้วคำนวณข้างใน
- `L39` `list(_generate(...))` แปลง tuple จาก cache เป็น list **ใหม่** คนเรียกแก้ list นี้ได้โดยไม่กระทบ cache

#### 3.5.4 `_generate()` — สร้างข้อมูลจริง

`backend/app/api/sample_data.py` · บรรทัด 42–69

```python
42  @lru_cache(maxsize=4)
43  def _generate(today: date, days: int) -> tuple[SlipRecord, ...]:
44      """สร้างสลิปตัวอย่างจริงๆ (เก็บผลไว้ใน cache เพราะหน้าเว็บเรียกบ่อย)
45  
46      คืน tuple ไม่ใช่ list เพราะ lru_cache ต้องคืนค่าที่แก้ไม่ได้ กันคนเรียกเผลอแก้ข้อมูลใน cache
47      """
48      rng = random.Random(463)  # seed คงที่ (เลขรหัสวิชา) → สุ่มได้ชุดเดิมทุกครั้ง
49      slips: list[SlipRecord] = []
50      for back in range(days, -1, -1):  # ไล่จากวันเก่าสุดมาถึงวันนี้
51          day = today - timedelta(days=back)
52          # วันนี้ให้มีอย่างน้อย 3 ใบ หน้าจอจะได้ไม่ว่างตอนเดโม (วันอื่นมี 0–8 ใบ)
53          for _ in range(rng.randint(3, 8) if back == 0 else rng.randint(0, 8)):
54              # เวลาร้านเปิด 08:00–22:59
55              created = datetime.combine(day, time(rng.randint(8, 22), rng.randint(0, 59)), BKK)
56              # สัดส่วนผล: ผ่าน 85% / รอตรวจ 10% / ไม่ผ่าน 5%
57              verdict = rng.choices(
58                  [Verdict.PASSED, Verdict.SUSPICIOUS, Verdict.FAILED], weights=[85, 10, 5]
59              )[0]
60              slips.append(
61                  SlipRecord(
62                      trans_ref=f"SAMPLE{len(slips):06d}",  # เลขไม่ซ้ำกัน ขึ้นต้น SAMPLE ให้รู้ว่าไม่ใช่ของจริง
63                      bank=rng.choice(BANKS),
64                      amount=Decimal(rng.choice(AMOUNTS)),
65                      verdict=verdict,
66                      created_at=created,
67                  )
68              )
69      return tuple(slips)
```

- `L42` `@lru_cache(maxsize=4)` จำผลไว้ 4 ชุด (คนละค่า today/days) เกินกว่านั้นลบชุดที่ใช้นานที่สุดก่อน (LRU = Least Recently Used)
- `L43` คืน `tuple[SlipRecord, ...]` = tuple ยาวเท่าไรก็ได้
- `L44–47` docstring: คืน tuple ไม่ใช่ list เพราะ cache ใช้ผลชุดเดียวกันซ้ำ ถ้าเป็น list คนเรียกเผลอ `.append()` จะไปแก้ของใน cache · tuple แก้ไม่ได้
- `L48` `random.Random(463)` สร้างตัวสุ่มของตัวเองที่มี **seed คงที่** (463 = เลขรหัสวิชา) ได้ลำดับเลขสุ่มเดิมทุกครั้ง → เทสต์ซ้ำได้ผลเดิม · ใช้ตัวสุ่มแยก (ไม่ใช้ `random.randint` ตรงๆ) เพื่อไม่ไปเปลี่ยนสถานะตัวสุ่มกลางของโปรแกรม · **ถ้าไม่ใส่ seed:** ข้อมูลเปลี่ยนทุกครั้ง เทสต์ `a == sample_slips(DAY)` จะแดง
- `L49` list เปล่าไว้ใส่ผล (มี type hint)
- `L50` `range(days, -1, -1)` = นับถอยหลัง days, days-1, ..., 0 (อาร์กิวเมนต์ที่ 3 คือก้าว −1, ตัวที่ 2 คือหยุดก่อนถึง −1) · `back` = ย้อนหลังกี่วัน · ไล่จากเก่าไปใหม่ ข้อมูลจึงเรียงตามเวลา
- `L51` วันที่ของรอบนี้
- `L52` คอมเมนต์
- `L53` จำนวนใบของวันนั้น: วันนี้ (`back == 0`) สุ่ม 3–8 ใบ (หน้าจอไม่ว่างตอน demo) วันอื่น 0–8 ใบ · `rng.randint(a, b)` รวมทั้ง a และ b · `for _ in ...` = ไม่ได้ใช้ตัวแปรรอบ (ธรรมเนียมใช้ `_`)
- `L54` คอมเมนต์
- `L55` เวลาสุ่ม ชั่วโมง 8–22 นาที 0–59 (ร้านเปิด 08:00–22:59) รวมกับวันที่และเขตเวลาไทย
- `L56` คอมเมนต์
- `L57–59` `rng.choices([...], weights=[85, 10, 5])` สุ่มแบบถ่วงน้ำหนัก: ผ่าน 85% รอตรวจ 10% ไม่ผ่าน 5% · `choices` คืน **list** (สุ่มได้หลายตัว) จึงต้องเอา `[0]`
- `L60–68` สร้าง SlipRecord แล้ว `append` เข้า list
- `L62` `f"SAMPLE{len(slips):06d}"` เลขอ้างอิงไม่ซ้ำ (ใช้จำนวนที่สร้างไปแล้วเป็นลำดับ เติม 0 ให้ครบ 6 หลัก) ขึ้นต้น `SAMPLE` ให้รู้ว่าไม่ใช่ของจริง และไม่ชนเลขจริงของธนาคาร
- `L63–64` สุ่มธนาคารและยอด (`rng.choice` เลือก 1 ตัวจาก list)
- `L69` แปลงเป็น tuple ก่อนคืน (เพื่อเก็บใน cache อย่างปลอดภัย)

---

<a id="f-seed"></a>
### 3.6 `backend/app/api/seed_demo.py` — ใส่/ลบสลิปตัวอย่างในฐานข้อมูลจริง

> **ภาพรวมไฟล์**
> - **ทำอะไร:** สคริปต์ command line ที่ลบแถวตัวอย่างเดิม แล้วใส่สลิปสุ่มลงตาราง `slips` จริง (หรือลบอย่างเดียวด้วย `--clear`)
> - **ทำไม:** ตอนระบบยังไม่ต่อครบวง ตาราง slips ว่าง หน้าเว็บขึ้น 0 ทุกช่อง ทดสอบหน้าเว็บไม่ได้
> - **ความปลอดภัย:** ลบเฉพาะแถวที่ `line_message_id` ขึ้นต้น `DEMO-` เท่านั้น

#### 3.6.1 Docstring (คู่มือการใช้)

`backend/app/api/seed_demo.py` · บรรทัด 1–15

```python
 1  """ใส่สลิปตัวอย่างลงตาราง slips ในฐานข้อมูลจริง เพื่อดูหน้าเว็บสรุปยอด (แอ๊น)
 2  
 3  ทำไมต้องมี: ตอนยังไม่ได้ต่อ LINE ครบวง ตาราง slips ยังว่าง หน้าเว็บจะขึ้นยอด 0 ทุกช่อง
 4  สคริปต์นี้สร้างตาราง (ถ้ายังไม่มี) แล้วใส่สลิปตัวอย่าง → หน้าเว็บอ่านจากฐานข้อมูลจริงได้ทันที
 5  
 6  วิธีใช้ (รันใน docker ตอน docker compose up อยู่):
 7      docker compose run --rm api python -m app.api.seed_demo            ใส่ข้อมูลตัวอย่าง 120 วันย้อนหลัง
 8      docker compose run --rm api python -m app.api.seed_demo --days 30  ใส่ 30 วัน
 9      docker compose run --rm api python -m app.api.seed_demo --clear    ลบข้อมูลตัวอย่างออกอย่างเดียว
10  
11  ปลอดภัยกับข้อมูลจริง:
12  - ทุกแถวที่สคริปต์ใส่มี line_message_id ขึ้นต้นด้วย "DEMO-" และลบเฉพาะแถวแบบนี้เท่านั้น
13  - รันซ้ำได้ ไม่เกิดแถวซ้ำ (ลบของเก่าก่อนใส่ใหม่ทุกครั้ง)
14  - **ก่อน demo จริงวันอังคาร ให้รัน --clear** ยอดบนเว็บจะได้เป็นสลิปที่ส่งผ่าน LINE จริงเท่านั้น
15  """
```

- `L1–15` วิธีใช้ 3 แบบ (L7–9) และกติกาความปลอดภัย (L11–14) · **L14 สำคัญ: ก่อน demo จริงต้องรัน `--clear`**

#### 3.6.2 Import และค่าคงที่

`backend/app/api/seed_demo.py` · บรรทัด 17–29

```python
17  import argparse
18  import os
19  from datetime import timezone
20  
21  from sqlalchemy import delete
22  from sqlalchemy.orm import Session
23  
24  from app.api.sample_data import SAMPLE_REASONS, SAMPLE_SCORES, sample_slips
25  from app.verify.db import create_tables, get_engine
26  from app.verify.models import Slip
27  
28  # ป้ายที่ใช้แยกข้อมูลตัวอย่างออกจากสลิปจริง
29  DEMO_PREFIX = "DEMO-"
```

- `L17` `argparse` อ่านอาร์กิวเมนต์ command line
- `L18` `os` อ่าน env `SHOP_ID`
- `L19` `timezone` แปลงเป็น UTC (L55)
- `L21` `delete` สร้างคำสั่ง DELETE
- `L22` `Session`
- `L24` ข้อมูลจาก sample_data.py
- `L25` `create_tables`, `get_engine` ของวี
- `L26` ตาราง `Slip` ของวี
- `L28–29` `DEMO_PREFIX = "DEMO-"` ป้ายแยกข้อมูลตัวอย่าง · ใช้ใน L40, L52 และเทสต์ · เก็บเป็นค่าคงที่ที่เดียว ถ้าเปลี่ยนจะเปลี่ยนทั้งการใส่และการลบพร้อมกัน

#### 3.6.3 `seed()` — งานหลัก

`backend/app/api/seed_demo.py` · บรรทัด 32–66

```python
32  def seed(session: Session, shop_id: str, days: int, clear_only: bool = False) -> tuple[int, int]:
33      """ลบข้อมูลตัวอย่างเก่า แล้วใส่ใหม่ (ถ้าไม่ใช่ clear_only)
34  
35      คืน (จำนวนแถวที่ลบ, จำนวนแถวที่ใส่) · ไม่ commit เอง ให้คนเรียกเป็นคน commit
36      (แบบเดียวกับ save_slip() ของวี) เทสต์จะได้ rollback ได้
37      """
38      # 1) ลบเฉพาะแถวที่ line_message_id ขึ้นต้นด้วย DEMO- (สลิปจริงจาก LINE ไม่โดนลบ)
39      removed = session.execute(
40          delete(Slip).where(Slip.line_message_id.like(f"{DEMO_PREFIX}%"))
41      ).rowcount or 0
42      if clear_only:
43          return removed, 0
44  
45      # 2) แปลงสลิปตัวอย่าง (SlipRecord) เป็นแถวในตาราง slips ทีละใบ
46      records = sample_slips(days=days)
47      for rec in records:
48          session.add(
49              Slip(
50                  shop_id=shop_id,
51                  line_user_id="DEMO-CUSTOMER",  # ไม่ใช่ userId ของ LINE จริง
52                  line_message_id=f"{DEMO_PREFIX}{rec.trans_ref}",
53                  # ใส่เวลาเองแทน server_default เพื่อกระจายย้อนหลังหลายวัน
54                  # แปลงเป็น UTC ก่อน: Postgres แปลงให้เองได้ แต่ SQLite (ในเทสต์) จะเก็บตัวเลขเวลาไทยไปตรงๆ
55                  received_at=rec.created_at.astimezone(timezone.utc),
56                  trans_ref=rec.trans_ref,  # ขึ้นต้น SAMPLE ไม่ชนกับเลขอ้างอิงจริงของธนาคาร
57                  bank=rec.bank,
58                  amount=rec.amount,
59                  verdict=rec.verdict.value,
60                  fraud_score=SAMPLE_SCORES[rec.verdict],
61                  reasons=list(SAMPLE_REASONS[rec.verdict]),
62                  layers=[],  # ข้อมูลตัวอย่างไม่มีผลรายชั้น
63              )
64          )
65      session.flush()  # ส่ง INSERT ไปฐานข้อมูล (ยังไม่ commit) ถ้ามี error จะเห็นตรงนี้
66      return removed, len(records)
```

> **รับ:** session, shop_id, days, clear_only · **คืน:** `(จำนวนแถวที่ลบ, จำนวนแถวที่ใส่)` · **ไม่ commit เอง**

- `L32` ประกาศ คืน `tuple[int, int]`
- `L33–37` docstring: ไม่ commit เอง ให้คนเรียก commit (แบบเดียวกับ `save_slip()` ของวี) · ข้อดี: เทสต์ใช้ฐานข้อมูลชั่วคราวได้ และถ้าพังกลางทางจะไม่มีข้อมูลครึ่งๆ กลางๆ
- `L38` คอมเมนต์
- `L39–41` `delete(Slip).where(Slip.line_message_id.like(f"{DEMO_PREFIX}%"))` = `DELETE FROM slips WHERE line_message_id LIKE 'DEMO-%'` · `%` ใน LIKE = อักขระอะไรก็ได้กี่ตัวก็ได้ · `.rowcount` = จำนวนแถวที่ถูกลบ · `or 0` กันกรณีได้ None · **ปลอดภัยเพราะ** message id ของ LINE เป็นตัวเลขล้วน ไม่มีทางขึ้นต้น `DEMO-` (เทสต์ `test_seed_demo_is_repeatable_and_keeps_real_slips`) · **ถ้าลบเงื่อนไข where:** ลบสลิปจริงทั้งตาราง!
- `L42–43` `--clear` = ลบแล้วจบ คืน `(removed, 0)`
- `L45` คอมเมนต์
- `L46` สร้างสลิปสุ่ม
- `L47` วนทีละใบ
- `L48–64` `session.add(Slip(...))` เพิ่ม object ใหม่เข้า session (ยังไม่เขียนลงฐานข้อมูลจริง)
- `L50` ร้านที่ระบุ
- `L51` `line_user_id="DEMO-CUSTOMER"` ไม่ใช่ userId ของ LINE จริง (คอลัมน์นี้ห้าม NULL จึงต้องใส่ค่า)
- `L52` `line_message_id=f"DEMO-{rec.trans_ref}"` เช่น `DEMO-SAMPLE000123` · คอลัมน์นี้ **unique** ในตารางของวี เลขจาก sample_data ไม่ซ้ำกันจึงใส่ได้ · ป้าย DEMO- ทำให้ L40 หาเจอตอนลบ
- `L53–54` คอมเมนต์
- `L55` ใส่ `received_at` เอง (ปกติฐานข้อมูลใส่เวลาปัจจุบันให้ผ่าน `server_default`) เพื่อกระจายย้อนหลังหลายวัน · แปลงเป็น UTC ก่อน เพราะ SQLite เก็บตัวเลขเวลาไปตรงๆ ไม่แปลงให้ (Postgres แปลงให้)
- `L56` `trans_ref` ขึ้นต้น SAMPLE · ข้อควรรู้: ตารางของวีมี unique index `uq_slips_active_trans_ref` (เลขอ้างอิงที่ passed/suspicious ซ้ำไม่ได้) เลข SAMPLE ไม่ซ้ำกันจึงไม่ชน
- `L57–58` ธนาคาร ยอด
- `L59` `rec.verdict.value` enum → ข้อความ `"passed"` (คอลัมน์ verdict เป็น String)
- `L60–61` คะแนน/เหตุผลตัวอย่างตามผล · `list(...)` สร้าง list ใหม่ ไม่ใช้ list ตัวเดียวกับใน `SAMPLE_REASONS` (ถ้าใช้ร่วมกันแล้วมีใครแก้ จะแก้ทุกแถว)
- `L62` `layers=[]` ข้อมูลตัวอย่างไม่มีผลรายชั้น
- `L65` `session.flush()` = ส่ง INSERT ไปฐานข้อมูลตอนนี้ แต่ **ยังไม่ commit** ถ้ามี error (เช่นค่าซ้ำ) จะเห็นตรงนี้ · และ query ถัดไปใน session เดียวกันจะเห็นแถวเหล่านี้
- `L66` คืนจำนวน

#### 3.6.4 `main()` — รับคำสั่งจาก command line

`backend/app/api/seed_demo.py` · บรรทัด 69–87

```python
69  def main(argv: list[str] | None = None) -> None:
70      """อ่านคำสั่งจาก command line แล้วรัน seed()"""
71      parser = argparse.ArgumentParser(description="ใส่/ลบสลิปตัวอย่างในตาราง slips")
72      parser.add_argument("--shop", default=os.getenv("SHOP_ID", "shop-1"), help="shop_id (ค่าเริ่มต้น SHOP_ID ใน .env)")
73      parser.add_argument("--days", type=int, default=120, help="ย้อนหลังกี่วัน (ค่าเริ่มต้น 120)")
74      parser.add_argument("--clear", action="store_true", help="ลบข้อมูลตัวอย่างอย่างเดียว ไม่ใส่ใหม่")
75      args = parser.parse_args(argv)
76  
77      engine = get_engine()
78      create_tables(engine)  # สร้างตาราง slips/shops ถ้ายังไม่มี (มีแล้วไม่ทำอะไร)
79      with Session(engine) as session:
80          removed, added = seed(session, args.shop, args.days, clear_only=args.clear)
81          session.commit()  # บันทึกจริงทีเดียวตอนจบ ถ้าพังกลางทางจะไม่มีข้อมูลครึ่งๆ กลางๆ
82      print(f"ลบข้อมูลตัวอย่างเดิม {removed} แถว · ใส่ใหม่ {added} แถว (ร้าน {args.shop})")
83  
84  
85  # รันด้วย python -m app.api.seed_demo จะเข้าเงื่อนไขนี้ (import จากไฟล์อื่นจะไม่รัน)
86  if __name__ == "__main__":
87      main()
```

- `L69–70` ประกาศ · `argv` ไม่บังคับ (ไม่ส่ง = argparse อ่านจาก `sys.argv` เอง) ทำให้เทสต์เรียก `main([...])` ได้
- `L71` สร้างตัวอ่านอาร์กิวเมนต์ พร้อมคำอธิบาย (เห็นตอนรัน `--help`)
- `L72` `--shop` ค่าเริ่มต้น `SHOP_ID` จาก env หรือ `"shop-1"`
- `L73` `--days` แปลงเป็น int (`type=int`) ค่าเริ่มต้น 120
- `L74` `--clear` แบบ flag: `action="store_true"` ใส่แล้วได้ True ไม่ต้องตามด้วยค่า
- `L75` `parse_args` อ่านและตรวจ (พิมพ์ผิดจะแสดงวิธีใช้แล้วออก)
- `L77` สร้าง engine (ไม่ต้อง cache เพราะสคริปต์รันครั้งเดียวแล้วจบ)
- `L78` `create_tables` สร้างตารางถ้ายังไม่มี (มีแล้วไม่ทำอะไร) → รันบนฐานข้อมูลเปล่าได้เลย
- `L79` เปิด session ด้วย with
- `L80` เรียก `seed` (args.clear → clear_only)
- `L81` `session.commit()` บันทึกจริงทีเดียวตอนจบ = ทั้งหมดหรือไม่มีเลย (atomic) ถ้าพังก่อนบรรทัดนี้ session ปิดโดยไม่ commit ข้อมูลไม่เปลี่ยน
- `L82` พิมพ์สรุป
- `L85` คอมเมนต์
- `L86` `if __name__ == "__main__":` — ตัวแปรพิเศษ `__name__` เป็น `"__main__"` เฉพาะตอนไฟล์นี้ถูกรันโดยตรง (`python -m app.api.seed_demo`) · ถ้าไฟล์อื่น import (เช่นเทสต์) จะเป็นชื่อโมดูล main() จึงไม่รัน · **ถ้าลบ if (เรียก main() ตรงๆ):** เทสต์ที่ import `seed` จะไปรันสคริปต์และพยายามต่อฐานข้อมูลจริง
- `L87` เรียก main()

<a id="f-timing"></a>
### 3.7 `backend/app/metrics/__init__.py` + `timing.py` — จับเวลาแต่ละชั้น

`backend/app/metrics/__init__.py` · บรรทัด 1–1

```python
1  """วัดเวลา/ต้นทุนของระบบ (แอ๊น) สำหรับ A6"""
```

- `L1` docstring ของ package `metrics` · ไฟล์นี้ทำให้ `from app.metrics.timing import timed` ได้

> **ภาพรวม `timing.py`**
> - **ทำอะไร:** ให้ทุกคนครอบโค้ดชั้นของตัวเองด้วย `with timed("ชื่อชั้น"):` แล้วได้ log 1 บรรทัด `TIMING layer=... ms=...`
> - **ทำไม:** A6 ต้องมีเวลาต่อสลิปแยกทุกชั้น (p50/p95) · ใช้ log แทนการเขียนฐานข้อมูล เพราะไม่ต้องแก้ตาราง ไม่ทำให้ระบบช้า และ `docker compose logs` เก็บไว้ให้แล้ว
> - **รับ:** ชื่อชั้น (str), dict สำหรับเก็บเวลา (ไม่บังคับ) · **ส่งออก:** บรรทัด log (stderr) + ค่าใน sink · **ต่อกับ:** `report.py` อ่าน log นี้

#### 3.7.1 Docstring (วิธีใช้ + ชื่อชั้นที่ตกลงกัน)

`backend/app/metrics/timing.py` · บรรทัด 1–20

```python
 1  """จับเวลาแต่ละชั้นของการตรวจสลิป เพื่อทำตารางต้นทุน A6 (แอ๊น)
 2  
 3  ทุกคนเรียกใช้แบบนี้ครอบโค้ดชั้นของตัวเอง (ไม่ต้องแก้อะไรอื่น):
 4  
 5      from app.metrics.timing import timed
 6  
 7      with timed("L2_qr"):
 8          layer, data = check_qr(image)
 9  
10  ทุกครั้งที่ออกจาก with จะพิมพ์ 1 บรรทัดลง log ของ api:
11  
12      TIMING layer=L2_qr ms=12.4
13  
14  แล้วแอ๊นเก็บ log มาสรุป p50/p95 ด้วย app/metrics/report.py
15  
16  ชื่อ layer ที่ตกลงกัน (ใช้ชื่อเดียวกับ LayerResult.layer จะได้เทียบกันง่าย):
17      L1_precheck, L2_qr, L3_ocr, L5_tamper  (บีม/ชิชา ใน run_pipeline)
18      L6_bank, L7_checks                      (วี ใน verify_slip)
19      download_image, total                   (แพรว ใน webhook: เวลาดึงรูปจาก LINE / ทั้งหมดตั้งแต่รับ webhook จนตอบ)
20  """
```

- `L1–20` วิธีใช้ (L5–8) · รูปแบบ log (L12) · ชื่อชั้นที่ตกลงกัน (L16–19) ใช้ชื่อเดียวกับ `LayerResult.layer` จะได้เทียบกันง่าย · **ข้อควรรู้:** ชื่อชั้น L6 ในนี้เขียน `L6_bank` แต่คอมเมนต์ใน `contracts.py` เขียน `L6_transaction` (ดูข้อ 8)

#### 3.7.2 Import และตั้งค่า logger

`backend/app/metrics/timing.py` · บรรทัด 22–37

```python
22  import logging
23  import time
24  from collections.abc import Iterator
25  from contextlib import contextmanager
26  
27  # logger แยกของเราเอง ชื่อ slipguard.timing
28  logger = logging.getLogger("slipguard.timing")
29  
30  # uvicorn ไม่ได้ตั้ง handler ให้ logger ที่เราสร้างเอง → ข้อความระดับ INFO จะหายไป
31  # จึงใส่ handler ที่พิมพ์ออกหน้าจอ (stderr → docker compose logs เห็น) ให้เองครั้งเดียว
32  if not logger.handlers:
33      _handler = logging.StreamHandler()
34      _handler.setFormatter(logging.Formatter("%(message)s"))  # พิมพ์แค่ข้อความ ให้ report.py อ่านง่าย
35      logger.addHandler(_handler)
36      logger.setLevel(logging.INFO)
37      logger.propagate = False  # ไม่ส่งต่อให้ root logger กันพิมพ์ซ้ำ 2 รอบ
```

- `L22` `logging` ไลบรารี log มาตรฐาน
- `L23` `time` ใช้ `perf_counter`
- `L24` `Iterator` type ของ generator (ใช้กับ L41)
- `L25` `contextmanager` decorator เปลี่ยน generator เป็น context manager (ใช้กับ `with`)
- `L27–28` `logging.getLogger("slipguard.timing")` ได้ logger ชื่อนี้ (เรียกชื่อเดิมที่ไหนก็ได้ตัวเดิม) · ชื่อมีจุด = ลำดับชั้น (ลูกของ `slipguard`)
- `L30–31` คอมเมนต์: uvicorn ไม่ตั้ง handler ให้ logger ที่เราสร้างเอง ข้อความระดับ INFO จะหาย
- `L32` `if not logger.handlers:` ตั้งค่าครั้งเดียว · ถ้าไม่มี if แล้วไฟล์ถูกโหลดซ้ำ (เช่น reload) จะมี handler ซ้อนกัน พิมพ์ซ้ำหลายบรรทัด
- `L33` `StreamHandler()` = handler ที่พิมพ์ออก stderr → docker compose logs เห็น
- `L34` `Formatter("%(message)s")` พิมพ์แค่ข้อความ ไม่มีวันที่/ระดับนำหน้า report.py อ่านง่าย (`_handler` ขึ้นต้น `_` = ตัวแปรภายในโมดูล)
- `L35` ผูก handler เข้ากับ logger
- `L36` `setLevel(logging.INFO)` ระดับค่าเริ่มต้นของ Python คือ WARNING ซึ่งจะ **ไม่พิมพ์** ข้อความ INFO · **ถ้าลบ:** `logger.info(...)` ใน L57 จะไม่ออกเลย report.py จะไม่เจอบรรทัด TIMING
- `L37` `propagate = False` ไม่ส่งต่อให้ logger ตัวแม่ (root) กันพิมพ์ซ้ำ 2 บรรทัด (ถ้า root มี handler อยู่แล้ว)

#### 3.7.3 `timed()` — context manager จับเวลา

`backend/app/metrics/timing.py` · บรรทัด 40–57

```python
40  @contextmanager
41  def timed(layer: str, sink: dict[str, float] | None = None) -> Iterator[None]:
42      """จับเวลาโค้ดใน with แล้ว log เป็นมิลลิวินาที
43  
44      layer: ชื่อชั้น เช่น "L2_qr"
45      sink:  (ไม่บังคับ) dict ที่จะเก็บเวลาไว้ด้วย เช่นอยากใส่เวลาลง LayerResult หรือใช้ในเทสต์
46  
47      ใช้ try/finally → ถึงโค้ดข้างในจะ error ก็ยัง log เวลา (ชั้นที่พังช้าแค่ไหนก็เป็นข้อมูลที่ต้องรู้)
48      """
49      # perf_counter = นาฬิกาสำหรับวัดช่วงเวลา แม่นกว่า time.time() และไม่เพี้ยนเมื่อเครื่องปรับนาฬิกา
50      start = time.perf_counter()
51      try:
52          yield  # ← โค้ดใน with ของคนเรียกทำงานตรงนี้
53      finally:
54          ms = (time.perf_counter() - start) * 1000  # วินาที → มิลลิวินาที
55          if sink is not None:
56              sink[layer] = ms
57          logger.info("TIMING layer=%s ms=%.1f", layer, ms)
```

- `L40` `@contextmanager` — ทำให้ฟังก์ชันที่มี `yield` ใช้กับ `with` ได้: โค้ด **ก่อน** `yield` รันตอนเข้า with · โค้ดหลัง `yield` (ใน finally) รันตอนออก
- `L41` รับ `layer` และ `sink: dict[str, float] | None = None` (dict ที่ key เป็นชื่อชั้น ค่าเป็น ms ไม่บังคับ) · คืน `Iterator[None]` (yield ไม่ส่งค่าออกไป `with timed(...)` จึงไม่ต้อง `as`)
- `L42–48` docstring
- `L49` คอมเมนต์
- `L50` `time.perf_counter()` นาฬิกาสำหรับวัดช่วงเวลา ความละเอียดสูง (ระดับนาโน/ไมโครวินาที) และ **ไม่เพี้ยนถ้าเครื่องปรับนาฬิกา** · **ถ้าใช้ `time.time()`:** ถ้านาฬิกาเครื่องถูกซิงก์ระหว่างวัด อาจได้เวลาติดลบ
- `L51` `try:`
- `L52` `yield` — จุดที่ **โค้ดใน with ของคนเรียกทำงาน** ฟังก์ชันหยุดรอตรงนี้ โค้ดข้างในเสร็จ (หรือ error) แล้วค่อยทำต่อ
- `L53` `finally:` ทำ **เสมอ** ไม่ว่าโค้ดข้างในจะสำเร็จหรือ error · error ยังถูกโยนต่อออกไปหลัง finally (เทสต์ `test_timed_still_records_when_code_raises`) · **ทำไม:** ชั้นที่พังใช้เวลาเท่าไรก็เป็นข้อมูลต้นทุน · **ถ้าไม่ใช้ try/finally:** ชั้นที่ error จะไม่มีเวลาใน log ตาราง A6 จะดูเร็วกว่าความจริง
- `L54` `(ตอนนี้ - ตอนเริ่ม) * 1000` วินาที → มิลลิวินาที
- `L55–56` ถ้าส่ง sink มา เก็บเวลาลง dict ด้วย
- `L57` `logger.info("TIMING layer=%s ms=%.1f", layer, ms)` → `TIMING layer=L2_qr ms=12.4` · `%s` = ข้อความ `%.1f` = ทศนิยม 1 ตำแหน่ง · ส่งค่าเป็นอาร์กิวเมนต์แยก (ไม่ใช้ f-string) เป็นธรรมเนียมของ logging: ถ้าระดับนั้นถูกปิด จะไม่เสียเวลาจัดข้อความ · **รูปแบบนี้ต้องตรงกับ regex ใน report.py L29** ถ้าเปลี่ยนข้อความต้องแก้ทั้งสองที่

---

<a id="f-report"></a>
### 3.8 `backend/app/metrics/report.py` — สรุป log เป็นตาราง p50/p95

> **ภาพรวมไฟล์**
> - **ทำอะไร:** อ่าน log (จากไฟล์หรือ pipe) → เก็บเวลาแยกตามชั้น → พิมพ์ตาราง markdown (ชั้น, n, p50, p95, max)
> - **รับ:** บรรทัด log · **ส่งออก:** ข้อความตารางทาง stdout (copy ไปใส่ slide ได้เลย)
> - **ต่อกับ:** `timing.py` (บรรทัด `TIMING`) และ webhook ของแพรว (บรรทัด `slip_timing ... total_ms=`)

#### 3.8.1 Docstring (วิธีใช้ + ความหมาย p50/p95)

`backend/app/metrics/report.py` · บรรทัด 1–20

```python
 1  """สรุปเวลาแต่ละชั้นจาก log เป็นตาราง p50/p95 สำหรับ slide A6 (แอ๊น)
 2  
 3  ขั้นตอน (หลังส่งสลิปทดสอบผ่านระบบหลายๆ ใบแล้ว):
 4  
 5      # PowerShell / bash เหมือนกัน: ดึง log ของ api แล้วส่งเข้าสคริปต์ผ่าน pipe (|)
 6      docker compose logs api --no-log-prefix | docker compose run --rm -T api python -m app.metrics.report
 7  
 8      # หรือบันทึก log เป็นไฟล์ไว้ในโฟลเดอร์ backend ก่อน (ห้าม commit ไฟล์ log)
 9      docker compose logs api --no-log-prefix > backend/timing.log
10      docker compose run --rm api python -m app.metrics.report timing.log
11  
12  ผลลัพธ์เป็นตาราง markdown copy ไปใส่ slide / docs ได้เลย:
13  
14      | ชั้น | n | p50 (ms) | p95 (ms) | max (ms) |
15      |---|---|---|---|---|
16      | L2_qr | 40 | 12.3 | 30.1 | 41.0 |
17  
18  p50 = ค่ากลาง (ครึ่งหนึ่งเร็วกว่านี้) · p95 = 95% ของสลิปเร็วกว่านี้ (ดูกรณีช้าที่เจอบ่อยพอสมควร)
19  ใช้ p95 แทนค่าเฉลี่ยเพราะค่าเฉลี่ยโดนสลิปที่ช้าผิดปกติไม่กี่ใบดึงจนไม่สะท้อนของจริง
20  """
```

- `L1–20` คำสั่งใช้งาน 2 แบบ (pipe L6 / ผ่านไฟล์ L9–10) · ตัวอย่างผลลัพธ์ (L14–16) · ความหมาย p50/p95 และเหตุผลที่ไม่ใช้ค่าเฉลี่ย (L18–19) · L8 ย้ำว่าห้าม commit ไฟล์ log

#### 3.8.2 Import และ regex

`backend/app/metrics/report.py` · บรรทัด 22–32

```python
22  import math
23  import re
24  import sys
25  from collections import defaultdict
26  from collections.abc import Iterable
27  
28  # หาบรรทัดแบบ "TIMING layer=L2_qr ms=12.4" (มีข้อความอื่นนำหน้าได้ เช่น prefix ของ docker)
29  LINE_RE = re.compile(r"TIMING layer=(\S+) ms=([0-9.]+)")
30  # บรรทัดเวลารวมจาก webhook ของแพรว: "slip_timing message_id=... verdict=passed total_ms=1234"
31  # = ตั้งแต่เริ่มทำงานกับรูปจนตอบลูกค้า → นับเป็นชั้นชื่อ "total" (ตัวเลขที่เทียบกับเป้า A1 ≤ 5 วินาที p95)
32  TOTAL_RE = re.compile(r"slip_timing .*total_ms=([0-9.]+)")
```

- `L22` `math` ใช้ `ceil` (ปัดขึ้น)
- `L23` `re` = regular expression (รูปแบบค้นหาข้อความ)
- `L24` `sys` ใช้ `argv`, `stdin`
- `L25` `defaultdict`
- `L26` `Iterable`
- `L28` คอมเมนต์
- `L29` `re.compile(r"TIMING layer=(\S+) ms=([0-9.]+)")` — `r"..."` = raw string (ไม่ตีความ `\` เป็น escape ของ Python) · `compile` แปลง regex ครั้งเดียวใช้ซ้ำได้เร็ว
  - `\S+` = อักขระที่ไม่ใช่ช่องว่าง 1 ตัวขึ้นไป (ชื่อชั้น) · วงเล็บ `( )` = **group** เก็บค่าที่จับได้ → group 1
  - `[0-9.]+` = ตัวเลขหรือจุด 1 ตัวขึ้นไป (เวลา) → group 2
- `L30–31` คอมเมนต์: บรรทัดเวลารวมของแพรว = ตั้งแต่เริ่มทำงานกับรูปจนตอบลูกค้า ตัวเลขที่เทียบกับเป้า A1 (≤ 5 วินาที p95)
- `L32` `slip_timing .*total_ms=([0-9.]+)` · `.*` = อักขระอะไรก็ได้กี่ตัวก็ได้ (ข้าม message_id และ verdict ไป)

#### 3.8.3 `parse()` — อ่าน log

`backend/app/metrics/report.py` · บรรทัด 35–46

```python
35  def parse(lines: Iterable[str]) -> dict[str, list[float]]:
36      """อ่าน log ทีละบรรทัด → {ชื่อชั้น: [เวลา ms, ...]} (บรรทัด TIMING + slip_timing ของแพรว, บรรทัดอื่นข้าม)"""
37      result: dict[str, list[float]] = defaultdict(list)  # ชั้นที่ยังไม่มีจะได้ list ว่างอัตโนมัติ
38      for line in lines:
39          m = LINE_RE.search(line)
40          if m:
41              result[m.group(1)].append(float(m.group(2)))
42              continue
43          t = TOTAL_RE.search(line)  # ไม่ใช่ TIMING ลองดูว่าเป็นเวลารวมของ webhook ไหม
44          if t:
45              result["total"].append(float(t.group(1)))
46      return dict(result)
```

> **รับ:** iterable ของบรรทัด (ไฟล์ที่เปิดแล้ว, `sys.stdin`, หรือ list ในเทสต์ — วน for ได้เหมือนกันหมด) · **คืน:** `{ชื่อชั้น: [ms, ms, ...]}`

- `L35–36` ประกาศ + docstring
- `L37` `defaultdict(list)` = dict ที่ถ้าเรียก key ที่ยังไม่มี จะสร้าง `list()` ว่างให้เอง → เขียน `result[ชื่อ].append(x)` ได้ทันที ไม่ต้องเช็ก `if ชื่อ not in result` · **ถ้าใช้ dict ธรรมดา:** `KeyError` ตอนเจอชั้นแรก
- `L38` วนทีละบรรทัด
- `L39` `LINE_RE.search(line)` หา **ที่ไหนก็ได้** ในบรรทัด (ต่างจาก `match` ที่ต้องเริ่มต้นบรรทัด) → อ่านได้แม้มี prefix `api-1  | ` ของ docker
- `L40–42` เจอ → `m.group(1)` ชื่อชั้น, `float(m.group(2))` เวลา · `continue` ไปบรรทัดถัดไป
- `L43` ไม่ใช่ TIMING ลองดูว่าเป็นบรรทัดเวลารวมไหม
- `L44–45` เจอ → เก็บใต้ชื่อ `"total"`
- บรรทัดอื่น (เช่น `GET /healthz 200 OK`) ไม่ตรงทั้งสองแบบ = ข้าม
- `L46` แปลงเป็น dict ธรรมดาก่อนคืน (คนเรียกจะไม่เผลอสร้าง key ใหม่ด้วยการอ่าน)

#### 3.8.4 `percentile()` — แบบ nearest-rank

`backend/app/metrics/report.py` · บรรทัด 49–58

```python
49  def percentile(values: list[float], p: float) -> float:
50      """เปอร์เซ็นไทล์แบบ nearest-rank (วิธีที่อธิบายง่ายที่สุด)
51  
52      เรียงค่าจากน้อยไปมาก แล้วหยิบตัวที่ ceil(p/100 × n) เช่น n=20, p95 → ตัวที่ 19
53      """
54      if not values:
55          raise ValueError("ไม่มีข้อมูล")
56      ordered = sorted(values)
57      rank = max(1, math.ceil(p / 100 * len(ordered)))  # อย่างน้อยตัวที่ 1
58      return ordered[rank - 1]  # list เริ่มนับที่ 0
```

- `L49` ประกาศ รับ list และ p (เช่น 95)
- `L50–53` docstring มีตัวอย่าง
- `L54–55` list ว่าง → `ValueError` (หาร/เลือกตัวไม่ได้)
- `L56` `sorted(values)` คืน list ใหม่ที่เรียงแล้ว (ไม่แก้ list เดิม)
- `L57` `math.ceil(p / 100 * len(ordered))` เช่น n = 20, p95 → ceil(19.0) = 19 · p50 → 10 · `max(1, ...)` กันได้ 0 (เช่น p = 0)
- `L58` `ordered[rank - 1]` เพราะ list นับตำแหน่งจาก 0
- **ทำไม nearest-rank:** อธิบายง่ายที่สุด และค่าที่ได้เป็นค่าที่วัดได้จริง (ไม่ใช่ค่าเฉลี่ยระหว่าง 2 ตัวแบบ interpolation)
- **ข้อจำกัด:** ถ้า n < 20 → ceil(0.95 × n) = n → p95 = ค่าช้าที่สุด จึงต้องรายงาน n ทุกแถว

#### 3.8.5 `markdown_table()` — สร้างตาราง

`backend/app/metrics/report.py` · บรรทัด 61–70

```python
61  def markdown_table(timings: dict[str, list[float]]) -> str:
62      """สร้างตาราง markdown เรียงตามชื่อชั้น (L1 → L7 แล้วค่อยตัวอื่น)"""
63      rows = ["| ชั้น | n | p50 (ms) | p95 (ms) | max (ms) |", "|---|---|---|---|---|"]
64      for layer in sorted(timings):
65          values = timings[layer]
66          rows.append(
67              f"| {layer} | {len(values)} | {percentile(values, 50):.1f} | "
68              f"{percentile(values, 95):.1f} | {max(values):.1f} |"
69          )
70      return "\n".join(rows)
```

- `L61–62` ประกาศ + docstring
- `L63` list เริ่มด้วยหัวตาราง markdown 2 บรรทัด (บรรทัดที่ 2 `|---|` บอกว่าบรรทัดแรกเป็นหัว)
- `L64` `sorted(timings)` เรียง key ตามตัวอักษร → `L1_precheck`, `L2_qr`, ..., `download_image`, `total` (ตัวพิมพ์ใหญ่ `L` มาก่อนตัวพิมพ์เล็ก)
- `L65` ค่าของชั้นนั้น
- `L66–69` เพิ่ม 1 แถว: ชื่อ, n (`len`), p50, p95, max · `:.1f` ทศนิยม 1 ตำแหน่ง · f-string 2 ก้อนในวงเล็บต่อกันเป็นข้อความเดียว
- `L70` `"\n".join(rows)` รวมทุกแถวด้วยการขึ้นบรรทัดใหม่

#### 3.8.6 `main()` — อ่านจากไฟล์หรือ pipe

`backend/app/metrics/report.py` · บรรทัด 73–89

```python
73  def main(argv: list[str] | None = None) -> None:
74      """อ่านจากไฟล์ (ถ้าส่งชื่อไฟล์มา) หรือจาก stdin (ถ้าส่งผ่าน pipe)"""
75      argv = sys.argv[1:] if argv is None else argv
76      if argv:
77          with open(argv[0], encoding="utf-8", errors="replace") as f:
78              timings = parse(f)
79      else:
80          timings = parse(sys.stdin)
81  
82      if not timings:
83          print("ไม่เจอบรรทัด TIMING ใน log (ส่งสลิปผ่านระบบแล้วหรือยัง? ทุกชั้นครอบ timed() แล้วหรือยัง?)")
84          return
85      print(markdown_table(timings))
86  
87  
88  if __name__ == "__main__":
89      main()
```

- `L73–74` ประกาศ + docstring
- `L75` `sys.argv[1:] if argv is None else argv` — `sys.argv` = อาร์กิวเมนต์จาก command line (ตัวที่ 0 คือชื่อสคริปต์ จึงตัดออก) · รับ argv เป็นพารามิเตอร์ได้ ทำให้เทสต์เรียก `main(["ไฟล์"])` ได้
- `L76–78` มีชื่อไฟล์ → เปิดไฟล์ (`encoding="utf-8"` รองรับภาษาไทย, `errors="replace"` อักขระเสียแทนด้วย � แทนที่จะพัง) · `with open(...)` ปิดไฟล์ให้เอง · ไฟล์ที่เปิดแล้ววน for ได้ทีละบรรทัด ส่งเข้า `parse` ตรงๆ
- `L79–80` ไม่มีชื่อไฟล์ → อ่าน `sys.stdin` (ข้อมูลที่ส่งมาทาง pipe `|`)
- `L82–84` ไม่เจอบรรทัดเลย → บอกสาเหตุที่น่าจะเป็น แล้ว `return`
- `L85` พิมพ์ตาราง
- `L88–89` รันเมื่อเรียกไฟล์นี้โดยตรง (`python -m app.metrics.report`)

**คำสั่งจริง:** `docker compose logs api --no-log-prefix | docker compose run --rm -T api python -m app.metrics.report` · `--no-log-prefix` ตัด `api-1 |` ออก · `-T` = ไม่จอง terminal (TTY) ให้ข้อมูลจาก pipe ไหลเข้า stdin ได้

---

<a id="f-touch"></a>
### 3.9 จุดที่โค้ดของแอ๊นต่อกับไฟล์ของคนอื่น

ไฟล์ในหัวข้อนี้ **ไม่ใช่ไฟล์ของแอ๊น** แสดงเฉพาะบรรทัดที่เกี่ยวข้อง เพื่อให้เห็นว่าโค้ดของแอ๊นถูกเรียกจากที่ไหน

**`backend/app/main.py`** (ไฟล์กลางของทีม)

`backend/app/main.py` · บรรทัด 1–14

```python
 1  from fastapi import FastAPI
 2  
 3  from app.api.reports import router as reports_router
 4  from app.line.webhook import router as line_router
 5  
 6  app = FastAPI(title="SlipGuard API")
 7  
 8  app.include_router(line_router)
 9  app.include_router(reports_router)
10  
11  
12  @app.get("/healthz")
13  def healthz():
14      return {"status": "ok"}
```

- `L3` import `router` จาก reports.py แล้วตั้งชื่อเล่นว่า `reports_router` (`as` = ตั้งชื่อใหม่ กันชนกับ `router` ของ LINE)
- `L9` `app.include_router(reports_router)` เอา endpoint ทั้งหมดในไฟล์ของแอ๊น (พร้อม prefix) มาต่อเข้าแอป · **ถ้าลบบรรทัดนี้:** `/api/v1/reports/...` จะได้ 404 หน้าเว็บขึ้นกล่อง error

**`backend/app/line/webhook.py`** (ไฟล์ของแพรว — ตาม handoff แอ๊นแก้ import ในฟังก์ชันนี้ใน PR #25)

`backend/app/line/webhook.py` · บรรทัด 135–147

```python
135  def summary_text(session, shop_id: str, period: str) -> str:
136      """ข้อความยอดสรุป จากฟังก์ชันของแอ๊น
137  
138      summarize() อยู่ใน app/api/store.py (อ่านตาราง slips) · summary_message() อยู่ใน app/api/summary.py (จัดข้อความ)
139      period = "day" / "month" / "year" · เรียกไม่ได้ → ตอบว่ายังไม่พร้อม (ไม่ล่ม)
140      """
141      try:
142          from app.api.store import summarize
143          from app.api.summary import summary_message
144          return summary_message(summarize(session, shop_id, period))
145      except Exception:  # noqa: BLE001 — ส่วนนี้ห้ามทำให้ webhook ล่ม
146          logger.exception("summary failed: shop=%s period=%s", shop_id, period)
147          return "ยังดูยอดไม่ได้ตอนนี้ค่ะ ลองใหม่อีกครั้งภายหลังนะคะ"
```

- `L135` แพรวเรียกฟังก์ชันนี้จาก `handle_text()` (L217 ของ webhook.py) หลังตรวจแล้วว่าคนพิมพ์คือแม่ค้าของร้าน
- `L141` `try:`
- `L142` `from app.api.store import summarize` — import ภายในฟังก์ชัน (โหลดตอนใช้จริงครั้งแรก) · **เดิม import จาก `app.api.summary` ซึ่งไม่มีฟังก์ชันนี้** → ImportError ทุกครั้ง
- `L143` `summary_message` อยู่ใน summary.py
- `L144` หายอด → จัดข้อความ → คืนให้ webhook ส่ง reply
- `L145–147` `except Exception:` จับ error ทุกแบบ แล้วตอบข้อความสุภาพ webhook จะไม่ล่ม · **ข้อเสีย:** กลืน error เงียบ บั๊ก import ผิดไฟล์จึงไม่มีใครเห็น (แม่ค้าได้ "ยังดูยอดไม่ได้" ทุกครั้ง) → แอ๊นจึงเพิ่มเทสต์ `test_line_summary_command_reads_real_db` (ดู 3.14) ที่เรียกฟังก์ชันนี้กับตารางจริง ถ้า import ผิดอีกเทสต์จะแดง

<a id="f-layout"></a>
### 3.10 `frontend/app/layout.tsx` — โครงที่ทุกหน้าใช้

> **ทำอะไร:** Next.js (App Router) บังคับให้มี `layout.tsx` ที่ root ทำหน้าที่เป็นโครง `<html><body>` และโหลด CSS ทั้งเว็บ · **รับ:** `children` (เนื้อหาของหน้า เช่น page.tsx) · **คืน:** JSX ของ HTML ทั้งหน้า

`frontend/app/layout.tsx` · บรรทัด 1–16

```tsx
 1  // โครงหน้าเว็บที่ทุกหน้าใช้ร่วมกัน (แอ๊น)
 2  // Next.js (App Router) จะเอาเนื้อหาของแต่ละหน้า (เช่น page.tsx) มาใส่ตรง {children}
 3  
 4  import "./globals.css"; // สไตล์ทั้งเว็บ (สี, ขนาดตัวอักษร, การ์ด, กราฟ)
 5  
 6  // ชื่อแท็บของเบราว์เซอร์
 7  export const metadata = { title: "SlipGuard หลังบ้าน" };
 8  
 9  export default function RootLayout({ children }: { children: React.ReactNode }) {
10    return (
11      // lang="th" ให้เบราว์เซอร์/โปรแกรมอ่านหน้าจอรู้ว่าเนื้อหาเป็นภาษาไทย
12      <html lang="th">
13        <body>{children}</body>
14      </html>
15    );
16  }
```

- `L1–2` คอมเมนต์ (`//` = คอมเมนต์ 1 บรรทัดใน TypeScript)
- `L4` `import "./globals.css"` import ไฟล์ CSS เพื่อผลข้างเคียง (ให้ Next.js ใส่สไตล์ในทุกหน้า) ไม่ได้รับค่าอะไรกลับ · **ถ้าลบ:** หน้าเว็บไม่มีสีและเลย์เอาต์เลย
- `L6` คอมเมนต์
- `L7` `export const metadata = { title: ... }` — Next.js อ่านตัวแปรที่ export ชื่อ `metadata` เองแล้วสร้าง `<title>` ให้ = ชื่อแท็บของเบราว์เซอร์
- `L9` `export default function RootLayout({ children }: { children: React.ReactNode })`
  - `export default` = ตัวหลักของไฟล์ (Next.js ใช้ตัวนี้)
  - `{ children }` = destructuring ดึงค่า `children` ออกจาก props object
  - `: { children: React.ReactNode }` = type ของ props (`ReactNode` = อะไรก็ได้ที่ React แสดงได้)
- `L10` `return (` คืน JSX (ไวยากรณ์คล้าย HTML ใน JavaScript)
- `L11` คอมเมนต์ใน JSX ต้องเขียน `{/* ... */}` (ตรงนี้อยู่ก่อน `<html>` จึงใช้ `//` ได้)
- `L12` `<html lang="th">` บอกเบราว์เซอร์/โปรแกรมอ่านหน้าจอว่าเนื้อหาภาษาไทย (ตัดคำ/อ่านออกเสียงถูก)
- `L13` `<body>{children}</body>` วงเล็บปีกกาใน JSX = ใส่ค่าจาก JavaScript · Next.js ใส่ page.tsx ตรงนี้
- `L14–16` ปิดแท็ก/ฟังก์ชัน

---

<a id="f-autorefresh"></a>
### 3.11 `frontend/app/AutoRefresh.tsx` — รีเฟรชทุก 15 วินาที

> - **ทำอะไร:** ตั้งตัวจับเวลาในเบราว์เซอร์ ทุก N วินาทีสั่งให้ server render หน้าใหม่ (ได้ตัวเลขล่าสุด) และแสดงป้าย "● อัปเดตอัตโนมัติ"
> - **ทำไม:** ตอน demo ส่งสลิปใน LINE แล้วยอดบนเว็บเปลี่ยนเองภายใน 15 วินาที ไม่ต้องกดรีเฟรช
> - **รับ (props):** `seconds` (number) · **ต่อกับ:** page.tsx L232 `<AutoRefresh seconds={15} />`

`frontend/app/AutoRefresh.tsx` · บรรทัด 1–20

```tsx
 1  // ตัวรีเฟรชข้อมูลอัตโนมัติ (แอ๊น)
 2  // "use client" = ไฟล์นี้รันในเบราว์เซอร์ (ต้องใช้ setInterval ซึ่งฝั่ง server ทำไม่ได้)
 3  // router.refresh() ขอให้ server render page.tsx ใหม่ → ได้ยอดล่าสุดโดยไม่ต้องโหลดทั้งหน้า/ไม่กระพริบ
 4  "use client";
 5  
 6  import { useRouter } from "next/navigation";
 7  import { useEffect } from "react";
 8  
 9  export default function AutoRefresh({ seconds }: { seconds: number }) {
10    const router = useRouter();
11  
12    useEffect(() => {
13      // ตั้งเวลาให้รีเฟรชทุก `seconds` วินาที
14      const id = setInterval(() => router.refresh(), seconds * 1000);
15      // ปิดตัวจับเวลาเมื่อออกจากหน้า (ไม่งั้นจะรีเฟรชค้างอยู่เบื้องหลัง)
16      return () => clearInterval(id);
17    }, [router, seconds]); // ตั้งใหม่เฉพาะตอนค่าพวกนี้เปลี่ยน
18  
19    return <span className="live">● อัปเดตอัตโนมัติทุก {seconds} วินาที</span>;
20  }
```

- `L1–3` คอมเมนต์
- `L4` `"use client";` — คำสั่งพิเศษของ Next.js ต้องอยู่บนสุดของไฟล์ (ก่อน import): ไฟล์นี้เป็น **Client Component** รันในเบราว์เซอร์ · **ทำไมต้องมี:** `useEffect`/`setInterval` ใช้ได้เฉพาะในเบราว์เซอร์ · **ถ้าลบ:** Next.js error ตอน build ว่าใช้ hook ใน Server Component ไม่ได้
- `L6` `useRouter` จาก `next/navigation` (ของ App Router ไม่ใช่ `next/router` ของ Pages Router แบบเก่า)
- `L7` `useEffect` hook ของ React
- `L9` `export default function AutoRefresh({ seconds }: { seconds: number })` component รับ props `seconds`
- `L10` `const router = useRouter();` ได้ตัวควบคุมการนำทาง · hook ต้องเรียกบนสุดของ component (กฎของ React hooks)
- `L12` `useEffect(() => { ... }, [router, seconds]);` = โค้ดที่รัน **หลัง** component แสดงบนจอ และรันใหม่เมื่อค่าใน array `[router, seconds]` (dependency array) เปลี่ยน · **ถ้าไม่ใส่ array:** รันใหม่ทุกครั้งที่ render → ตั้ง interval ซ้อนกันเรื่อยๆ
- `L13` คอมเมนต์
- `L14` `setInterval(() => router.refresh(), seconds * 1000)` เรียกฟังก์ชันซ้ำทุก N มิลลิวินาที · `() => ...` = arrow function · `router.refresh()` = ขอให้ server render Server Component ของหน้านี้ใหม่ (page.tsx รันอีกรอบ → fetch API ใหม่) แล้วอัปเดตเฉพาะส่วนที่เปลี่ยน **โดยไม่โหลดทั้งหน้า** หน้าจอไม่กระพริบ ตำแหน่ง scroll ไม่หาย · `const id` เก็บรหัสตัวจับเวลาไว้ยกเลิก
- `L15` คอมเมนต์
- `L16` `return () => clearInterval(id);` — ฟังก์ชันที่ return จาก useEffect = **cleanup** รันตอน component ถูกถอดออก (ออกจากหน้า) หรือก่อน effect รันใหม่ · **ถ้าลบ:** ตัวจับเวลาค้างอยู่เบื้องหลัง และเมื่อ effect รันใหม่จะมีหลายตัวซ้อน
- `L17` dependency array + คอมเมนต์
- `L19` แสดงป้าย `<span className="live">` · `className` = attribute `class` ของ HTML (`class` เป็นคำสงวนใน JavaScript) · `{seconds}` ใส่ค่าตัวแปร
- `L20` ปิดฟังก์ชัน

<a id="f-page"></a>
### 3.12 `frontend/app/page.tsx` — หน้าสรุปยอด

> **ภาพรวมไฟล์**
> - **ทำอะไร:** หน้า `/` ทั้งหน้า: อ่านค่าจาก URL → เรียก API 2 ตัวพร้อมกัน → คำนวณค่าสำหรับวาด → คืน JSX (แถบเมนู, เลื่อนช่วง, การ์ดยอด, กราฟ/ปฏิทิน, ตารางสลิปล่าสุด, ท้ายหน้า)
> - **ชนิด:** **Server Component** (ไม่มี `"use client"`) โค้ดทั้งไฟล์รันใน server ของ Next.js ไม่ได้รันในเบราว์เซอร์
> - **รับ:** `searchParams` (ค่าใน URL) · **ข้อมูลจาก:** `/api/v1/reports/summary` และ `/api/v1/reports/recent` · **ส่งออก:** HTML ที่มีตัวเลขครบแล้ว
> - **State:** ไม่มี `useState` สถานะทั้งหมดอยู่ใน URL (`mode`, `date`, `filter`) ปุ่มทุกปุ่มเป็นลิงก์

#### 3.12.1 คอมเมนต์หัวไฟล์

`frontend/app/page.tsx` · บรรทัด 1–13

```tsx
 1  // หน้าสรุปยอดของแม่ค้า (แอ๊น) — โทนสีมิ้นต์/เทาเข้มแบบ flat ผสมกับเลย์เอาต์การ์ดแบบเดิม
 2  // - อ่านยอดจาก API /api/v1/reports/summary และสลิปล่าสุดจาก /api/v1/reports/recent
 3  // - API อ่านจากตาราง slips (ของวี) → ส่งสลิปใน LINE แล้วยอดบนหน้านี้เปลี่ยนตาม
 4  // - แผนเร่ง: ตัด LIFF แล้ว demo หน้านี้บนเว็บ (http://localhost:3000)
 5  //
 6  // - ข้อมูลทุกตัวเลขบนหน้านี้มาจากตาราง slips ของวีเท่านั้น (ไม่มีข้อมูลปลอมใน API)
 7  //
 8  // โครงหน้า (บนลงล่าง): แถบเมนูสีเข้ม → เลื่อนช่วง ‹ › → การ์ดยอดรวม + การ์ด 3 สถานะ
 9  //                     → กราฟแท่ง (วัน/ปี) หรือปฏิทิน (เดือน) → ตารางสลิปล่าสุด + ตัวกรอง → ท้ายหน้าสีเข้ม
10  //
11  // หน้านี้เป็น "Server Component" (ไม่มี "use client") แปลว่าโค้ดทั้งไฟล์รันใน server ของ Next.js
12  // ไม่ได้รันในเบราว์เซอร์ → เรียก API ด้วยชื่อ service ของ docker (http://api:8000) ได้
13  // และเบราว์เซอร์ได้รับ HTML ที่มีตัวเลขครบแล้ว ไม่ต้องรอโหลดข้อมูลเอง
```

- `L1–13` อธิบายแหล่งข้อมูล (L2–6) โครงหน้าบนลงล่าง (L8–9) และ **ทำไมเป็น Server Component** (L11–13): เรียก API ด้วยชื่อ service ของ docker (`http://api:8000`) ได้ เพราะ fetch เกิดใน container ของ frontend ซึ่งอยู่ในเครือข่าย docker เดียวกับ api · และเบราว์เซอร์ได้ HTML ที่มีตัวเลขแล้ว ไม่ต้องรอโหลดข้อมูลเอง

#### 3.12.2 Import และที่อยู่ API

`frontend/app/page.tsx` · บรรทัด 15–21

```tsx
15  import Link from "next/link";
16  
17  import AutoRefresh from "./AutoRefresh";
18  
19  // ที่อยู่ของ API: ใน docker ตั้งไว้ใน docker-compose.yml = http://api:8000
20  // ถ้ารัน npm run dev บนเครื่องเอง (ไม่ใช้ docker) จะใช้ http://localhost:8000
21  const API = process.env.NEXT_PUBLIC_API_BASE_URL ?? "http://localhost:8000";
```

- `L15` `Link` ของ Next.js = แท็ก `<a>` ที่เปลี่ยนหน้าแบบ client-side (ไม่โหลดใหม่ทั้งหน้า) และ prefetch ล่วงหน้า
- `L17` import component รีเฟรช (Server Component ใส่ Client Component ไว้ข้างในได้)
- `L19–20` คอมเมนต์
- `L21` `process.env.NEXT_PUBLIC_API_BASE_URL ?? "http://localhost:8000"` — อ่าน env (ใน docker-compose ตั้งเป็น `http://api:8000`) · `??` (nullish coalescing) = ถ้าซ้ายเป็น `null`/`undefined` ใช้ขวา (ต่างจาก `||` ที่ข้อความว่างก็ใช้ขวา) · กรณีรัน `npm run dev` เอง (run-local.bat) ไม่มี env จึงใช้ localhost

#### 3.12.3 เมนูและ type หลัก

`frontend/app/page.tsx` · บรรทัด 23–34

```tsx
23  // เมนูด้านบน: key = ค่า mode ที่ส่งให้ API, label = ข้อความที่แสดง
24  const MODES = [
25    { key: "daily", label: "วันนี้" },
26    { key: "monthly", label: "เดือนนี้" },
27    { key: "yearly", label: "ปีนี้" },
28  ] as const;
29  
30  // type ของ mode = "daily" | "monthly" | "yearly" (ดึงจาก MODES ข้างบน จะได้ไม่ต้องพิมพ์ซ้ำ)
31  type Mode = (typeof MODES)[number]["key"];
32  type Verdict = "passed" | "suspicious" | "failed";
33  // ตัวกรองตารางสลิปล่าสุด: "all" = ทุกผล
34  type Filter = Verdict | "all";
```

- `L23` คอมเมนต์
- `L24–28` `MODES` array ของ object 3 เมนู · `key` = ค่าที่ส่งให้ API, `label` = ข้อความบนจอ
- `L28` `as const` = บอก TypeScript ให้ถือค่าตามตัวอักษร (`"daily"` ไม่ใช่แค่ `string`) และอ่านอย่างเดียว · **ถ้าลบ:** type `Mode` ใน L31 จะกลายเป็น `string`
- `L30` คอมเมนต์
- `L31` `type Mode = (typeof MODES)[number]["key"];` — `typeof MODES` = type ของ array · `[number]` = type ของสมาชิกตัวใดก็ได้ · `["key"]` = type ของช่อง key → `"daily" | "monthly" | "yearly"` · เพิ่มเมนูใน MODES แล้ว type เปลี่ยนตามเอง
- `L32` `Verdict` union type 3 ค่า (ต้องตรงกับ enum ของ backend)
- `L33–34` `Filter` = Verdict หรือ `"all"`

#### 3.12.4 Type ของข้อมูลจาก API และป้ายผลตรวจ

`frontend/app/page.tsx` · บรรทัด 36–63

```tsx
36  // รูปแบบข้อมูลที่ API /summary ส่งกลับ (ต้องตรงกับ build_summary() ใน backend/app/api/summary.py)
37  type Summary = {
38    mode: string;
39    start: string; // วันแรกของช่วง "YYYY-MM-DD"
40    end: string; // วันสุดท้ายของช่วง (รวมวันนั้น)
41    total_amount: string; // เป็นข้อความ เช่น "3098.00" กันทศนิยมเพี้ยน
42    slip_count: number; // จำนวนสลิปที่ผ่าน
43    status_counts: Record<Verdict, number>;
44    series: { label: string; amount: string; slip_count: number }[]; // ข้อมูลกราฟ/ปฏิทิน
45  };
46  
47  // 1 แถวของตารางสลิปล่าสุด (ต้องตรงกับ recent_item() ใน backend/app/api/store.py)
48  type RecentSlip = {
49    received_at: string; // เวลาไทยแบบ ISO เช่น "2026-11-19T21:05+07:00"
50    bank: string | null;
51    amount: string | null; // null = OCR อ่านยอดไม่ได้
52    verdict: Verdict;
53    fraud_score: number;
54    reasons: string[];
55    reviewed: boolean;
56  };
57  
58  // ป้ายของแต่ละผลตรวจ: ไอคอน + ข้อความ + ชื่อ class สี (สีอยู่ใน globals.css)
59  const VERDICT: Record<Verdict, { icon: string; text: string; tone: string }> = {
60    passed: { icon: "✅", text: "ผ่าน", tone: "ok" },
61    suspicious: { icon: "⚠️", text: "รอตรวจ", tone: "warn" },
62    failed: { icon: "❌", text: "ไม่ผ่าน", tone: "bad" },
63  };
```

- `L36` คอมเมนต์: **ต้องตรงกับ `build_summary()`** (สัญญาระหว่าง backend–frontend)
- `L37–45` `type Summary` · `total_amount: string` (backend ส่งเป็นข้อความกันทศนิยมเพี้ยน) · `Record<Verdict, number>` = object ที่มี key passed/suspicious/failed ค่าเป็น number · `series: {...}[]` = array ของ object
- `L47` คอมเมนต์: ต้องตรงกับ `recent_item()`
- `L48–56` `type RecentSlip` · `string | null` = อาจเป็น null (OCR อ่านไม่ได้)
- `L58` คอมเมนต์
- `L59` `const VERDICT: Record<Verdict, { icon: string; text: string; tone: string }>` — map จากผลตรวจ → ไอคอน/ข้อความ/ชื่อ class สี · ใช้ `Record<Verdict, ...>` TypeScript บังคับให้มีครบทั้ง 3 key
- `L60–62` `tone` = `ok`/`warn`/`bad` ไปกำหนดสีใน globals.css (`.badge.ok`, `.count.warn` ฯลฯ)

#### 3.12.5 ตัวกรอง ปฏิทิน จำนวนแถว

`frontend/app/page.tsx` · บรรทัด 65–78

```tsx
65  // ปุ่มตัวกรองของตารางสลิปล่าสุด (เหมือนเมนู "SORT BY FILTER" ใน UI kit)
66  const FILTERS: { key: Filter; label: string }[] = [
67    { key: "all", label: "ทั้งหมด" },
68    { key: "passed", label: "✅ ผ่าน" },
69    { key: "suspicious", label: "⚠️ รอตรวจ" },
70    { key: "failed", label: "❌ ไม่ผ่าน" },
71  ];
72  
73  // ชื่อวันย่อสำหรับหัวปฏิทิน เริ่มวันอาทิตย์ (ตรงกับ getUTCDay(): 0 = อาทิตย์)
74  const WEEKDAYS = ["อา", "จ", "อ", "พ", "พฤ", "ศ", "ส"];
75  
76  // จำนวนสลิปที่แสดงในตาราง และจำนวนที่ดึงมาก่อนกรอง (ดึงเผื่อไว้ กรองแล้วจะยังเหลือพอแสดง)
77  const SHOW_ROWS = 8;
78  const FETCH_ROWS = 50; // = ค่าสูงสุดที่ API /recent ยอม
```

- `L65–71` `FILTERS` ปุ่มกรอง 4 ปุ่ม (type `{ key: Filter; label: string }[]`)
- `L73–74` `WEEKDAYS` เริ่มวันอาทิตย์ ตรงกับ `getUTCDay()` ของ JavaScript (0 = อาทิตย์) · **ถ้าเริ่มวันจันทร์:** ต้องแก้ L205 ด้วย ไม่งั้นวันที่จะตรงคอลัมน์ผิด
- `L76` คอมเมนต์
- `L77` แสดง 8 แถว
- `L78` ดึงมา 50 (= `le=50` ของ API) เผื่อกรองแล้วยังเหลือพอแสดง

#### 3.12.6 `getJson()` — เรียก API แบบไม่ให้หน้าพัง

`frontend/app/page.tsx` · บรรทัด 80–93

```tsx
80  // เรียก API แล้วคืนข้อมูล หรือคืน "ข้อความ" บอกว่าทำไมอ่านไม่ได้ (หน้าเว็บจะได้ไม่พังทั้งหน้า)
81  async function getJson<T>(path: string): Promise<T | string> {
82    try {
83      // no-store = ไม่ cache ทุกครั้งที่เปิดหน้าได้ยอดล่าสุดจากฐานข้อมูล
84      const res = await fetch(`${API}${path}`, { cache: "no-store" });
85      if (res.ok) return (await res.json()) as T;
86      // API ตอบ error (เช่น 503 ยังไม่ได้สร้างตาราง) → เอาข้อความ detail ที่ backend เขียนไว้มาแสดง
87      const body = await res.json().catch(() => null);
88      return body?.detail ?? `API ตอบกลับ ${res.status}`;
89    } catch {
90      // ต่อ API ไม่ได้เลย (API ยังไม่เปิด / ชื่อ host ผิด)
91      return `ยังเชื่อมต่อ API ไม่ได้ (${API})`;
92    }
93  }
```

> **รับ:** path ของ API · **คืน:** `Promise<T | string>` = ข้อมูล (type T) หรือข้อความบอกสาเหตุที่อ่านไม่ได้ · **ทำไมคืนข้อความแทน throw:** ถ้า throw ทั้งหน้าจะพัง (Next.js แสดงหน้า error) แบบนี้หน้ายังแสดง แค่ส่วนนั้นขึ้นกล่อง error พร้อมวิธีแก้

- `L80` คอมเมนต์
- `L81` `async function getJson<T>(path: string): Promise<T | string>` · `async` = ฟังก์ชันที่ใช้ `await` ได้ และคืน Promise เสมอ · `<T>` = generic คนเรียกกำหนด type ผลเอง เช่น `getJson<Summary>(...)`
- `L82` `try {`
- `L83` คอมเมนต์
- `L84` `await fetch(url, { cache: "no-store" })` · `await` = รอให้ได้ผลก่อนไปบรรทัดถัดไป · `cache: "no-store"` = ห้าม Next.js cache ผล fetch (Next 14 cache fetch เป็นค่าเริ่มต้น) · **ถ้าลบ:** ยอดบนเว็บอาจค้างเป็นค่าเก่า
- `L85` `res.ok` = true เมื่อสถานะ 200–299 → `await res.json()` แปลง body เป็น object · `as T` บอก TypeScript ว่าเป็น type T (ไม่ได้ตรวจจริงตอนรัน)
- `L86` คอมเมนต์
- `L87` อ่าน body ของ error · `.catch(() => null)` ถ้า body ไม่ใช่ JSON ได้ null แทนการพัง
- `L88` ``body?.detail ?? `API ตอบกลับ ${res.status}` `` · `?.` (optional chaining) = ถ้า body เป็น null ไม่ error ได้ undefined · ใช้ข้อความ `detail` ที่ FastAPI ส่ง (เช่น `DB_ERROR_DETAIL` หรือเหตุผล 422) ไม่มีก็บอกเลขสถานะ · `` `...${}` `` = template literal (คล้าย f-string)
- `L89` `} catch {` จับ error ของ fetch เอง (ต่อไม่ได้เลย เช่น API ยังไม่เปิด)
- `L90–91` บอกที่อยู่ที่พยายามต่อ (ช่วย debug ว่า env ผิดไหม)
- `L92–93` ปิด

#### 3.12.7 ตัวช่วยวันที่

`frontend/app/page.tsx` · บรรทัด 95–104

```tsx
 95  // ---------- ตัวช่วยเรื่องวันที่ (ทำงานกับข้อความ "YYYY-MM-DD" ตลอด กันปัญหา timezone) ----------
 96  
 97  // วันนี้ตามเวลาไทย เป็น "YYYY-MM-DD" (locale en-CA จัดรูปแบบวันที่เป็น ปี-เดือน-วัน พอดี)
 98  function todayInBangkok(): string {
 99    return new Intl.DateTimeFormat("en-CA", { timeZone: "Asia/Bangkok" }).format(new Date());
100  }
101  
102  // "YYYY-MM-DD" → Date ตอนเที่ยงคืน UTC (ใช้ UTC ทั้งไฟล์ วันที่จะได้ไม่เลื่อนตาม timezone ของเครื่อง)
103  const toDate = (iso: string) => new Date(`${iso}T00:00:00Z`);
104  const toIso = (d: Date) => d.toISOString().slice(0, 10);
```

- `L95` คอมเมนต์หัวส่วน: ทำงานกับข้อความ `"YYYY-MM-DD"` ตลอด กันปัญหา timezone
- `L97` คอมเมนต์
- `L98–100` `todayInBangkok()` · `Intl.DateTimeFormat("en-CA", { timeZone: "Asia/Bangkok" })` ตัวจัดรูปแบบวันที่ locale แคนาดา (ให้ผลแบบ `2026-11-21` พอดี) ในเขตเวลาไทย · `new Date()` = ตอนนี้ · **ทำไมต้องระบุเขตเวลา:** server ใน docker เป็น UTC ช่วง 00:00–06:59 ไทยจะได้ "เมื่อวาน"
- `L102` คอมเมนต์
- `L103` `toDate` arrow function แปลง `"2026-11-18"` → Date ตอน 00:00 UTC (`Z` = UTC)
- `L104` `toIso` แปลงกลับ: `toISOString()` ได้ `"2026-11-18T00:00:00.000Z"` แล้ว `.slice(0, 10)` ตัดเอา 10 ตัวแรก
- **ทำไมใช้ UTC ทั้งไฟล์:** ทั้งสร้างและอ่านเป็น UTC วันที่จึงไม่เลื่อนตาม timezone ของเครื่องที่รัน (ถ้าใช้ `new Date("2026-11-18")` แล้ว `getDate()` บางเครื่องได้ 17)

#### 3.12.8 เลื่อนช่วง และช่วงปัจจุบัน

`frontend/app/page.tsx` · บรรทัด 106–120

```tsx
106  // เลื่อนช่วงไปข้างหน้า (step = 1) หรือย้อนหลัง (step = -1) ตามโหมด คืนวันแรกของช่วงใหม่
107  function shiftPeriod(mode: Mode, startIso: string, step: number): string {
108    const d = toDate(startIso);
109    if (mode === "daily") d.setUTCDate(d.getUTCDate() + step);
110    if (mode === "monthly") d.setUTCMonth(d.getUTCMonth() + step, 1); // วันที่ 1 ของเดือนใหม่
111    if (mode === "yearly") d.setUTCFullYear(d.getUTCFullYear() + step, 0, 1); // 1 ม.ค. ของปีใหม่
112    return toIso(d);
113  }
114  
115  // วันแรกของช่วงที่ "วันนี้" อยู่ ใช้เช็กว่ากดถัดไปได้ไหม (ห้ามไปอนาคต)
116  function currentPeriodStart(mode: Mode, today: string): string {
117    if (mode === "monthly") return `${today.slice(0, 8)}01`; // "2026-11-01"
118    if (mode === "yearly") return `${today.slice(0, 4)}-01-01`;
119    return today;
120  }
```

- `L106` คอมเมนต์
- `L107` `shiftPeriod(mode, startIso, step)` รับวันแรกของช่วง + ทิศทาง (+1/−1) คืนวันแรกของช่วงใหม่
- `L108` แปลงเป็น Date (`const` = ตัวแปรที่ผูกค่าครั้งเดียว แต่ object ข้างในแก้ได้)
- `L109` daily: `setUTCDate(วัน + step)` Date จัดการข้ามเดือน/ปีให้เอง (31 + 1 → วันที่ 1 เดือนถัดไป)
- `L110` monthly: `setUTCMonth(เดือน + step, 1)` อาร์กิวเมนต์ที่ 2 ตั้งวันที่เป็น 1 · **ถ้าไม่ใส่ 1:** 31 ม.ค. + 1 เดือน → "31 ก.พ." ล้นเป็น 3 มี.ค.
- `L111` yearly: `setUTCFullYear(ปี + step, 0, 1)` = 1 ม.ค. (เดือนใน JavaScript นับจาก 0)
- `L112` คืนเป็นข้อความ
- `L115` คอมเมนต์
- `L116–120` `currentPeriodStart` วันแรกของช่วงที่วันนี้อยู่ · `today.slice(0, 8)` = `"2026-11-"` ต่อ `"01"` · ใช้เช็กว่าห้ามกดไปอนาคต (L203)

#### 3.12.9 หัวข้อช่วงและป้ายแกนกราฟ

`frontend/app/page.tsx` · บรรทัด 122–139

```tsx
122  // หัวข้อของช่วง แบบไทย พ.ศ. เช่น "19 พ.ย. 2569", "พฤศจิกายน 2569", "ปี 2569"
123  // toLocaleDateString("th-TH") ใช้ปฏิทินพุทธ (พ.ศ.) ให้อัตโนมัติ
124  function periodTitle(mode: Mode, startIso: string): string {
125    const d = toDate(startIso);
126    if (mode === "daily")
127      return d.toLocaleDateString("th-TH", { day: "numeric", month: "short", year: "numeric", timeZone: "UTC" });
128    if (mode === "monthly") return d.toLocaleDateString("th-TH", { month: "long", year: "numeric", timeZone: "UTC" });
129    return `ปี ${d.getUTCFullYear() + 543}`;
130  }
131  
132  // ป้ายใต้แท่งกราฟ: รายวัน = ชั่วโมง, รายปี = ชื่อเดือนย่อ
133  function axisLabel(mode: Mode, label: string): string {
134    if (mode === "daily") return label.slice(0, 2); // "09:00" → "09"
135    return toDate(`${label}-01`).toLocaleDateString("th-TH", { month: "short", timeZone: "UTC" }); // "2026-01" → "ม.ค."
136  }
137  
138  // แสดงป้ายทุกกี่แท่ง (รายวันมี 24 แท่ง ถ้าแสดงทุกแท่ง ตัวหนังสือจะชนกัน)
139  const showTick = (mode: Mode, index: number) => mode !== "daily" || index % 3 === 0;
```

- `L122–123` คอมเมนต์
- `L124–130` `periodTitle` · `toLocaleDateString("th-TH", {...})` จัดรูปแบบไทย ใช้ปฏิทินพุทธ (พ.ศ.) อัตโนมัติ · `timeZone: "UTC"` ให้ตรงกับที่สร้างด้วย UTC (ไม่งั้นบางเครื่องวันเลื่อน) · daily → "19 พ.ย. 2569" · monthly → "พฤศจิกายน 2569" · yearly → `ปี ${ค.ศ. + 543}`
- `L132` คอมเมนต์
- `L133–136` `axisLabel` ป้ายใต้แท่ง: daily `"09:00".slice(0, 2)` = `"09"` · yearly `"2026-01"` + `"-01"` → Date → ชื่อเดือนย่อ `"ม.ค."` (ฟังก์ชันนี้ไม่ถูกเรียกในโหมดเดือน เพราะโหมดเดือนแสดงปฏิทิน)
- `L138` คอมเมนต์
- `L139` `showTick` daily มี 24 แท่ง แสดงป้ายทุก 3 แท่ง (`index % 3 === 0` → 00, 03, 06, ...) · `%` = เศษจากการหาร · `===` = เท่ากันแบบเข้มงวด (type ต้องตรงด้วย) · โหมดอื่นแสดงทุกแท่ง

#### 3.12.10 จัดรูปแบบเงินและความเข้มสี

`frontend/app/page.tsx` · บรรทัด 141–149

```tsx
141  // จัดรูปแบบเงินบาท: "3098.5" → "3,098.50"
142  const baht = (v: string) =>
143    Number(v).toLocaleString("th-TH", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
144  
145  // เงินแบบสั้นสำหรับช่องปฏิทินที่แคบ: 1250 → "1.3k", 890 → "890"
146  const shortBaht = (v: number) => (v >= 1000 ? `${(v / 1000).toFixed(1)}k` : String(Math.round(v)));
147  
148  // ความเข้มสีของช่องปฏิทิน 0–4 ตามยอดเทียบกับวันที่ยอดสูงสุด (0 = ไม่มียอด)
149  const heat = (amount: number, peak: number) => (amount <= 0 ? 0 : Math.min(4, Math.ceil((amount / peak) * 4)));
```

- `L141–143` `baht(v)` · `Number(v)` ข้อความ → ตัวเลข · `toLocaleString("th-TH", { minimumFractionDigits: 2, maximumFractionDigits: 2 })` ใส่ลูกน้ำ + ทศนิยม 2 ตำแหน่งเสมอ → `"3,098.50"` (แปลงเป็น number ตอนแสดงผลเท่านั้น ไม่ได้เอาไปบวกกัน จึงไม่มีปัญหาทศนิยมเพี้ยน)
- `L145–146` `shortBaht` สำหรับช่องปฏิทินที่แคบ · `≥ 1000` → `(v / 1000).toFixed(1)` + `"k"` (1250 → `"1.3k"`) · ไม่ถึงพัน → ปัดเป็นจำนวนเต็ม
- `L148–149` `heat(amount, peak)` ระดับสี 0–4 · `amount <= 0` → 0 (เทา) · ไม่งั้น `Math.ceil(amount / peak * 4)` ไม่เกิน 4 (`Math.min`) · ตัวอย่าง peak 1000: 1–250 → 1, 251–500 → 2, 501–750 → 3, 751–1000 → 4

#### 3.12.11 เวลาในตาราง ค่าจาก URL และการสร้างลิงก์

`frontend/app/page.tsx` · บรรทัด 151–168

```tsx
151  // เวลาของสลิปในตาราง: วันนี้แสดงแค่ "21:05" วันอื่นแสดง "18 พ.ย. 21:05"
152  function slipTime(iso: string, today: string): string {
153    const hhmm = iso.slice(11, 16); // ISO จาก API เป็นเวลาไทยอยู่แล้ว ตัดเอาส่วน HH:MM ได้เลย
154    const day = iso.slice(0, 10);
155    if (day === today) return hhmm;
156    return `${toDate(day).toLocaleDateString("th-TH", { day: "numeric", month: "short", timeZone: "UTC" })} ${hhmm}`;
157  }
158  
159  // ค่าจาก URL อาจเป็น string, string[] (ใส่ซ้ำ) หรือไม่มี → เอาตัวแรกเสมอ
160  const first = (v: string | string[] | undefined) => (Array.isArray(v) ? v[0] : v);
161  
162  // สร้างลิงก์ของหน้านี้ โดยใส่เฉพาะค่าที่มี (ไม่มี date/filter ก็ไม่ใส่ลง URL)
163  function href(mode: Mode, date?: string, filter?: Filter): string {
164    const q = new URLSearchParams({ mode });
165    if (date) q.set("date", date);
166    if (filter && filter !== "all") q.set("filter", filter);
167    return `/?${q}`;
168  }
```

- `L151–157` `slipTime(iso, today)` · `iso.slice(11, 16)` ตัด `"21:05"` จาก `"2026-11-19T21:05+07:00"` (ตำแหน่ง 11–15) · ตัดตรงๆ ได้เพราะ API ส่ง **เวลาไทย** มาแล้ว · `iso.slice(0, 10)` = วันที่ · วันนี้แสดงแค่เวลา วันอื่นแสดง "18 พ.ย. 21:05"
- `L159` คอมเมนต์
- `L160` `first(v)` ค่าใน URL อาจเป็น `string`, `string[]` (เมื่อใส่ซ้ำ `?mode=a&mode=b`) หรือ `undefined` → `Array.isArray(v) ? v[0] : v` เอาตัวแรกเสมอ
- `L162` คอมเมนต์
- `L163–168` `href(mode, date?, filter?)` สร้างลิงก์ของหน้านี้ · `date?: string` = พารามิเตอร์ไม่บังคับ · `new URLSearchParams({ mode })` (`{ mode }` = ย่อจาก `{ mode: mode }`) · ใส่ date/filter เฉพาะเมื่อมีและไม่ใช่ `"all"` · `` `/?${q}` `` URLSearchParams แปลงเป็นข้อความ `mode=daily&date=...` และ encode อักขระพิเศษให้

#### 3.12.12 ฟังก์ชันหลัก: อ่านค่าจาก URL

`frontend/app/page.tsx` · บรรทัด 170–184

```tsx
170  // ---------- หน้าเว็บ ----------
171  
172  export default async function Home({
173    searchParams,
174  }: {
175    searchParams: { [key: string]: string | string[] | undefined };
176  }) {
177    // 1) อ่านค่าจาก URL เช่น /?mode=monthly&date=2026-10-01&filter=failed (ค่าแปลกๆ ใช้ค่าเริ่มต้น)
178    const rawMode = first(searchParams.mode);
179    const mode: Mode = MODES.some((m) => m.key === rawMode) ? (rawMode as Mode) : "daily";
180    const rawDate = first(searchParams.date);
181    const date = rawDate && /^\d{4}-\d{2}-\d{2}$/.test(rawDate) ? rawDate : undefined; // กันค่าที่ไม่ใช่วันที่
182    const rawFilter = first(searchParams.filter);
183    const filter: Filter = FILTERS.some((f) => f.key === rawFilter) ? (rawFilter as Filter) : "all";
184    const today = todayInBangkok();
```

- `L170` คอมเมนต์หัวส่วน
- `L172` `export default async function Home(...)` component หลักของหน้า `/` · เป็น `async` ได้เพราะเป็น Server Component (รอ fetch ก่อน render · Client Component ทำแบบนี้ไม่ได้)
- `L173–176` รับ props `searchParams` · type `{ [key: string]: string | string[] | undefined }` = object ที่ key เป็นข้อความอะไรก็ได้ (index signature) · Next.js 14 ส่งเป็น object ธรรมดา (ใน Next 15 เปลี่ยนเป็น Promise ต้อง await)
- `L177` คอมเมนต์: **ค่าแปลกใช้ค่าเริ่มต้น** (ไม่เชื่อค่าที่ผู้ใช้พิมพ์ใน URL)
- `L178–179` mode: `MODES.some(m => m.key === rawMode)` (`.some` = มีอย่างน้อย 1 ตัวที่เงื่อนไขจริงไหม) ใช่ → ใช้ค่านั้น (`as Mode` บอก TypeScript) ไม่ใช่ → `"daily"`
- `L180–181` date: ต้องตรง regex `/^\d{4}-\d{2}-\d{2}$/` · `^` ต้นข้อความ `$` ท้ายข้อความ `\d{4}` ตัวเลข 4 หลัก · `.test()` คืน true/false · ไม่ผ่าน → `undefined` (ใช้วันนี้) · **ทำไม:** กันค่าแปลกถูกส่งต่อไป API แล้วได้ 422
- `L182–183` filter แบบเดียวกับ mode
- `L184` วันนี้ตามเวลาไทย

#### 3.12.13 เรียก API พร้อมกัน

`frontend/app/page.tsx` · บรรทัด 186–196

```tsx
186    // 2) เรียก API 2 ตัวพร้อมกัน (Promise.all) ไม่ต้องรอตัวแรกเสร็จก่อน → หน้าโหลดเร็วขึ้น
187    const query = new URLSearchParams({ mode, ...(date ? { date } : {}) });
188    const [summaryResult, recentResult] = await Promise.all([
189      getJson<Summary>(`/api/v1/reports/summary?${query}`),
190      getJson<{ items: RecentSlip[] }>(`/api/v1/reports/recent?limit=${FETCH_ROWS}`),
191    ]);
192    // ถ้าได้ข้อความ (string) แปลว่า error → ตั้งเป็น null แล้วแสดงกล่อง error แทน
193    const s = typeof summaryResult === "string" ? null : summaryResult;
194    const recentAll = typeof recentResult === "string" ? null : recentResult.items;
195    // กรองตามปุ่มที่เลือก แล้วเอาแค่ SHOW_ROWS ใบแรก (API เรียงใหม่ → เก่ามาให้แล้ว)
196    const recent = recentAll?.filter((r) => filter === "all" || r.verdict === filter).slice(0, SHOW_ROWS) ?? null;
```

- `L186` คอมเมนต์
- `L187` `new URLSearchParams({ mode, ...(date ? { date } : {}) })` · `...` (spread) กระจาย object เข้าไป · ถ้ามี date กระจาย `{ date }` ไม่มีกระจาย `{}` (ไม่เพิ่มอะไร)
- `L188–191` `const [a, b] = await Promise.all([p1, p2])` · `Promise.all` เริ่ม 2 คำขอ **พร้อมกัน** แล้วรอให้ครบทั้งคู่ · `[a, b] =` = array destructuring · **ทำไม:** เวลารวม ≈ ตัวที่ช้ากว่า แทนผลบวกของทั้งสอง · **ถ้าเขียน await ทีละตัว:** หน้าโหลดช้าลงเท่าเวลาของคำขอแรก
- `L192` คอมเมนต์
- `L193` `typeof summaryResult === "string" ? null : summaryResult` ได้ข้อความ = error → null (ตอนวาดเช็ก null แล้วแสดงกล่อง error) · TypeScript รู้ว่าในฝั่ง else เป็น `Summary` (type narrowing)
- `L194` แบบเดียวกัน แล้วเอา `.items`
- `L195` คอมเมนต์
- `L196` `recentAll?.filter(...).slice(0, SHOW_ROWS) ?? null` · `?.` ถ้า recentAll เป็น null ทั้งก้อนได้ undefined แล้ว `?? null` → null · `.filter` เก็บแถวที่ filter เป็น all หรือ verdict ตรง · `.slice(0, 8)` เอา 8 ตัวแรก (API เรียงใหม่ → เก่ามาแล้ว)

#### 3.12.14 เตรียมค่าที่ใช้วาด

`frontend/app/page.tsx` · บรรทัด 198–206

```tsx
198    // 3) เตรียมค่าที่ใช้วาด
199    const peak = s ? Math.max(0, ...s.series.map((b) => Number(b.amount))) : 0; // ยอดของช่องที่สูงที่สุด
200    const max = Math.max(1, peak); // แท่งสูงสุด = 100% (อย่างน้อย 1 กันหารด้วย 0 ตอนไม่มียอด)
201    const prev = s ? shiftPeriod(mode, s.start, -1) : undefined;
202    const next = s ? shiftPeriod(mode, s.start, 1) : undefined;
203    const canGoNext = !!next && next <= currentPeriodStart(mode, today); // ห้ามกดไปช่วงในอนาคต
204    // ปฏิทิน: ช่องว่างหน้าวันที่ 1 = วันในสัปดาห์ของวันที่ 1 (เช่น วันพุธ = 3 ช่อง)
205    const leadingBlanks = s && mode === "monthly" ? toDate(s.start).getUTCDay() : 0;
206    const updatedAt = new Date().toLocaleTimeString("th-TH", { timeZone: "Asia/Bangkok" });
```

- `L198` คอมเมนต์
- `L199` `peak` ยอดของช่องสูงสุด · `s.series.map(b => Number(b.amount))` ได้ array ตัวเลข · `...` กระจายเป็นอาร์กิวเมนต์ของ `Math.max` · ใส่ `0` นำหน้ากัน `Math.max()` ไม่มีอาร์กิวเมนต์ (ได้ `-Infinity`)
- `L200` `max` อย่างน้อย 1 กัน **หารด้วย 0** ตอนยอดเป็น 0 ทุกช่อง (L341, L314)
- `L201–202` วันแรกของช่วงก่อน/ถัดไป (ถ้าไม่มี s ได้ undefined)
- `L203` `canGoNext` · `!!next` แปลงเป็น boolean · `next <= currentPeriodStart(...)` เทียบ **ข้อความ** ได้ตรงๆ เพราะรูปแบบ `YYYY-MM-DD` เรียงตามตัวอักษร = เรียงตามเวลา · ห้ามกดไปช่วงในอนาคต
- `L204` คอมเมนต์
- `L205` ปฏิทิน: จำนวนช่องว่างก่อนวันที่ 1 = วันในสัปดาห์ของวันที่ 1 (`getUTCDay()` 0 = อาทิตย์) เช่น 1 พ.ย. 2569 เป็นวันอาทิตย์ → 0 ช่อง
- `L206` เวลาอัปเดตล่าสุด (เวลาไทย) แสดงท้ายหน้า

#### 3.12.15 JSX: แถบเมนูและหัวหน้า

`frontend/app/page.tsx` · บรรทัด 208–233

```tsx
208    return (
209      <>
210        {/* แถบเมนูสีเข้มด้านบน: โลโก้ซ้าย เมนูขวา (เมนูที่เลือกอยู่เป็นสีมิ้นต์) */}
211        <nav className="topbar">
212          <div className="topbar-inner">
213            <span className="logo">
214              SLIP<b>GUARD</b>
215            </span>
216            <div className="menu">
217              {MODES.map((m) => (
218                // กดเมนูแล้วกลับไปช่วงปัจจุบันเสมอ จึงไม่ใส่ date
219                <Link key={m.key} href={href(m.key)} className={m.key === mode ? "menu-item active" : "menu-item"}>
220                  {m.label}
221                </Link>
222              ))}
223            </div>
224          </div>
225        </nav>
226  
227        <main className="page">
228          {/* หัวข้อหน้า + ป้ายรีเฟรชอัตโนมัติ */}
229          <header className="header">
230            <p className="muted">สรุปยอดรับเงินผ่านสลิป</p>
231            {/* รีเฟรชข้อมูลเองทุก 15 วินาที ตอน demo ส่งสลิปใน LINE แล้วจะเห็นขึ้นเองไม่ต้องกด */}
232            <AutoRefresh seconds={15} />
233          </header>
```

- `L208` `return (` เริ่ม JSX
- `L209` `<>` Fragment = ห่อหลาย element โดยไม่เพิ่ม div ใน HTML
- `L210` คอมเมนต์ JSX `{/* */}`
- `L211–212` `<nav className="topbar">` แถบสีเข้ม · `topbar-inner` จำกัดความกว้างกลางจอ
- `L213–215` โลโก้ `SLIP<b>GUARD</b>` (ส่วน GUARD เป็นตัวหนาสีมิ้นต์ตาม CSS)
- `L216` กล่องเมนู
- `L217` `{MODES.map((m) => (...))}` วนสร้างลิงก์เมนูจาก array (`.map` คืน array ของ JSX ซึ่ง React แสดงต่อกันได้)
- `L218` คอมเมนต์: กดเมนูกลับไปช่วงปัจจุบันเสมอ จึงไม่ใส่ date
- `L219` `<Link key={m.key} href={href(m.key)} className={เงื่อนไข ? "menu-item active" : "menu-item"}>` · `key` = ตัวระบุที่ React ใช้แยกรายการในลิสต์ (ต้องไม่ซ้ำ) · **ถ้าไม่มี key:** React เตือนใน console และอาจอัปเดตรายการผิดตัว · เมนูที่ตรงกับ mode ปัจจุบันได้ class `active` (พื้นมิ้นต์)
- `L220–222` ข้อความเมนู · ปิด
- `L223–225` ปิดแท็ก
- `L227` `<main className="page">` เนื้อหาหลัก
- `L228–233` หัวหน้า: ข้อความ "สรุปยอดรับเงินผ่านสลิป" + `<AutoRefresh seconds={15} />` (`{15}` = ส่งเป็น number ถ้าเขียน `seconds="15"` จะเป็น string)

#### 3.12.16 JSX: กล่อง error (เมื่ออ่านยอดไม่ได้)

`frontend/app/page.tsx` · บรรทัด 235–251

```tsx
235          {!s ? (
236            // อ่านยอดไม่ได้: บอกเหตุผล + วิธีเช็กที่ใช้บ่อย
237            <section className="card error">
238              <strong>อ่านยอดไม่ได้</strong>
239              <p>{summaryResult as string}</p>
240              <ul>
241                <li>
242                  เปิด <code>http://localhost:8000/healthz</code> ต้องเห็น <code>{'{"status":"ok"}'}</code>
243                </li>
244                <li>
245                  docker: <code>docker compose logs api --tail 50</code>
246                </li>
247                <li>
248                  ยังไม่มีตาราง: <code>python -m app.verify.db</code> (หรือรัน <code>run-local.bat</code>)
249                </li>
250              </ul>
251            </section>
```

- `L235` `{!s ? ( ... ) : ( ... )}` = if-else ใน JSX (ternary) · s เป็น null → แสดงกล่อง error
- `L236` คอมเมนต์
- `L237` `className="card error"` = 2 class (การ์ด + พื้นแดงอ่อน)
- `L238–239` หัวข้อ + ข้อความจาก API (`summaryResult as string` บอก TypeScript ว่าตรงนี้เป็น string แน่นอน เพราะ s เป็น null)
- `L240–250` วิธีเช็ก 3 ข้อที่เจอบ่อย · `L242` `{'{"status":"ok"}'}` ต้องห่อเป็นข้อความ JavaScript เพราะ `{ }` ใน JSX หมายถึง "ใส่ค่า JavaScript" ถ้าพิมพ์ตรงๆ จะ error
- `L251` ปิดกล่อง

#### 3.12.17 JSX: แถวเลื่อนช่วง

`frontend/app/page.tsx` · บรรทัด 252–272

```tsx
252          ) : (
253            <>
254              {/* เลื่อนดูช่วงก่อนหน้า / ถัดไป */}
255              <div className="period">
256                <Link className="arrow" href={href(mode, prev, filter)} aria-label="ช่วงก่อนหน้า">
257                  ‹
258                </Link>
259                <span className="period-title">{periodTitle(mode, s.start)}</span>
260                {canGoNext ? (
261                  <Link className="arrow" href={href(mode, next, filter)} aria-label="ช่วงถัดไป">
262                    ›
263                  </Link>
264                ) : (
265                  <span className="arrow disabled">›</span>
266                )}
267                {date && (
268                  <Link className="back" href={href(mode, undefined, filter)}>
269                    กลับไปปัจจุบัน
270                  </Link>
271                )}
272              </div>
```

- `L252–253` `) : (` ฝั่ง else ของ L235 (มีข้อมูล) · `<>` ห่อหลายส่วน
- `L254–255` คอมเมนต์ + กล่อง `period`
- `L256–258` ลิงก์ ‹ ไปช่วงก่อนหน้า (คง filter เดิม) · `aria-label` = คำอธิบายสำหรับโปรแกรมอ่านหน้าจอ (ปุ่มมีแค่สัญลักษณ์)
- `L259` หัวข้อช่วง
- `L260–266` ถ้า `canGoNext` เป็นลิงก์ › ไม่งั้นเป็น `<span>` สีจาง กดไม่ได้
- `L267–271` `{date && (...)}` = แสดงเฉพาะเมื่อ date มีค่า (ดูช่วงอื่นที่ไม่ใช่ปัจจุบัน) · `&&` ใน JSX: ซ้ายเป็นเท็จ → ไม่แสดงอะไร
- `L272` ปิด

#### 3.12.18 JSX: การ์ดตัวเลข

`frontend/app/page.tsx` · บรรทัด 274–290

```tsx
274              {/* การ์ดยอดรวม (กว้าง) + การ์ด 3 สถานะ เรียงในแถวเดียว */}
275              <section className="stats">
276                <div className="card total">
277                  <span className="muted">ยอดรับ (สลิปที่ผ่าน)</span>
278                  <strong className="amount">฿{baht(s.total_amount)}</strong>
279                  <span className="muted">จาก {s.slip_count.toLocaleString("th-TH")} สลิป</span>
280                </div>
281                {(Object.keys(VERDICT) as Verdict[]).map((v) => (
282                  <div key={v} className={`card count ${VERDICT[v].tone}`}>
283                    <span>
284                      {VERDICT[v].icon} {VERDICT[v].text}
285                    </span>
286                    <strong>{s.status_counts[v].toLocaleString("th-TH")}</strong>
287                    <span className="muted">ใบ</span>
288                  </div>
289                ))}
290              </section>
```

- `L274–275` คอมเมนต์ + `<section className="stats">` (grid 4 คอลัมน์ใน CSS)
- `L276–280` การ์ดยอดรวม (พื้นมิ้นต์) · `฿{baht(s.total_amount)}` · `s.slip_count.toLocaleString("th-TH")` ใส่ลูกน้ำให้จำนวนใบ
- `L281` `(Object.keys(VERDICT) as Verdict[]).map((v) => ...)` · `Object.keys` คืน `string[]` จึงต้อง `as Verdict[]` เพื่อใช้เป็น key ของ `VERDICT`/`status_counts` · ลำดับเป็นไปตามที่เขียนใน VERDICT (ผ่าน → รอตรวจ → ไม่ผ่าน)
- `L282` `` className={`card count ${VERDICT[v].tone}`} `` ต่อชื่อ class ด้วย template literal → `"card count ok"`
- `L283–285` ไอคอน + ข้อความ
- `L286` จำนวนใบของสถานะนั้น
- `L287–290` ปิด

#### 3.12.19 JSX: ปฏิทิน (โหมดเดือน)

`frontend/app/page.tsx` · บรรทัด 292–326

```tsx
292              {mode === "monthly" ? (
293                // โหมดเดือน: ปฏิทิน ช่องไหนยอดมากสีเข้ม (แบบการ์ดปฏิทินใน UI kit)
294                <section className="card">
295                  <div className="card-title">
296                    <strong>ยอดรายวัน</strong>
297                    <span className="muted">สูงสุด ฿{baht(String(peak))}</span>
298                  </div>
299                  <div className="calendar">
300                    {/* หัวคอลัมน์ อา–ส */}
301                    {WEEKDAYS.map((w) => (
302                      <span key={w} className="cal-weekday">
303                        {w}
304                      </span>
305                    ))}
306                    {/* ช่องว่างก่อนวันที่ 1 ให้วันที่ 1 ตรงกับวันในสัปดาห์ที่ถูกต้อง */}
307                    {Array.from({ length: leadingBlanks }, (_, i) => (
308                      <span key={`blank-${i}`} />
309                    ))}
310                    {s.series.map((b) => {
311                      const amount = Number(b.amount);
312                      const classes = [
313                        "cal-day",
314                        `heat-${heat(amount, max)}`,
315                        b.label === today ? "today" : "", // วันนี้มีกรอบ
316                        b.label > today ? "future" : "", // วันที่ยังมาไม่ถึงจางลง
317                      ].join(" ");
318                      return (
319                        <div key={b.label} className={classes} title={`฿${baht(b.amount)} (${b.slip_count} สลิป)`}>
320                          <span className="cal-num">{Number(b.label.slice(8, 10))}</span>
321                          <span className="cal-amount">{amount > 0 ? shortBaht(amount) : ""}</span>
322                        </div>
323                      );
324                    })}
325                  </div>
326                </section>
```

- `L292` `{mode === "monthly" ? (ปฏิทิน) : (กราฟแท่ง)}` · **ทำไม:** 30 แท่งเล็กเกินอ่าน ปฏิทินดูง่ายว่าวันไหนขายดี
- `L293–298` การ์ด + หัวข้อ "ยอดรายวัน" + ยอดสูงสุด
- `L299` `<div className="calendar">` grid 7 คอลัมน์
- `L300–305` หัวคอลัมน์ อา–ส (`key={w}` ชื่อวันไม่ซ้ำกัน)
- `L306–309` `Array.from({ length: leadingBlanks }, (_, i) => <span key={...} />)` สร้าง array ยาว n แล้วแปลงแต่ละตัวเป็น span ว่าง ดันวันที่ 1 ไปตรงคอลัมน์ที่ถูกต้อง · `_` = ไม่ใช้ค่า ใช้แค่ index `i`
- `L310` วนทุกวันของเดือน (`s.series` มี 1 ช่องต่อวัน จาก `bucket_labels`)
- `L311` ยอดเป็นตัวเลข
- `L312–317` สร้าง array ชื่อ class แล้ว `.join(" ")` · `heat-${heat(amount, max)}` สี 0–4 · `b.label === today ? "today" : ""` วันนี้มีกรอบ · `b.label > today ? "future" : ""` วันที่ยังมาไม่ถึงจาง (เทียบข้อความวันที่ได้)
- `L318–319` `return (<div ... title={...}>)` · `title` = tooltip ตอนชี้เมาส์ แสดงยอดเต็ม + จำนวนใบ
- `L320` เลขวัน `Number("2026-11-08".slice(8, 10))` = 8 (Number ตัด 0 ข้างหน้า)
- `L321` ยอดแบบสั้น ถ้ายอด 0 ไม่แสดง
- `L322–326` ปิด

#### 3.12.20 JSX: กราฟแท่ง (โหมดวัน/ปี)

`frontend/app/page.tsx` · บรรทัด 327–350

```tsx
327              ) : (
328                // โหมดวัน/ปี: กราฟแท่ง ทำด้วย div + CSS ไม่ต้องลงไลบรารีกราฟเพิ่ม
329                <section className="card">
330                  <div className="card-title">
331                    <strong>ยอดราย{mode === "daily" ? "ชั่วโมง" : "เดือน"}</strong>
332                    <span className="muted">สูงสุด ฿{baht(String(peak))}</span>
333                  </div>
334                  <div className="chart">
335                    {s.series.map((b, i) => (
336                      <div key={b.label} className="col">
337                        {/* title = ข้อความที่ขึ้นตอนเอาเมาส์ชี้แท่ง */}
338                        <div
339                          className="bar"
340                          title={`${b.label}: ฿${baht(b.amount)} (${b.slip_count} สลิป)`}
341                          style={{ height: `${(Number(b.amount) / max) * 100}%` }}
342                        />
343                        <span className="tick">{showTick(mode, i) ? axisLabel(mode, b.label) : ""}</span>
344                      </div>
345                    ))}
346                  </div>
347                </section>
348              )}
349            </>
350          )}
```

- `L327–328` ฝั่ง else ของ L292 + คอมเมนต์: กราฟทำด้วย div + CSS **ไม่ลงไลบรารีกราฟเพิ่ม** (เบา ไม่มี dependency)
- `L329–333` การ์ด + หัวข้อ "ยอดรายชั่วโมง" หรือ "ยอดรายเดือน" (`{mode === "daily" ? "ชั่วโมง" : "เดือน"}`)
- `L334` `<div className="chart">` (flex แนวนอน สูง 170px)
- `L335` วนทุกช่อง (24 หรือ 12) พร้อม index `i`
- `L336` คอลัมน์ 1 แท่ง
- `L337–342` แท่ง · `style={{ height: ... }}` = inline style (ปีกกาชั้นนอก = ใส่ค่า JavaScript, ชั้นใน = object) · ความสูง = ยอด ÷ max × 100% · ใช้ `max` (≥ 1) จึงไม่หารด้วย 0 · CSS `justify-content: flex-end` ทำให้แท่งตั้งจากล่าง
- `L343` ป้ายแกน แสดงตาม `showTick` ไม่งั้นข้อความว่าง (ยังมีช่องว่างไว้ แท่งจึงสูงเท่ากัน)
- `L344–350` ปิด (`L349–350` ปิด Fragment และ ternary ของ L235)

#### 3.12.21 JSX: ตารางสลิปล่าสุด — ปุ่มกรองและกรณีพิเศษ

`frontend/app/page.tsx` · บรรทัด 352–370

```tsx
352          {/* ตารางสลิปล่าสุด + ปุ่มกรองตามผล (ไม่ขึ้นกับช่วงที่เลือก) */}
353          <section className="card">
354            <div className="card-title">
355              <strong>สลิปล่าสุด</strong>
356            </div>
357            <div className="filters">
358              {FILTERS.map((f) => (
359                <Link key={f.key} href={href(mode, date, f.key)} className={f.key === filter ? "chip active" : "chip"}>
360                  {f.label}
361                </Link>
362              ))}
363            </div>
364            {recent === null ? (
365              <p className="muted">{recentResult as string}</p>
366            ) : recent.length === 0 ? (
367              <p className="muted">
368                {filter === "all" ? "ยังไม่มีสลิป ลองส่งรูปสลิปเข้า LINE OA ของร้าน" : "ไม่มีสลิปแบบนี้ในรายการล่าสุด"}
369              </p>
370            ) : (
```

- `L352` คอมเมนต์: ตารางนี้ **ไม่ขึ้นกับช่วงที่เลือก** (แสดงล่าสุดเสมอ) และอยู่นอก ternary ของ L235 จึงแสดงแม้ /summary error
- `L353–356` การ์ด + หัวข้อ
- `L357–363` ปุ่มกรองเป็น `Link` (`?filter=failed`) ไม่ใช่ปุ่ม JavaScript → ทำงานใน Server Component ได้ แชร์ลิงก์ได้ และคง mode/date เดิม · ปุ่มที่เลือกได้ class `chip active`
- `L364–365` `recent === null` (API error) → แสดงข้อความ error
- `L366–369` ไม่มีแถว → ถ้า filter all ชวนลองส่งสลิป ไม่งั้นบอกว่าไม่มีแบบนี้ในรายการล่าสุด
- `L370` `) : (` มีข้อมูล → ตาราง

#### 3.12.22 JSX: ตาราง

`frontend/app/page.tsx` · บรรทัด 371–403

```tsx
371              // ห่อด้วย div ที่เลื่อนซ้ายขวาได้ จอมือถือแคบตารางจะได้ไม่ล้นออกนอกการ์ด
372              <div className="table-wrap">
373                <table className="table">
374                  <thead>
375                    <tr>
376                      <th>เวลา</th>
377                      <th>ธนาคาร</th>
378                      <th className="num">ยอด</th>
379                      <th>ผล</th>
380                      <th>เหตุผล</th>
381                    </tr>
382                  </thead>
383                  <tbody>
384                    {recent.map((r, i) => (
385                      // ใช้ index เป็น key ได้เพราะรายการไม่ถูกเรียงใหม่/ลบบนหน้านี้
386                      <tr key={i}>
387                        <td>{slipTime(r.received_at, today)}</td>
388                        <td>{r.bank ?? "–"}</td>
389                        <td className="num">{r.amount ? `฿${baht(r.amount)}` : "อ่านไม่ได้"}</td>
390                        <td>
391                          <span className={`badge ${VERDICT[r.verdict].tone}`}>
392                            {VERDICT[r.verdict].icon} {VERDICT[r.verdict].text}
393                          </span>
394                          {r.reviewed && <span className="muted small"> (ร้านตรวจแล้ว)</span>}
395                        </td>
396                        <td className="reason">{r.reasons.join(" · ") || "–"}</td>
397                      </tr>
398                    ))}
399                  </tbody>
400                </table>
401              </div>
402            )}
403          </section>
```

- `L371–372` คอมเมนต์ + `table-wrap` (เลื่อนซ้ายขวาได้บนจอแคบ)
- `L373` `<table className="table">`
- `L374–382` หัวตาราง 5 คอลัมน์ · `className="num"` ชิดขวา
- `L383` `<tbody>`
- `L384` วนทุกแถว พร้อม index
- `L385` คอมเมนต์: ใช้ index เป็น key ได้เพราะรายการไม่ถูกเรียงใหม่/ลบบนหน้านี้ (ปกติ React แนะนำ key จากข้อมูลที่ไม่ซ้ำ)
- `L386` `<tr key={i}>`
- `L387` เวลา (`slipTime`)
- `L388` `r.bank ?? "–"` อ่านธนาคารไม่ได้แสดงขีด
- `L389` ``r.amount ? `฿${baht(r.amount)}` : "อ่านไม่ได้" `` (null → "อ่านไม่ได้")
- `L390–395` ป้ายผล (`badge ok/warn/bad`) + `{r.reviewed && <span>(ร้านตรวจแล้ว)</span>}`
- `L396` `r.reasons.join(" · ") || "–"` รวมเหตุผล ถ้า array ว่าง `join` ได้ `""` ซึ่งเป็นเท็จ → ใช้ `"–"` (ใช้ `||` ไม่ใช่ `??` เพราะต้องการให้ข้อความว่างใช้ค่าขวาด้วย)
- `L397–403` ปิดแท็ก

#### 3.12.23 JSX: ท้ายหน้า

`frontend/app/page.tsx` · บรรทัด 404–412

```tsx
404        </main>
405  
406        {/* ท้ายหน้าสีเข้ม */}
407        <footer className="footer">
408          SlipGuard · ข้อมูลจากระบบตรวจสลิปใน LINE OA · อัปเดตเมื่อ {updatedAt} น. (เวลาไทย)
409        </footer>
410      </>
411    );
412  }
```

- `L404` ปิด `<main>`
- `L406–409` ท้ายหน้าสีเข้ม: ที่มาข้อมูล + เวลาอัปเดตล่าสุด (เปลี่ยนทุก 15 วินาทีจาก AutoRefresh ใช้สังเกตได้ว่ารีเฟรชทำงาน)
- `L410–412` ปิด Fragment / return / ฟังก์ชัน

<a id="f-css"></a>
### 3.13 `frontend/app/globals.css` — สไตล์ทั้งเว็บ

> **ภาพรวมไฟล์**
> - **ทำอะไร:** กำหนดสี ฟอนต์ ขนาด และเลย์เอาต์ของทุกส่วนในหน้า โดยใช้ CSS ธรรมดา (ไม่ลง Tailwind/Bootstrap)
> - **ต่อกับ:** ถูก import ใน `layout.tsx` L4 · ชื่อ class ที่ page.tsx ใช้ (`topbar`, `card`, `stats`, `bar`, `heat-3`, `badge ok` ...) ต้องตรงกับ selector ในไฟล์นี้
> - **รูปแบบ CSS:** `selector { property: value; }` · `.ชื่อ` = เลือก element ที่มี class นั้น · `.a .b` = `.b` ที่อยู่ข้างใน `.a` · `.a.b` = element ที่มีทั้งสอง class · `:hover` = ตอนเมาส์ชี้

#### 3.13.1 คอมเมนต์หัวไฟล์ + ตัวแปรสี

`frontend/app/globals.css` · บรรทัด 1–20

```css
 1  /* สไตล์ของหน้าสรุปยอด (แอ๊น)
 2     ผสม 2 แบบ: เลย์เอาต์การ์ดมุมมนแบบเดิม + แถบเมนู/ท้ายหน้าสีเข้ม ปฏิทิน และปุ่มกรองแบบ flat (UI kit)
 3     โทนสี: เขียวมิ้นต์ + เทาเข้ม · ใช้ CSS ธรรมดา ไม่ลงไลบรารีเพิ่ม */
 4  
 5  /* ตัวแปรสีที่ใช้ทั้งเว็บ (อยากเปลี่ยนสีทั้งเว็บ แก้ที่นี่ที่เดียว) */
 6  :root {
 7    --mint: #6fc2ad; /* สีหลัก (เขียวมิ้นต์): เมนูที่เลือก, แท่งกราฟ */
 8    --mint-dark: #4fa792; /* มิ้นต์เข้ม: ตัวหนังสือสีมิ้นต์บนพื้นขาว (อ่านง่ายกว่ามิ้นต์อ่อน) */
 9    --mint-soft: #e3f3ef; /* มิ้นต์อ่อน: พื้นหลังป้าย "ผ่าน" */
10    --dark: #3b4043; /* เทาเข้ม: แถบเมนู, ท้ายหน้า, ปุ่มกรองที่เลือก */
11    --dark-2: #2f3336; /* เทาเข้มกว่า: ตอนเอาเมาส์ชี้เมนู */
12    --amber: #c98a1e; /* รอตรวจ */
13    --amber-soft: #fbf1df;
14    --red: #d0564b; /* ไม่ผ่าน */
15    --red-soft: #fbe7e5;
16    --text: #3b4043; /* ตัวหนังสือหลัก */
17    --muted: #8a9296; /* ตัวหนังสือรอง */
18    --line: #e3e7e8; /* สีเส้นขอบ */
19    --bg: #eef1f2; /* พื้นหลังเทาอ่อน */
20  }
```

- `L1–3` คอมเมนต์ (CSS ใช้ `/* */` เท่านั้น)
- `L5` คอมเมนต์
- `L6` `:root` = element รากของเอกสาร (`<html>`) ตัวแปรที่ประกาศตรงนี้ใช้ได้ทั้งหน้า
- `L7–19` `--ชื่อ: ค่า;` = **CSS custom property** (ตัวแปร) เรียกใช้ด้วย `var(--ชื่อ)` · `#6fc2ad` = สีแบบ hex (แดง-เขียว-น้ำเงิน 2 หลักละ 16 ระดับ) · แบ่งกลุ่ม: มิ้นต์ (หลัก/เข้ม/อ่อน), เทาเข้ม 2 ระดับ, เหลืองอำพัน (รอตรวจ), แดง (ไม่ผ่าน), สีตัวหนังสือ/เส้น/พื้นหลัง · `L8` ใช้มิ้นต์เข้มกับตัวหนังสือบนพื้นขาว เพราะมิ้นต์อ่อนคอนทราสต์ต่ำอ่านยาก · **ถ้าเปลี่ยนค่า `--mint`:** สีเมนู แท่งกราฟ การ์ดยอดรวม เปลี่ยนพร้อมกันทั้งเว็บ
- `L20` ปิด

#### 3.13.2 กติกาพื้นฐาน

`frontend/app/globals.css` · บรรทัด 22–55

```css
22  /* box-sizing: border-box = ความกว้างรวม padding/ขอบแล้ว คำนวณขนาดง่ายกว่า */
23  * {
24    box-sizing: border-box;
25  }
26  
27  body {
28    margin: 0;
29    background: var(--bg);
30    color: var(--text);
31    /* ฟอนต์ที่อ่านภาษาไทยได้ดี มีในเครื่องส่วนใหญ่ ไม่มีก็ใช้ตัวถัดไป */
32    font-family: "Noto Sans Thai", "Sarabun", "Leelawadee UI", "Tahoma", sans-serif;
33    line-height: 1.5;
34    /* เรียงแนวตั้ง + สูงอย่างน้อยเต็มจอ → ท้ายหน้าอยู่ล่างสุดเสมอ แม้เนื้อหาจะสั้น (ตอนยังไม่มีสลิป) */
35    min-height: 100vh;
36    display: flex;
37    flex-direction: column;
38  }
39  
40  /* กว้างสุด 760px อยู่กลางจอ (ดูบนมือถือก็ได้ จอกว้างก็ไม่ยืดจนอ่านยาก) */
41  .page {
42    flex: 1; /* ยืดกินพื้นที่ที่เหลือ ดันท้ายหน้าลงล่าง */
43    width: 100%;
44    max-width: 760px;
45    margin: 0 auto;
46    padding: 16px 16px 32px;
47  }
48  
49  .muted {
50    color: var(--muted);
51  }
52  
53  .small {
54    font-size: 12px;
55  }
```

- `L22–25` `*` = ทุก element · `box-sizing: border-box` = width/height **รวม** padding และเส้นขอบแล้ว (ค่าเริ่มต้นไม่รวม ทำให้กล่องกว้าง 100% + padding ล้นออก)
- `L27` `body`
- `L28` `margin: 0` ลบขอบ 8px ที่เบราว์เซอร์ใส่ให้ แถบเมนูจะชิดขอบจอ
- `L29–30` พื้นหลังและสีตัวหนังสือจากตัวแปร
- `L31–32` `font-family` รายการฟอนต์เรียงลำดับ: เครื่องไหนไม่มีตัวแรกใช้ตัวถัดไป · `sans-serif` = ฟอนต์ไม่มีเชิงของระบบเป็นตัวสุดท้าย
- `L33` `line-height: 1.5` ระยะบรรทัด 1.5 เท่าของขนาดตัวอักษร (ภาษาไทยมีวรรณยุกต์ ต้องการระยะมากกว่าอังกฤษ)
- `L34–37` `min-height: 100vh` สูงอย่างน้อยเต็มจอ (`vh` = 1% ของความสูงจอ) · `display: flex; flex-direction: column` เรียงลูกในแนวตั้ง → ร่วมกับ `.page { flex: 1 }` (L42) ทำให้ **ท้ายหน้าอยู่ล่างสุดเสมอ** แม้เนื้อหาสั้น
- `L40–41` คอมเมนต์ + `.page`
- `L42` `flex: 1` ยืดกินพื้นที่ที่เหลือในแนวตั้ง ดันท้ายหน้าลงล่าง
- `L43–45` `width: 100%; max-width: 760px; margin: 0 auto` = กว้างเต็มจอแต่ไม่เกิน 760px และ **จัดกึ่งกลาง** (`auto` ซ้ายขวาแบ่งที่ว่างเท่ากัน) · จอกว้างก็ไม่ยืดจนอ่านยาก
- `L46` `padding: 16px 16px 32px` = บน ซ้ายขวา ล่าง
- `L49–51` `.muted` ตัวหนังสือรองสีเทา
- `L53–55` `.small` ตัวเล็ก 12px

#### 3.13.3 แถบเมนูด้านบน

`frontend/app/globals.css` · บรรทัด 57–104

```css
 57  /* ---------- แถบเมนูสีเข้มด้านบน ---------- */
 58  .topbar {
 59    background: var(--dark);
 60    color: #fff;
 61  }
 62  
 63  .topbar-inner {
 64    max-width: 760px;
 65    margin: 0 auto;
 66    padding: 0 16px;
 67    display: flex;
 68    justify-content: space-between;
 69    align-items: center;
 70    height: 52px;
 71  }
 72  
 73  .logo {
 74    letter-spacing: 2px; /* ตัวอักษรห่างกันนิดหน่อย ดูเป็นโลโก้ */
 75    font-weight: 300;
 76  }
 77  
 78  .logo b {
 79    color: var(--mint);
 80    font-weight: 700;
 81  }
 82  
 83  .menu {
 84    display: flex;
 85    height: 100%; /* เมนูสูงเต็มแถบ กดง่าย */
 86  }
 87  
 88  .menu-item {
 89    display: flex;
 90    align-items: center;
 91    padding: 0 18px;
 92    text-decoration: none;
 93    color: #d5d9db;
 94    font-size: 14px;
 95  }
 96  
 97  .menu-item:hover {
 98    background: var(--dark-2);
 99  }
100  
101  .menu-item.active {
102    background: var(--mint); /* เมนูที่เลือกอยู่เป็นช่องสีมิ้นต์ */
103    color: #fff;
104  }
```

- `L57` คอมเมนต์หัวส่วน
- `L58–61` `.topbar` พื้นเทาเข้ม ตัวหนังสือขาว (`#fff`)
- `L63–71` `.topbar-inner` กว้างไม่เกิน 760px กึ่งกลาง (ให้ขอบตรงกับเนื้อหาข้างล่าง) · `display: flex` เรียงลูกแนวนอน · `justify-content: space-between` โลโก้ชิดซ้าย เมนูชิดขวา · `align-items: center` กึ่งกลางแนวตั้ง · `height: 52px`
- `L73–76` `.logo` `letter-spacing: 2px` ตัวอักษรห่างกัน · `font-weight: 300` ตัวบาง
- `L78–81` `.logo b` = แท็ก `<b>` ในโลโก้ (คำว่า GUARD) สีมิ้นต์ หนา 700
- `L83–86` `.menu` flex แนวนอน · `height: 100%` สูงเต็มแถบ พื้นที่กดใหญ่
- `L88–95` `.menu-item` flex + `align-items: center` ให้ข้อความกึ่งกลางแนวตั้ง · `padding: 0 18px` · `text-decoration: none` เอาขีดเส้นใต้ลิงก์ออก · สีเทาอ่อน · 14px
- `L97–99` `:hover` ชี้เมาส์แล้วพื้นเข้มขึ้น
- `L101–104` `.menu-item.active` เมนูที่เลือกอยู่ (page.tsx L219 ใส่ class นี้) พื้นมิ้นต์ ตัวขาว

#### 3.13.4 หัวหน้าและแถวเลื่อนช่วง

`frontend/app/globals.css` · บรรทัด 106–162

```css
106  /* ---------- หัวหน้า + เลื่อนช่วง ---------- */
107  .header {
108    display: flex;
109    justify-content: space-between;
110    align-items: center;
111    gap: 12px;
112    margin-bottom: 8px;
113  }
114  
115  .header p {
116    margin: 0;
117  }
118  
119  /* ป้าย "อัปเดตอัตโนมัติ" มุมขวา */
120  .live {
121    font-size: 12px;
122    color: var(--mint-dark);
123    white-space: nowrap;
124  }
125  
126  /* แถวเลื่อนช่วง ‹ 19 พ.ย. 2569 › */
127  .period {
128    display: flex;
129    align-items: center;
130    gap: 8px;
131    margin-bottom: 12px;
132  }
133  
134  .period-title {
135    font-weight: 600;
136    min-width: 130px;
137    text-align: center;
138  }
139  
140  .arrow {
141    width: 30px;
142    height: 30px;
143    display: inline-flex;
144    align-items: center;
145    justify-content: center;
146    border-radius: 8px;
147    background: #fff;
148    border: 1px solid var(--line);
149    text-decoration: none;
150    color: var(--text);
151    font-size: 18px;
152  }
153  
154  .arrow.disabled {
155    color: #c4c8cf; /* ปุ่มถัดไปที่กดไม่ได้ (ช่วงอนาคต) */
156  }
157  
158  .back {
159    margin-left: auto;
160    font-size: 13px;
161    color: var(--mint-dark);
162  }
```

- `L106` คอมเมนต์
- `L107–113` `.header` flex แนวนอน ข้อความซ้าย ป้ายรีเฟรชขวา · `gap: 12px` ระยะห่างระหว่างลูก
- `L115–117` `.header p` เอาขอบของย่อหน้าออก
- `L119–124` `.live` ป้ายรีเฟรช 12px สีมิ้นต์เข้ม · `white-space: nowrap` ห้ามขึ้นบรรทัดใหม่
- `L126–132` `.period` แถว ‹ หัวข้อ › flex · `align-items: center`
- `L134–138` `.period-title` หนา 600 · `min-width: 130px` กว้างอย่างน้อย 130px ปุ่ม › จะไม่ขยับเมื่อหัวข้อยาว/สั้นต่างกัน · จัดกลาง
- `L140–152` `.arrow` ปุ่มสี่เหลี่ยม 30×30 · `display: inline-flex` + center ทั้ง 2 แกน = สัญลักษณ์อยู่กลางปุ่ม · `border-radius: 8px` มุมมน · พื้นขาว เส้นขอบ · ไม่มีขีดเส้นใต้
- `L154–156` `.arrow.disabled` ปุ่ม › ที่กดไม่ได้ สีจาง (page.tsx L265)
- `L158–162` `.back` ลิงก์ "กลับไปปัจจุบัน" · `margin-left: auto` ใน flex = ดันไปชิดขวาสุด

#### 3.13.5 การ์ดและการ์ดตัวเลข

`frontend/app/globals.css` · บรรทัด 164–226

```css
164  /* ---------- การ์ด ---------- */
165  /* การ์ดพื้นขาว มุมมน ใช้กับทุกกล่อง */
166  .card {
167    background: #fff;
168    border: 1px solid var(--line);
169    border-radius: 14px;
170    padding: 16px;
171    margin-bottom: 12px;
172  }
173  
174  /* หัวข้อในการ์ด: ชื่อซ้าย ข้อมูลเสริมขวา */
175  .card-title {
176    display: flex;
177    justify-content: space-between;
178    align-items: baseline;
179    margin-bottom: 10px;
180  }
181  
182  /* แถวการ์ดตัวเลข: ยอดรวมกว้าง 2 ส่วน การ์ดสถานะ 1 ส่วนเท่ากัน */
183  .stats {
184    display: grid;
185    grid-template-columns: 2fr 1fr 1fr 1fr;
186    gap: 10px;
187    margin-bottom: 12px;
188  }
189  
190  .stats .card {
191    margin: 0;
192    display: flex;
193    flex-direction: column;
194    gap: 2px;
195  }
196  
197  /* การ์ดยอดรวมเป็นพื้นมิ้นต์ ให้เด่นกว่าการ์ดอื่น */
198  .stats .total {
199    background: var(--mint);
200    border-color: var(--mint);
201    color: #fff;
202  }
203  
204  .stats .total .muted {
205    color: rgba(255, 255, 255, 0.85);
206  }
207  
208  .amount {
209    font-size: 30px;
210    line-height: 1.2;
211  }
212  
213  .count strong {
214    font-size: 24px;
215  }
216  
217  /* สีตัวเลขของแต่ละสถานะ */
218  .count.ok strong {
219    color: var(--mint-dark);
220  }
221  .count.warn strong {
222    color: var(--amber);
223  }
224  .count.bad strong {
225    color: var(--red);
226  }
```

- `L164–165` คอมเมนต์
- `L166–172` `.card` พื้นขาว เส้นขอบ 1px มุมมน 14px ระยะใน 16px ระยะล่าง 12px · ใช้กับทุกกล่อง
- `L174–180` `.card-title` flex ชื่อซ้าย ข้อมูลเสริมขวา · `align-items: baseline` ให้เส้นฐานตัวอักษรตรงกัน (ตัวหนังสือคนละขนาดก็เรียงสวย)
- `L182` คอมเมนต์
- `L183–188` `.stats` `display: grid` · `grid-template-columns: 2fr 1fr 1fr 1fr` = 4 คอลัมน์ `fr` = ส่วนแบ่งของพื้นที่ว่าง (ยอดรวมกว้าง 2 ส่วน การ์ดสถานะ 1 ส่วนเท่ากัน) · `gap: 10px`
- `L190–195` `.stats .card` การ์ดในแถวนี้ไม่ต้องมี margin (ใช้ gap แทน) · flex แนวตั้ง ระยะ 2px
- `L197–202` `.stats .total` การ์ดยอดรวมพื้นมิ้นต์ ตัวขาว ให้เด่นกว่าการ์ดอื่น
- `L204–206` ตัวหนังสือรองในการ์ดมิ้นต์ใช้ขาวโปร่ง 85% (`rgba` = สี + ความทึบ) เพราะเทาจะอ่านไม่ออกบนพื้นมิ้นต์
- `L208–211` `.amount` ตัวเลขยอดใหญ่ 30px · `line-height: 1.2` กระชับ
- `L213–215` `.count strong` ตัวเลขจำนวนใบ 24px
- `L217–226` สีตัวเลขตามสถานะ `.count.ok` มิ้นต์เข้ม · `.warn` อำพัน · `.bad` แดง

#### 3.13.6 กราฟแท่ง

`frontend/app/globals.css` · บรรทัด 228–262

```css
228  /* ---------- กราฟแท่ง (โหมดวัน/ปี) ---------- */
229  .chart {
230    display: flex;
231    align-items: stretch;
232    gap: 3px;
233    height: 170px;
234  }
235  
236  .col {
237    flex: 1; /* ทุกแท่งกว้างเท่ากัน */
238    display: flex;
239    flex-direction: column;
240    justify-content: flex-end; /* แท่งตั้งจากล่างขึ้นบน */
241    min-width: 0;
242  }
243  
244  .bar {
245    background: var(--mint);
246    border-radius: 3px 3px 0 0;
247    min-height: 2px; /* ยอด 0 ก็ยังเห็นเส้นบางๆ รู้ว่ามีช่องนั้น */
248  }
249  
250  .bar:hover {
251    background: var(--mint-dark);
252  }
253  
254  .tick {
255    height: 18px;
256    font-size: 11px;
257    color: var(--muted);
258    text-align: center;
259    white-space: nowrap;
260    border-top: 1px solid var(--line);
261    padding-top: 2px;
262  }
```

- `L228` คอมเมนต์
- `L229–234` `.chart` flex แนวนอน · `align-items: stretch` ทุกคอลัมน์สูงเต็ม 170px · `gap: 3px` ระยะระหว่างแท่ง
- `L236–242` `.col` `flex: 1` **ทุกแท่งกว้างเท่ากัน** · flex แนวตั้ง · `justify-content: flex-end` ดันลูกลงล่าง = **แท่งตั้งจากล่างขึ้นบน** · `min-width: 0` ให้หดได้เล็กกว่าเนื้อหา (ไม่งั้น 24 แท่งบนมือถือล้นจอ)
- `L244–248` `.bar` พื้นมิ้นต์ มุมบนมน (`3px 3px 0 0` = บนซ้าย บนขวา ล่างขวา ล่างซ้าย) · `min-height: 2px` ยอด 0 ก็ยังเห็นเส้นบางๆ · ความสูงจริงมาจาก inline style ใน page.tsx L341
- `L250–252` ชี้แล้วเข้มขึ้น
- `L254–262` `.tick` ป้ายใต้แท่ง สูงคงที่ 18px (แท่งทุกแท่งจึงมีพื้นที่เท่ากัน) 11px สีเทา กลาง ไม่ขึ้นบรรทัด · `border-top` เส้นแกน x

#### 3.13.7 ปฏิทิน

`frontend/app/globals.css` · บรรทัด 264–327

```css
264  /* ---------- ปฏิทิน (โหมดเดือน) ---------- */
265  /* grid 7 คอลัมน์ = 7 วันในสัปดาห์
266     minmax(0, 1fr) = ช่องหดได้จนพอดีจอ (1fr เฉยๆ จะไม่หดเล็กกว่าตัวหนังสือข้างใน แล้วล้นจอมือถือ) */
267  .calendar {
268    display: grid;
269    grid-template-columns: repeat(7, minmax(0, 1fr));
270    gap: 4px;
271  }
272  
273  .cal-weekday {
274    text-align: center;
275    font-size: 12px;
276    color: var(--muted);
277    padding-bottom: 4px;
278  }
279  
280  .cal-day {
281    aspect-ratio: 1 / 0.8; /* ช่องเกือบสี่เหลี่ยมจัตุรัส */
282    padding: 4px 6px;
283    border-radius: 6px;
284    display: flex;
285    flex-direction: column;
286    justify-content: space-between;
287    font-size: 12px;
288    overflow: hidden; /* ตัวเลขยาวเกินช่องให้ตัดทิ้ง ไม่ดันช่องให้กว้าง */
289  }
290  
291  .cal-num {
292    font-weight: 600;
293  }
294  
295  .cal-amount {
296    text-align: right;
297    font-size: 11px;
298  }
299  
300  /* ความเข้มสีตามยอด: heat-0 ไม่มียอด → heat-4 ยอดสูงสุดของเดือน */
301  .heat-0 {
302    background: #f4f6f7;
303    color: var(--muted);
304  }
305  .heat-1 {
306    background: #d7efe9;
307  }
308  .heat-2 {
309    background: #aedccf;
310  }
311  .heat-3 {
312    background: #85cbb8;
313    color: #fff;
314  }
315  .heat-4 {
316    background: var(--mint-dark);
317    color: #fff;
318  }
319  
320  .cal-day.today {
321    outline: 2px solid var(--dark); /* กรอบวันนี้ */
322    outline-offset: -2px;
323  }
324  
325  .cal-day.future {
326    opacity: 0.45; /* วันที่ยังมาไม่ถึง */
327  }
```

- `L264–266` คอมเมนต์อธิบาย `minmax(0, 1fr)`
- `L267–271` `.calendar` grid · `repeat(7, minmax(0, 1fr))` = 7 คอลัมน์เท่ากัน · `minmax(0, 1fr)` ให้ช่องหดได้ถึง 0 (ถ้าใช้ `1fr` เฉยๆ ช่องหดเล็กกว่าเนื้อหาข้างในไม่ได้ ล้นจอมือถือ)
- `L273–278` `.cal-weekday` หัวคอลัมน์ อา–ส
- `L280–289` `.cal-day` `aspect-ratio: 1 / 0.8` สัดส่วนกว้าง:สูง (เกือบจัตุรัส) · flex แนวตั้ง `space-between` เลขวันบน ยอดล่าง · `overflow: hidden` ตัวเลขยาวเกินช่องตัดทิ้ง ไม่ดันช่องกว้าง
- `L291–293` `.cal-num` เลขวันหนา
- `L295–298` `.cal-amount` ยอดชิดขวา 11px
- `L300` คอมเมนต์
- `L301–318` `.heat-0` ถึง `.heat-4` ไล่สีจากเทาอ่อน (ไม่มียอด) ไปมิ้นต์เข้ม (ยอดสูงสุด) · `heat-3/4` ตัวหนังสือขาวเพราะพื้นเข้ม · ชื่อ class มาจาก `heat()` ใน page.tsx L149
- `L320–323` `.cal-day.today` กรอบเทาเข้ม 2px · `outline` ไม่กินพื้นที่เหมือน border · `outline-offset: -2px` วาดกรอบเข้าด้านใน (ไม่ทับช่องข้างๆ)
- `L325–327` `.cal-day.future` `opacity: 0.45` จางลง (วันที่ยังมาไม่ถึง)

#### 3.13.8 ตารางสลิปล่าสุด ป้าย และกล่อง error

`frontend/app/globals.css` · บรรทัด 329–427

```css
329  /* ---------- ตารางสลิปล่าสุด ---------- */
330  /* ปุ่มกรอง ทั้งหมด / ผ่าน / รอตรวจ / ไม่ผ่าน */
331  .filters {
332    display: flex;
333    flex-wrap: wrap;
334    gap: 6px;
335    margin-bottom: 12px;
336  }
337  
338  .chip {
339    padding: 3px 12px;
340    border: 1px solid var(--line);
341    border-radius: 999px; /* มุมมนสุด = ทรงแคปซูล */
342    text-decoration: none;
343    color: var(--text);
344    font-size: 13px;
345  }
346  
347  .chip.active {
348    background: var(--dark);
349    border-color: var(--dark);
350    color: #fff;
351  }
352  
353  /* กล่องนอกเลื่อนซ้ายขวาได้ ถ้าตารางกว้างกว่าจอ */
354  .table-wrap {
355    overflow-x: auto;
356  }
357  
358  .table {
359    width: 100%;
360    min-width: 520px; /* แคบกว่านี้ให้เลื่อนดูแทนการบีบคอลัมน์จนอ่านไม่ออก */
361    border-collapse: collapse; /* เส้นขอบเซลล์ติดกัน ไม่เป็นเส้นคู่ */
362    font-size: 14px;
363  }
364  
365  .table th {
366    text-align: left;
367    font-weight: 600;
368    color: var(--muted);
369    border-bottom: 1px solid var(--line);
370    padding: 6px 8px;
371  }
372  
373  .table td {
374    border-bottom: 1px solid var(--line);
375    padding: 8px;
376    vertical-align: top;
377  }
378  
379  .table .num {
380    text-align: right; /* ตัวเลขชิดขวา เทียบหลักง่าย */
381    white-space: nowrap;
382  }
383  
384  .reason {
385    color: var(--muted);
386    font-size: 13px;
387  }
388  
389  /* ป้ายผลตรวจในตาราง */
390  .badge {
391    display: inline-block;
392    padding: 1px 8px;
393    border-radius: 999px;
394    font-size: 13px;
395    white-space: nowrap;
396  }
397  .badge.ok {
398    background: var(--mint-soft);
399    color: var(--mint-dark);
400  }
401  .badge.warn {
402    background: var(--amber-soft);
403    color: var(--amber);
404  }
405  .badge.bad {
406    background: var(--red-soft);
407    color: var(--red);
408  }
409  
410  /* กล่อง error ตอนอ่านยอดไม่ได้ */
411  .card.error {
412    border-color: #f2c6c1;
413    background: var(--red-soft);
414  }
415  
416  .card.error ul {
417    margin: 8px 0 0;
418    padding-left: 18px;
419    font-size: 14px;
420  }
421  
422  code {
423    background: rgba(0, 0, 0, 0.06);
424    padding: 1px 5px;
425    border-radius: 4px;
426    font-size: 12px;
427  }
```

- `L329–330` คอมเมนต์
- `L331–336` `.filters` flex · `flex-wrap: wrap` ปุ่มเยอะเกินแถวให้ขึ้นแถวใหม่
- `L338–345` `.chip` ปุ่มกรอง · `border-radius: 999px` มุมมนมากกว่าครึ่งความสูง = **ทรงแคปซูล**
- `L347–351` `.chip.active` ปุ่มที่เลือก พื้นเทาเข้ม ตัวขาว
- `L353–356` `.table-wrap` `overflow-x: auto` ตารางกว้างกว่ากล่องให้มีแถบเลื่อนแนวนอน (แทนการล้นการ์ด)
- `L358–363` `.table` กว้างเต็ม · `min-width: 520px` แคบกว่านี้ให้เลื่อนแทนการบีบคอลัมน์จนอ่านไม่ออก · `border-collapse: collapse` เส้นขอบเซลล์ติดกันเป็นเส้นเดียว
- `L365–371` `.table th` หัวตาราง ชิดซ้าย หนา สีเทา เส้นล่าง
- `L373–377` `.table td` เส้นล่างทุกแถว · `vertical-align: top` เนื้อหาชิดบน (แถวที่เหตุผลยาวหลายบรรทัดอ่านง่าย)
- `L379–382` `.table .num` ตัวเลขชิดขวา (เทียบหลักง่าย) ไม่ขึ้นบรรทัด
- `L384–387` `.reason` เหตุผลสีเทา 13px
- `L389` คอมเมนต์
- `L390–396` `.badge` ป้ายผล `display: inline-block` (มี padding ได้แต่ยังอยู่ในบรรทัด) ทรงแคปซูล
- `L397–408` สีป้าย ok/warn/bad = พื้นอ่อน + ตัวหนังสือเข้มของสีเดียวกัน
- `L410–414` `.card.error` กล่อง error เส้นแดงอ่อน พื้นแดงอ่อน
- `L416–420` รายการวิธีแก้ในกล่อง error · `padding-left: 18px` ที่สำหรับจุดหน้ารายการ
- `L422–427` `code` ตัวหนังสือคำสั่ง พื้นเทาโปร่ง (`rgba(0,0,0,0.06)`) มุมมน

#### 3.13.9 ท้ายหน้า

`frontend/app/globals.css` · บรรทัด 429–436

```css
429  /* ---------- ท้ายหน้าสีเข้ม ---------- */
430  .footer {
431    background: var(--dark);
432    color: #aeb4b7;
433    text-align: center;
434    font-size: 12px;
435    padding: 18px 16px;
436  }
```

- `L429` คอมเมนต์
- `L430–436` `.footer` พื้นเทาเข้ม ตัวเทาอ่อน กลาง 12px ระยะใน 18px

#### 3.13.10 จอเล็ก (มือถือ)

`frontend/app/globals.css` · บรรทัด 438–463

```css
438  /* ---------- จอเล็ก (มือถือ) ---------- */
439  @media (max-width: 560px) {
440    .menu-item {
441      padding: 0 10px; /* เมนูชิดกันขึ้น ให้พอดีจอ */
442    }
443    .logo {
444      letter-spacing: 1px;
445    }
446    /* ยอดรวมเต็มแถว การ์ดสถานะ 3 ใบเรียงข้างล่าง */
447    .stats {
448      grid-template-columns: 1fr 1fr 1fr;
449    }
450    .stats .total {
451      grid-column: 1 / -1;
452    }
453    .count > span:first-child {
454      font-size: 13px;
455      white-space: nowrap; /* "⚠️ รอตรวจ" ไม่ขึ้นบรรทัดใหม่ */
456    }
457    .cal-day {
458      padding: 3px;
459    }
460    .cal-amount {
461      font-size: 9px;
462    }
463  }
```

- `L438` คอมเมนต์
- `L439` `@media (max-width: 560px) { ... }` = **media query** กฎข้างในใช้เฉพาะเมื่อจอกว้างไม่เกิน 560px (มือถือ) และทับกฎเดิมเพราะอยู่ท้ายไฟล์
- `L440–442` เมนูชิดกันขึ้น ให้พอดีจอ
- `L443–445` โลโก้ห่างน้อยลง
- `L446` คอมเมนต์
- `L447–449` `.stats` เหลือ 3 คอลัมน์เท่ากัน
- `L450–452` `.stats .total` `grid-column: 1 / -1` = ตั้งแต่เส้นแบ่งคอลัมน์แรกถึงเส้นสุดท้าย (−1 = นับจากท้าย) → การ์ดยอดรวมเต็มแถว การ์ดสถานะ 3 ใบเรียงข้างล่าง
- `L453–456` `.count > span:first-child` (`>` = ลูกโดยตรง, `:first-child` = ลูกตัวแรก) ข้อความ "⚠️ รอตรวจ" เล็กลงและห้ามขึ้นบรรทัดใหม่
- `L457–462` ช่องปฏิทินระยะในน้อยลง ยอดตัวเล็ก 9px
- `L463` ปิด media query

<a id="f-tests"></a>
### 3.14 เทสต์ 3 ไฟล์

> **ทำไมมีเทสต์:** CI ของทีม (หน้าที่ร่วมของแอ๊น) รัน pytest ทุก PR ถ้าใครแก้โค้ดแล้วทำให้กติกาการนับยอดพัง เทสต์จะแดงก่อน merge
> **วิธีรัน** (ต้องอยู่ในโฟลเดอร์ `backend`): `pytest tests/test_reports.py tests/test_reports_db.py tests/test_metrics.py -q`
> **ศัพท์ pytest:** ฟังก์ชันที่ชื่อขึ้นต้น `test_` = 1 เทสต์ · `assert เงื่อนไข` ถ้าเท็จ = เทสต์ไม่ผ่าน · `pytest.raises(X)` = ต้องเกิด error X ข้างใน with · **fixture** = ฟังก์ชันเตรียมของให้เทสต์ (ส่งเข้ามาทางชื่อพารามิเตอร์)

#### 3.14.1 `tests/test_reports.py` — เทสต์การคำนวณและ API (ไม่ใช้ฐานข้อมูล)

`backend/tests/test_reports.py` · บรรทัด 1–15

```python
 1  """เทสต์การคำนวณยอดสรุป (summary.py) + API /api/v1/reports (แอ๊น)
 2  
 3  เทสต์ในไฟล์นี้ไม่ใช้ฐานข้อมูล: ใช้สลิปที่สร้างในโค้ด (SLIPS) หรือแหล่งข้อมูลปลอม
 4  เทสต์ที่อ่านจากตาราง slips จริงอยู่ใน test_reports_db.py
 5  """
 6  
 7  from datetime import date, datetime, timedelta, timezone
 8  from decimal import Decimal
 9  
10  import pytest
11  
12  from app.api.sample_data import sample_slips
13  from app.api.summary import BKK, SlipRecord, build_summary, period_range, summary_message
14  from app.core.contracts import Verdict
15  
```

- `L1–5` docstring: ไฟล์นี้ไม่ใช้ฐานข้อมูล
- `L7–8` วันเวลา + Decimal
- `L10` `pytest`
- `L12–14` import ของที่จะเทสต์

`backend/tests/test_reports.py` · บรรทัด 17–46

```python
17  def slip(amount, verdict, when):
18      """สร้างสลิป 1 ใบแบบย่อ (เลขอ้างอิงสร้างจากยอด+เวลา ไม่ซ้ำกันในชุดเทสต์)"""
19      return SlipRecord(
20          trans_ref=f"T{amount}{when:%H%M}",
21          bank="KBANK",
22          amount=Decimal(amount),
23          verdict=verdict,
24          created_at=when,
25      )
26  
27  
28  # วันอ้างอิงของเทสต์ (ล็อกไว้ ไม่ใช้วันนี้จริง เทสต์จะได้ผลเดิมทุกวัน)
29  DAY = date(2026, 11, 18)
30  
31  # ชุดสลิปที่ใช้ร่วมกัน: วันที่ 18 มี ผ่าน 2 (350 บาท) / รอตรวจ 1 / ไม่ผ่าน 1
32  # ที่เหลือกระจายไปวันอื่นในเดือน/ปีเดียวกัน และปีก่อน
33  SLIPS = [
34      slip(100, Verdict.PASSED, datetime(2026, 11, 18, 9, 15, tzinfo=BKK)),
35      slip(250, Verdict.PASSED, datetime(2026, 11, 18, 9, 40, tzinfo=BKK)),
36      slip(999, Verdict.SUSPICIOUS, datetime(2026, 11, 18, 12, 0, tzinfo=BKK)),
37      slip(500, Verdict.FAILED, datetime(2026, 11, 18, 20, 0, tzinfo=BKK)),
38      slip(70, Verdict.PASSED, datetime(2026, 11, 3, 10, 0, tzinfo=BKK)),
39      slip(30, Verdict.PASSED, datetime(2026, 2, 1, 10, 0, tzinfo=BKK)),
40      slip(1000, Verdict.PASSED, datetime(2025, 12, 31, 23, 0, tzinfo=BKK)),
41  ]
42  
43  
44  def run(mode, ref=DAY, start=None, end=None, slips=SLIPS):
45      """ตัวช่วย: หาช่วงวันแล้วสรุปยอด (ทำแบบเดียวกับที่ API ทำ)"""
46      return build_summary(slips, mode, *period_range(mode, ref, start, end))
```

- `L17–25` `slip(amount, verdict, when)` ตัวช่วยสร้าง SlipRecord สั้นๆ · `f"T{amount}{when:%H%M}"` = เลขอ้างอิงจากยอด+เวลา (`%H%M` = รูปแบบเวลา ชั่วโมงนาที)
- `L28–29` `DAY` วันอ้างอิงล็อกไว้ ไม่ใช้วันนี้จริง → **ผลเทสต์เหมือนเดิมทุกวัน**
- `L31–41` `SLIPS` ชุดข้อมูลกลาง: วันที่ 18 มี ✅100, ✅250, ⚠️999, ❌500 · วันอื่นในเดือน (✅70 วันที่ 3) · ในปี (✅30 ก.พ.) · ปีก่อน (✅1000 31 ธ.ค. 2025) · ออกแบบให้ทุกโหมดได้ยอดต่างกัน ตรวจได้ว่าตัดช่วงถูก
- `L44–46` `run(...)` ทำแบบเดียวกับ API: `period_range` แล้วส่งผลเข้า `build_summary` · `*period_range(...)` = **unpacking** กระจาย tuple (start, end) เป็น 2 อาร์กิวเมนต์

`backend/tests/test_reports.py` · บรรทัด 49–95

```python
49  def test_daily_counts_only_passed_amount():
50      # ยอดนับเฉพาะ ✅ (100 + 250) แต่ status_counts นับทุกสถานะ
51      s = run("daily")
52      assert s["total_amount"] == "350.00"
53      assert s["slip_count"] == 2
54      assert s["status_counts"] == {"passed": 2, "suspicious": 1, "failed": 1}
55  
56  
57  def test_daily_series_by_hour():
58      # รายวันมี 24 ช่อง (ชั่วโมง) สลิป 09:15 กับ 09:40 รวมอยู่ช่อง 09:00
59      s = run("daily")
60      assert len(s["series"]) == 24
61      nine = next(b for b in s["series"] if b["label"] == "09:00")
62      assert nine == {"label": "09:00", "amount": "350.00", "slip_count": 2}
63  
64  
65  def test_monthly_and_yearly():
66      # เดือน พ.ย. = 100 + 250 + 70 · พ.ย. มี 30 วัน → 30 ช่อง
67      m = run("monthly")
68      assert m["total_amount"] == "420.00"
69      assert len(m["series"]) == 30
70      assert (m["start"], m["end"]) == ("2026-11-01", "2026-11-30")
71  
72      # ปี 2026 = 420 + 30 (ก.พ.)
73      y = run("yearly")
74      assert y["total_amount"] == "450.00"  # ไม่รวมสลิปปี 2025
75      assert [b["label"] for b in y["series"]][:2] == ["2026-01", "2026-02"]
76  
77  
78  def test_december_rolls_into_next_year():
79      # ธ.ค. ต้องจบที่ 1 ม.ค. ปีหน้า (กันบั๊ก replace(month=13))
80      assert period_range("monthly", date(2026, 12, 15)) == (date(2026, 12, 1), date(2027, 1, 1))
81  
82  
83  def test_day_is_cut_by_thai_time_not_utc():
84      # 17:30 UTC วันที่ 18 = 00:30 เวลาไทยวันที่ 19
85      late = slip(40, Verdict.PASSED, datetime(2026, 11, 18, 17, 30, tzinfo=timezone.utc))
86      assert run("daily", slips=[late])["total_amount"] == "0.00"
87      assert run("daily", ref=DAY + timedelta(days=1), slips=[late])["total_amount"] == "40.00"
88  
89  
90  def test_custom_range():
91      # 31 ธ.ค. 2025 ถึง 1 ก.พ. 2026 (รวมทั้งสองวัน) = 1000 + 30 · 33 วัน → 33 ช่อง
92      s = run("custom", start=date(2025, 12, 31), end=date(2026, 2, 1))
93      assert s["total_amount"] == "1030.00"
94      assert len(s["series"]) == 33
95  
```

- `L49–54` **ยอดนับเฉพาะ ✅** = 100+250 = `"350.00"` · slip_count 2 · status_counts นับครบ 2/1/1
- `L57–62` รายวันมี 24 ช่อง · `next(b for b in ... if ...)` = หาตัวแรกที่ตรงเงื่อนไข (generator expression) · ช่อง 09:00 ต้องเป็น 350 บาท 2 ใบ (09:15 + 09:40)
- `L65–75` เดือน พ.ย. = 100+250+70 = 420 · 30 ช่อง · start/end ถูก · ปี 2026 = 420+30 = 450 **ไม่รวม 1000 ของปี 2025** · ช่องแรก 2 ช่องเป็น `"2026-01"`, `"2026-02"` (`[:2]` = 2 ตัวแรก)
- `L78–80` ธ.ค. ต้องจบที่ 1 ม.ค. ปีหน้า (กันบั๊ก `replace(month=13)`)
- `L83–87` **ตัดวันตามเวลาไทย:** 17:30 UTC วันที่ 18 = 00:30 ไทยวันที่ 19 → วันที่ 18 ยอด 0, วันที่ 19 ยอด 40
- `L90–94` custom 31 ธ.ค. 2025 – 1 ก.พ. 2026 (รวม 2 วันปลาย) = 1000 + 30 · 33 ช่อง (31 ธ.ค. 1 วัน + ม.ค. 31 วัน + 1 ก.พ. 1 วัน)

`backend/tests/test_reports.py` · บรรทัด 97–129

```python
 97  @pytest.mark.parametrize(
 98      "start,end",
 99      [
100          (None, None),                          # ไม่ส่งช่วงมา
101          (date(2026, 2, 1), date(2026, 1, 1)),  # start หลัง end
102          (date(2025, 1, 1), date(2026, 6, 1)),  # ยาวเกิน 366 วัน
103      ],
104  )
105  def test_custom_range_rejects_bad_input(start, end):
106      with pytest.raises(ValueError):
107          period_range("custom", DAY, start, end)
108  
109  
110  def test_sample_data_is_repeatable_and_has_today():
111      # seed คงที่ → เรียก 2 ครั้งได้ชุดเดิม, วันนี้ต้องมีสลิป, ทุกใบต้องมี timezone
112      a = sample_slips(DAY)
113      assert a == sample_slips(DAY)
114      assert any(s.created_at.date() == DAY for s in a)
115      assert all(s.created_at.tzinfo is not None for s in a)
116  
117  
118  def test_summary_message_for_line():
119      # ข้อความที่แพรวส่งให้แม่ค้าใน LINE: วันที่เป็น พ.ศ. + ยอด + จำนวนแต่ละสถานะ
120      text = summary_message(run("daily"))
121      assert "18/11/2569" in text
122      assert "฿350.00 จาก 2 สลิปที่ผ่าน" in text
123      assert "รอตรวจ 1 ใบ" in text and "ไม่ผ่าน 1 ใบ" in text
124      assert "ยอดเดือน 11/2569" in summary_message(run("monthly"))
125  
126  
127  def test_summary_message_uses_thousands_separator():
128      big = [slip(12500, Verdict.PASSED, datetime(2026, 11, 18, 9, 0, tzinfo=BKK))]
129      assert "฿12,500.00" in summary_message(run("daily", slips=big))
```

- `L97–104` `@pytest.mark.parametrize("start,end", [...])` = รันเทสต์เดียวกัน 3 รอบด้วยข้อมูล 3 ชุด: ไม่ส่ง / start หลัง end / ยาวเกิน 366 วัน
- `L105–107` ทุกชุดต้อง `ValueError`
- `L110–115` sample_data: เรียก 2 ครั้งได้ชุดเดิม (seed คงที่) · `any(...)` มีอย่างน้อย 1 ใบของวันนั้น · `all(...)` ทุกใบมี timezone
- `L118–124` ข้อความ LINE: มี `18/11/2569` (พ.ศ.) · ยอด + จำนวน · จำนวนรอตรวจ/ไม่ผ่าน · หัวข้อรายเดือน `ยอดเดือน 11/2569`
- `L127–129` 12500 → `฿12,500.00` (ลูกน้ำหลักพัน)

`backend/tests/test_reports.py` · บรรทัด 132–181

```python
132  def test_summary_endpoint():
133      """เรียก API จริงผ่าน TestClient แต่สลับแหล่งข้อมูลเป็นของปลอม (ไม่ต้องมีฐานข้อมูล)"""
134      from fastapi.testclient import TestClient
135  
136      from app.api.reports import get_source
137      from app.main import app
138  
139      seen = {}  # จดว่า API ส่ง shop_id อะไรมาให้แหล่งข้อมูล
140  
141      def fake_source(shop_id, start, end):
142          seen["shop_id"] = shop_id
143          return SLIPS
144  
145      # dependency_overrides = บอก FastAPI ว่า "ตอนเรียก get_source ให้ใช้ตัวนี้แทน"
146      app.dependency_overrides[get_source] = lambda: fake_source
147      try:
148          client = TestClient(app)
149          r = client.get(
150              "/api/v1/reports/summary", params={"mode": "monthly", "date": "2026-11-18", "shop_id": "shop-9"}
151          )
152          assert r.status_code == 200
153          body = r.json()
154          assert body["start"] == "2026-11-01"
155          assert body["total_amount"] == "420.00"
156          assert seen["shop_id"] == "shop-9"
157  
158          # custom แต่ไม่ส่ง start/end → 422
159          assert client.get("/api/v1/reports/summary", params={"mode": "custom"}).status_code == 422
160      finally:
161          app.dependency_overrides.clear()  # คืนค่าเดิม กันเทสต์อื่นได้ของปลอมไปด้วย
162  
163  
164  def test_summary_endpoint_reports_database_error():
165      """ฐานข้อมูลพัง (เช่นยังไม่สร้างตาราง) → 503 พร้อมคำสั่งแก้"""
166      from fastapi.testclient import TestClient
167      from sqlalchemy.exc import OperationalError
168  
169      from app.api.reports import get_source
170      from app.main import app
171  
172      def broken_source(shop_id, start, end):
173          raise OperationalError("SELECT", {}, Exception("no such table: slips"))
174  
175      app.dependency_overrides[get_source] = lambda: broken_source
176      try:
177          r = TestClient(app).get("/api/v1/reports/summary")
178          assert r.status_code == 503
179          assert "app.verify.db" in r.json()["detail"]
180      finally:
181          app.dependency_overrides.clear()
```

- `L132–161` **เทสต์ API `/summary` จริงผ่าน `TestClient`** (จำลองการเรียก HTTP โดยไม่ต้องเปิดเซิร์ฟเวอร์)
  - `L134–137` import ภายในฟังก์ชัน (โหลดแอปเฉพาะเทสต์ที่ต้องใช้)
  - `L139` dict ไว้จดว่าแหล่งข้อมูลได้รับ shop_id อะไร
  - `L141–143` แหล่งข้อมูลปลอม คืน SLIPS ทุกใบ (build_summary กรองช่วงเอง — นี่คือเหตุผลของ L123 ใน summary.py)
  - `L146` `app.dependency_overrides[get_source] = lambda: fake_source` บอก FastAPI ว่าเวลาต้องการ `get_source` ให้ใช้ฟังก์ชันนี้แทน · `lambda: x` = ฟังก์ชันไม่มีชื่อที่คืน x
  - `L147–148` `try:` + สร้าง client
  - `L149–151` เรียก `GET /api/v1/reports/summary?mode=monthly&date=2026-11-18&shop_id=shop-9`
  - `L152–156` ต้องได้ 200, start ถูก, ยอด 420, shop_id ส่งถึงแหล่งข้อมูล
  - `L158–159` custom ไม่ส่ง start/end → 422
  - `L160–161` `finally: app.dependency_overrides.clear()` คืนค่าเดิมเสมอ กันเทสต์อื่นได้ของปลอมไปด้วย
- `L164–181` **ฐานข้อมูลพัง → 503**
  - `L172–173` แหล่งข้อมูลที่โยน `OperationalError` (ลูกของ SQLAlchemyError) จำลอง "no such table"
  - `L177–179` ต้องได้ 503 และข้อความมีคำสั่ง `app.verify.db`

`backend/tests/test_reports.py` · บรรทัด 184–210

```python
184  def test_recent_endpoint():
185      """/recent ส่ง shop_id + limit ให้แหล่งข้อมูล แล้วห่อผลด้วย {"items": ...}"""
186      from fastapi.testclient import TestClient
187  
188      from app.api.reports import get_recent_source
189      from app.main import app
190  
191      seen = {}
192      item = {"received_at": "2026-11-18T09:15+07:00", "bank": "KBANK", "amount": "100.00",
193              "verdict": "passed", "fraud_score": 0, "reasons": [], "reviewed": False}
194  
195      def fake_recent(shop_id, limit):
196          seen.update(shop_id=shop_id, limit=limit)
197          return [item]
198  
199      app.dependency_overrides[get_recent_source] = lambda: fake_recent
200      try:
201          client = TestClient(app)
202          r = client.get("/api/v1/reports/recent", params={"limit": 3, "shop_id": "shop-9"})
203          assert r.status_code == 200
204          assert r.json() == {"items": [item]}
205          assert seen == {"shop_id": "shop-9", "limit": 3}
206  
207          # limit เกิน 50 → FastAPI ตอบ 422 ให้เอง
208          assert client.get("/api/v1/reports/recent", params={"limit": 999}).status_code == 422
209      finally:
210          app.dependency_overrides.clear()
```

- `L184–210` **เทสต์ `/recent`**
  - `L191–193` แถวตัวอย่าง 1 แถว
  - `L195–197` แหล่งข้อมูลปลอมจด shop_id + limit (`seen.update(...)` เพิ่มหลาย key พร้อมกัน)
  - `L202–205` ส่ง limit 3 / shop-9 → ได้ `{"items": [item]}` และแหล่งข้อมูลได้รับค่าถูก
  - `L207–208` limit 999 → 422 (จาก `le=50`)
  - `L209–210` คืนค่า override

#### 3.14.2 `tests/test_reports_db.py` — เทสต์กับตาราง slips จริง (SQLite)

`backend/tests/test_reports_db.py` · บรรทัด 1–28

```python
 1  """เทสต์ summarize() / load_recent() / seed_demo ที่อ่านเขียนตาราง slips จริง (แอ๊น)
 2  
 3  ใช้ SQLite ในหน่วยความจำ ("sqlite://") แทน Postgres → ไม่ต้องเปิด docker ก็รันได้ และ CI รันได้
 4  ข้อควรระวัง: SQLite เก็บเวลาแบบไม่มี timezone จึงใส่เวลาเป็น UTC ทุกครั้งในเทสต์นี้
 5  """
 6  
 7  from datetime import date, datetime, timezone
 8  from decimal import Decimal
 9  
10  import pytest
11  from sqlalchemy import create_engine, func, select
12  from sqlalchemy.orm import Session
13  
14  from app.api.seed_demo import DEMO_PREFIX, seed
15  from app.api.store import load_recent, summarize
16  from app.verify.models import Base, Slip
17  
18  UTC = timezone.utc
19  DAY = date(2026, 11, 18)
20  
21  
22  @pytest.fixture
23  def session():
24      """ฐานข้อมูลใหม่เอี่ยมทุกเทสต์ (สร้างตารางตาม models.py ของวี) แล้วทิ้งเมื่อเทสต์จบ"""
25      engine = create_engine("sqlite://")
26      Base.metadata.create_all(engine)
27      with Session(engine) as s:
28          yield s
```

- `L1–5` docstring: ใช้ SQLite ในหน่วยความจำ (`"sqlite://"`) แทน Postgres ไม่ต้องเปิด docker CI รันได้ · ข้อควรระวัง: SQLite เก็บเวลาไม่มี timezone จึงใส่ UTC ทุกครั้ง
- `L7–16` import · `Base` จาก models.py ของวี ใช้สร้างตาราง
- `L18–19` ค่าคงที่ UTC และวันอ้างอิง
- `L22` `@pytest.fixture` = ฟังก์ชันเตรียมของ เทสต์ที่มีพารามิเตอร์ชื่อ `session` จะได้ค่านี้
- `L23–28` สร้างฐานข้อมูลใหม่เอี่ยม (`sqlite://` ไม่มีชื่อไฟล์ = อยู่ในหน่วยความจำ) · `create_all` สร้างตารางตาม models.py · `yield s` ส่ง session ให้เทสต์ หลังเทสต์จบ with ปิด session และฐานข้อมูลหายไป → **ทุกเทสต์เริ่มจากตารางว่าง ไม่กระทบกัน**

`backend/tests/test_reports_db.py` · บรรทัด 31–54

```python
31  _counter = 0  # ใช้สร้าง line_message_id / trans_ref ไม่ซ้ำกัน (ทั้งสองคอลัมน์มี unique)
32  
33  
34  def add(session, *, amount="100.00", verdict="passed", at, shop_id="shop-1", merchant_verdict=None,
35          reasons=None):
36      """เพิ่มสลิป 1 ใบ · at = เวลาแบบ UTC"""
37      global _counter
38      _counter += 1
39      session.add(
40          Slip(
41              shop_id=shop_id,
42              line_user_id="U1",
43              line_message_id=f"M{_counter}",
44              trans_ref=f"REF{_counter}",
45              bank="KBANK",
46              amount=Decimal(amount) if amount is not None else None,
47              verdict=verdict,
48              fraud_score=0,
49              reasons=reasons or [],
50              received_at=at,
51              merchant_verdict=merchant_verdict,
52          )
53      )
54      session.flush()  # ส่งเข้าฐานข้อมูลทันที (ยังไม่ commit) query ถัดไปจะเห็นแถวนี้
```

- `L31` ตัวนับระดับโมดูล สร้าง `line_message_id`/`trans_ref` ไม่ซ้ำ (ทั้งสองคอลัมน์มี unique)
- `L34–35` `add(session, *, amount=..., ...)` · `*` = พารามิเตอร์หลังจากนี้ต้องส่งแบบระบุชื่อ (keyword-only) อ่านเทสต์ง่าย
- `L37–38` `global _counter` บอกว่าจะแก้ตัวแปรระดับโมดูล (ไม่งั้น `+=` จะ error เพราะ Python มองเป็นตัวแปรใหม่ในฟังก์ชัน)
- `L39–53` เพิ่ม 1 แถว · `Decimal(amount) if amount is not None else None` ทดสอบกรณียอด NULL ได้
- `L54` `flush()` ส่งเข้าฐานข้อมูลทันที query ถัดไปเห็นแถวนี้

`backend/tests/test_reports_db.py` · บรรทัด 57–115

```python
 57  def test_day_counts_only_passed_slips_of_this_shop(session):
 58      add(session, amount="100.00", at=datetime(2026, 11, 18, 3, 0, tzinfo=UTC))  # 10:00 ไทย
 59      add(session, amount="250.00", at=datetime(2026, 11, 18, 5, 0, tzinfo=UTC))
 60      add(session, amount="999.00", verdict="suspicious", at=datetime(2026, 11, 18, 6, 0, tzinfo=UTC))
 61      add(session, amount="500.00", shop_id="shop-2", at=datetime(2026, 11, 18, 6, 0, tzinfo=UTC))  # ร้านอื่น
 62  
 63      r = summarize(session, "shop-1", "day", today=DAY)
 64      assert r["total_amount"] == "350.00"
 65      assert r["slip_count"] == 2
 66      assert r["status_counts"] == {"passed": 2, "suspicious": 1, "failed": 0}
 67  
 68  
 69  def test_day_is_cut_by_thai_time(session):
 70      # 17:30 UTC วันที่ 17 = 00:30 ไทยวันที่ 18 → นับเป็นวันที่ 18
 71      add(session, amount="40.00", at=datetime(2026, 11, 17, 17, 30, tzinfo=UTC))
 72      # 16:59 UTC วันที่ 17 = 23:59 ไทยวันที่ 17 → ไม่นับ
 73      add(session, amount="60.00", at=datetime(2026, 11, 17, 16, 59, tzinfo=UTC))
 74  
 75      assert summarize(session, "shop-1", "day", today=DAY)["total_amount"] == "40.00"
 76  
 77  
 78  def test_month_and_year(session):
 79      add(session, amount="100.00", at=datetime(2026, 11, 1, 3, 0, tzinfo=UTC))
 80      add(session, amount="200.00", at=datetime(2026, 11, 30, 3, 0, tzinfo=UTC))
 81      add(session, amount="400.00", at=datetime(2026, 2, 1, 3, 0, tzinfo=UTC))
 82      add(session, amount="800.00", at=datetime(2025, 12, 31, 3, 0, tzinfo=UTC))  # ปีก่อน ไม่นับ
 83  
 84      assert summarize(session, "shop-1", "month", today=DAY)["total_amount"] == "300.00"
 85      assert summarize(session, "shop-1", "year", today=DAY)["total_amount"] == "700.00"
 86  
 87  
 88  def test_merchant_review_overrides_system_verdict(session):
 89      # ระบบให้ ⚠️ แต่แม่ค้าเช็กแอปธนาคารแล้วกดว่าผ่าน → นับยอด
 90      add(session, amount="300.00", verdict="suspicious", merchant_verdict="passed",
 91          at=datetime(2026, 11, 18, 3, 0, tzinfo=UTC))
 92      # ระบบให้ผ่าน แต่แม่ค้ากดว่าไม่ผ่าน → ไม่นับยอด
 93      add(session, amount="700.00", verdict="passed", merchant_verdict="failed",
 94          at=datetime(2026, 11, 18, 4, 0, tzinfo=UTC))
 95  
 96      r = summarize(session, "shop-1", "day", today=DAY)
 97      assert r["total_amount"] == "300.00"
 98      assert r["status_counts"] == {"passed": 1, "suspicious": 0, "failed": 1}
 99  
100  
101  def test_passed_slip_without_amount_counts_as_zero(session):
102      # OCR อ่านยอดไม่ได้ (NULL) แต่ผ่าน → นับเป็น 1 ใบ ยอด 0
103      add(session, amount=None, at=datetime(2026, 11, 18, 3, 0, tzinfo=UTC))
104  
105      r = summarize(session, "shop-1", "day", today=DAY)
106      assert r["slip_count"] == 1
107      assert r["total_amount"] == "0.00"
108  
109  
110  def test_empty_shop_gives_zero(session):
111      # ร้านยังไม่มีสลิปเลย → ยอด 0 แต่กราฟยังมีครบ 24 ช่อง
112      r = summarize(session, "shop-1", "day", today=DAY)
113      assert r["total_amount"] == "0.00"
114      assert r["slip_count"] == 0
115      assert len(r["series"]) == 24
```

- `L57–66` ร้าน shop-1 มี ✅100, ✅250, ⚠️999 และร้าน shop-2 มี 500 → ยอด shop-1 ต้อง 350 **ไม่รวมร้านอื่น** (ทดสอบ `WHERE shop_id`) · คอมเมนต์ L58: 03:00 UTC = 10:00 ไทย
- `L69–75` ขอบเที่ยงคืนไทย: 17:30 UTC วันที่ 17 = 00:30 ไทยวันที่ 18 (นับ) · 16:59 UTC วันที่ 17 = 23:59 ไทยวันที่ 17 (ไม่นับ) → ยอด 40 (ทดสอบ `_utc_midnight`)
- `L78–85` เดือน = 100+200 (วันที่ 1 และ 30) · ปี = 100+200+400 = 700 ไม่รวม 800 ของปีก่อน
- `L88–98` **ผลแม่ค้าชนะ:** ⚠️ ที่แม่ค้ากดผ่าน → นับ 300 · ✅ ที่แม่ค้ากดไม่ผ่าน → ไม่นับ 700 · status_counts ใช้ผลแม่ค้าด้วย
- `L101–107` ยอด NULL แต่ผ่าน → 1 ใบ 0 บาท
- `L110–115` ร้านว่าง → 0 แต่กราฟยังครบ 24 ช่อง

`backend/tests/test_reports_db.py` · บรรทัด 118–141

```python
118  def test_load_recent_newest_first_thai_time_and_shop_only(session):
119      add(session, amount="100.00", at=datetime(2026, 11, 18, 3, 0, tzinfo=UTC))
120      add(session, amount="250.00", verdict="failed", reasons=["สลิปนี้ถูกใช้ไปแล้ว"],
121          at=datetime(2026, 11, 18, 5, 0, tzinfo=UTC))
122      add(session, amount="999.00", shop_id="shop-2", at=datetime(2026, 11, 18, 6, 0, tzinfo=UTC))
123  
124      items = load_recent(session, "shop-1", limit=10)
125      assert [i["amount"] for i in items] == ["250.00", "100.00"]  # ใหม่ก่อน, ไม่มีของร้านอื่น
126      assert items[0]["received_at"] == "2026-11-18T12:00+07:00"   # 05:00 UTC = 12:00 ไทย
127      assert items[0]["verdict"] == "failed"
128      assert items[0]["reasons"] == ["สลิปนี้ถูกใช้ไปแล้ว"]
129      assert load_recent(session, "shop-1", limit=1)[0]["amount"] == "250.00"  # limit ใช้ได้
130  
131  
132  def test_load_recent_uses_merchant_verdict(session):
133      add(session, verdict="suspicious", merchant_verdict="passed", at=datetime(2026, 11, 18, 3, 0, tzinfo=UTC))
134      item = load_recent(session, "shop-1")[0]
135      assert item["verdict"] == "passed"
136      assert item["reviewed"] is True
137  
138  
139  def _count(session):
140      """นับแถวทั้งหมดในตาราง slips"""
141      return session.scalar(select(func.count()).select_from(Slip))
```

- `L118–129` `load_recent`: ใหม่ก่อน (250 แล้ว 100) ไม่มีร้านอื่น · 05:00 UTC → `"2026-11-18T12:00+07:00"` · เหตุผลส่งถึง · limit 1 ได้ใบล่าสุดใบเดียว
- `L132–136` สลิปที่แม่ค้าตรวจ: verdict เป็นผลแม่ค้า และ `reviewed` เป็น True
- `L139–141` ตัวช่วยนับแถว `SELECT count(*) FROM slips` (`session.scalar` คืนค่าเดียว)

`backend/tests/test_reports_db.py` · บรรทัด 144–176

```python
144  def test_seed_demo_is_repeatable_and_keeps_real_slips(session):
145      # สลิปจริง 1 ใบ (line_message_id ไม่ได้ขึ้นต้นด้วย DEMO-) ต้องไม่โดนลบ
146      add(session, at=datetime(2026, 11, 18, 3, 0, tzinfo=UTC))
147  
148      _, added = seed(session, "shop-1", days=10)
149      assert added > 0
150      assert _count(session) == added + 1
151  
152      # รันซ้ำ: ลบของเก่าก่อนใส่ใหม่ → จำนวนเท่าเดิม ไม่ซ้ำ
153      removed, added_again = seed(session, "shop-1", days=10)
154      assert removed == added
155      assert _count(session) == added_again + 1
156  
157      # --clear: เหลือแค่สลิปจริง
158      seed(session, "shop-1", days=10, clear_only=True)
159      assert _count(session) == 1
160      assert not session.scalar(select(Slip).where(Slip.line_message_id.like(f"{DEMO_PREFIX}%")))
161  
162  
163  def test_line_summary_command_reads_real_db(session):
164      """คำสั่ง "ยอดวันนี้" ใน LINE (webhook.summary_text ของแพรว) ต้องเรียก summarize() ของแอ๊นได้จริง
165  
166      กันพังซ้ำแบบเดิม: webhook เคย import summarize จาก app.api.summary (ไฟล์ผิด)
167      → except กลืน error เงียบ → แม่ค้าได้ "ยังดูยอดไม่ได้" ทุกครั้ง
168      เทสต์ของ webhook ใช้ summary_text ปลอม จึงจับไม่ได้ เทสต์นี้ใช้ตาราง slips จริง (SQLite)
169      """
170      from app.line import webhook  # import ในเทสต์ จะได้ไม่โหลด LINE/vision ตอนเทสต์อื่นในไฟล์นี้
171  
172      add(session, amount="350.00", at=datetime.now(UTC))  # สลิปผ่าน 1 ใบ เวลาตอนนี้ (= วันนี้)
173  
174      text = webhook.summary_text(session, "shop-1", "day")
175      assert "ยังดูยอดไม่ได้" not in text  # ไม่ตกไปทาง except
176      assert "฿350.00 จาก 1 สลิปที่ผ่าน" in text
```

- `L144–160` **seed_demo ปลอดภัยและรันซ้ำได้**
  - `L146` ใส่สลิป "จริง" 1 ใบก่อน (message id `M1` ไม่ขึ้นต้น DEMO-)
  - `L148–150` seed 10 วัน → จำนวนแถว = ที่ใส่ + 1 · `_, added =` = ทิ้งค่าแรก
  - `L152–155` seed ซ้ำ → ลบเท่าที่ใส่รอบแรก จำนวนแถวไม่บวม
  - `L157–160` `clear_only=True` → เหลือ 1 แถว (ของจริง) และไม่มีแถว DEMO- เหลือ
- `L163–176` **คำสั่งยอดใน LINE ใช้ได้จริง** (กันบั๊ก import ผิดไฟล์กลับมา ดู 3.9)
  - `L170` import webhook ในเทสต์นี้เท่านั้น (ไม่ให้เทสต์อื่นในไฟล์ต้องโหลด LINE/vision)
  - `L172` สลิปผ่าน 1 ใบเวลาตอนนี้ (= วันนี้) เพราะ `summary_text` ไม่รับ today
  - `L174–176` เรียกฟังก์ชันของแพรวกับตารางจริง ต้องไม่ตกไปทาง except และมีข้อความยอด

#### 3.14.3 `tests/test_metrics.py` — เทสต์ timing และ report

`backend/tests/test_metrics.py` · บรรทัด 1–34

```python
 1  """เทสต์ตัวจับเวลา (timing.py) และตัวสรุป p50/p95 (report.py) สำหรับ A6 (แอ๊น)"""
 2  
 3  import logging
 4  
 5  import pytest
 6  
 7  from app.metrics.report import main, markdown_table, parse, percentile
 8  from app.metrics.timing import logger, timed
 9  
10  
11  def test_timed_records_and_logs():
12      sink = {}
13      # ต่อ handler ชั่วคราวเข้า logger เพื่อดักข้อความที่ log ออกมา
14      records = []
15      handler = logging.Handler()
16      handler.emit = records.append
17      logger.addHandler(handler)
18      try:
19          with timed("L2_qr", sink):
20              sum(range(1000))  # งานสั้นๆ ให้มีเวลาให้จับ
21      finally:
22          logger.removeHandler(handler)
23  
24      assert sink["L2_qr"] >= 0
25      assert any("TIMING layer=L2_qr ms=" in r.getMessage() for r in records)
26  
27  
28  def test_timed_still_records_when_code_raises():
29      # โค้ดใน with พัง → error ยังโยนออกไปเหมือนเดิม แต่เวลาต้องถูกบันทึก
30      sink = {}
31      with pytest.raises(ZeroDivisionError):
32          with timed("L3_ocr", sink):
33              1 / 0
34      assert "L3_ocr" in sink
```

- `L1` docstring
- `L3–8` import · `logger` จาก timing.py เพื่อดักข้อความ
- `L11–25` `timed` เก็บเวลาและ log
  - `L14–17` สร้าง `logging.Handler` แล้วแทน `emit` (เมธอดที่ถูกเรียกเมื่อมีข้อความ) ด้วย `records.append` → ทุกข้อความถูกเก็บลง list · ผูกเข้า logger ชั่วคราว
  - `L18–22` `try/finally` ถอด handler ออกเสมอ
  - `L19–20` ครอบงานสั้นๆ ด้วย `timed("L2_qr", sink)`
  - `L24` sink มีเวลา ≥ 0 · `L25` มีบรรทัดที่มี `TIMING layer=L2_qr ms=` (`r.getMessage()` ได้ข้อความหลังแทน %s)
- `L28–34` โค้ดข้างในโยน `ZeroDivisionError` (`1 / 0`) → error ยังออกมา (`pytest.raises`) และ sink ยังมีเวลา (พิสูจน์ try/finally)

`backend/tests/test_metrics.py` · บรรทัด 37–77

```python
37  def test_parse_ignores_other_lines_and_docker_prefix():
38      lines = [
39          "INFO:     172.18.0.1 - GET /healthz 200 OK",  # บรรทัดอื่นต้องข้าม
40          "TIMING layer=L2_qr ms=12.5",
41          "api-1  | TIMING layer=L2_qr ms=7.5",          # มี prefix ของ docker ก็ยังอ่านได้
42          "TIMING layer=L3_ocr ms=820.0",
43      ]
44      assert parse(lines) == {"L2_qr": [12.5, 7.5], "L3_ocr": [820.0]}
45  
46  
47  def test_parse_reads_total_time_from_line_webhook():
48      # รูปแบบ log เวลารวมใน webhook ของแพรว (logger.info("slip_timing message_id=%s verdict=%s total_ms=%.0f"))
49      lines = [
50          "INFO:     slip_timing message_id=468789577898262530 verdict=passed total_ms=1840",
51          "api-1  | INFO:     slip_timing message_id=1 verdict=failed total_ms=2310",
52          "TIMING layer=L2_qr ms=9.0",
53      ]
54      assert parse(lines) == {"total": [1840.0, 2310.0], "L2_qr": [9.0]}
55  
56  
57  def test_percentile_nearest_rank():
58      values = list(range(1, 21))  # 1..20
59      assert percentile(values, 50) == 10  # ตัวที่ ceil(0.5×20) = 10
60      assert percentile(values, 95) == 19  # ตัวที่ ceil(0.95×20) = 19
61      assert percentile([5.0], 95) == 5.0  # มีค่าเดียว
62      with pytest.raises(ValueError):
63          percentile([], 50)
64  
65  
66  def test_markdown_table_and_main(tmp_path, capsys):
67      table = markdown_table({"L2_qr": [10.0, 20.0], "L1_precheck": [1.0]})
68      lines = table.splitlines()
69      assert lines[0].startswith("| ชั้น |")
70      assert lines[2].startswith("| L1_precheck | 1 |")  # เรียงตามชื่อชั้น
71      assert lines[3] == "| L2_qr | 2 | 10.0 | 20.0 | 20.0 |"
72  
73      # main() อ่านจากไฟล์ได้
74      log = tmp_path / "timing.log"
75      log.write_text("TIMING layer=total ms=900.0\n", encoding="utf-8")
76      main([str(log)])
77      assert "| total | 1 | 900.0 |" in capsys.readouterr().out
```

- `L37–44` parse ข้ามบรรทัดอื่น และอ่านได้แม้มี prefix ของ docker (`api-1  | `) · ผลเรียงตามลำดับที่เจอ
- `L47–54` อ่านบรรทัด `slip_timing ... total_ms=` ของแพรวเป็นชั้น `total` (รูปแบบเดียวกับ webhook.py L197)
- `L57–63` percentile 1..20: p50 = 10, p95 = 19 · ค่าเดียว → ค่านั้น · ว่าง → ValueError
- `L66–77` markdown_table: หัวตารางถูก · เรียงชื่อชั้น (L1_precheck ก่อน L2_qr) · แถว L2_qr = `| L2_qr | 2 | 10.0 | 20.0 | 20.0 |` (n=2: p50 → ceil(1) = ตัวที่ 1 = 10, p95 → ceil(1.9) = ตัวที่ 2 = 20) · `tmp_path` (fixture ของ pytest = โฟลเดอร์ชั่วคราว) เขียนไฟล์ log แล้วเรียก `main([ไฟล์])` · `capsys` (fixture) ดักสิ่งที่พิมพ์ออก stdout

---

<a id="s5"></a>
## 5. Workflow / Logic Flow — ลำดับการทำงาน

โค้ดของแอ๊นถูกเริ่มทำงานได้ 4 ทาง

### 5.1 แม่ค้าเปิดหน้าเว็บ (หรือ AutoRefresh ครบ 15 วินาที)

```
เบราว์เซอร์ GET http://localhost:3000/?mode=monthly&date=2026-10-01&filter=failed
│
▼ Next.js server รัน Home({ searchParams })                       [page.tsx L172]
  1. อ่าน+ตรวจค่า URL: mode / date / filter (ค่าแปลก → ค่าเริ่มต้น)   [L178–184]
  2. Promise.all (พร้อมกัน 2 คำขอ)                                  [L188–191]
     ├─ getJson("/api/v1/reports/summary?mode=monthly&date=2026-10-01")
     │    ▼ FastAPI → reports.summary()                             [reports.py L94]
     │      ref = 2026-10-01 · shop_id = SHOP_ID หรือ "shop-1"
     │      period_range("monthly", ref) → (2026-10-01, 2026-11-01) [summary.py L46]
     │        ค่าผิด → ValueError → 422
     │      source = db_source → Session → load_slips()             [store.py L70]
     │        SELECT ... WHERE shop_id=? AND received_at >= ? AND < ?
     │        แต่ละแถว → to_record() (ผลแม่ค้าก่อน, NULL→0, เวลา UTC)
     │        ฐานข้อมูลพัง → 503
     │      build_summary(slips, ...)                               [summary.py L99]
     │        วนสลิป: นับสถานะทุกใบ → ถ้า passed บวกยอดเข้าช่องของวัน
     │      return dict → JSON
     └─ getJson("/api/v1/reports/recent?limit=50")
          ▼ reports.recent() → load_recent() ORDER BY ใหม่→เก่า LIMIT 50 → recent_item()
          return {"items": [...]}
  3. ผลเป็น string = error → null                                   [L193–194]
  4. กรอง recent ตาม filter แล้วเอา 8 แถว                            [L196]
  5. คำนวณ peak/max, prev/next, canGoNext, leadingBlanks            [L199–206]
  6. คืน JSX: เมนู → (error | เลื่อนช่วง + การ์ด + ปฏิทิน/กราฟ) → ตาราง → ท้ายหน้า
│
▼ เบราว์เซอร์ได้ HTML ที่มีตัวเลขครบ → AutoRefresh (client) เริ่ม setInterval 15 วิ
  ครบเวลา → router.refresh() → วนกลับไปข้อ 1 (ไม่โหลดทั้งหน้า)
```

### 5.2 แม่ค้าพิมพ์ "ยอดวันนี้" ใน LINE

```
LINE → POST /webhooks/line (แพรว) → handle_text()
  PERIOD_COMMANDS["ยอดวันนี้"] = "day"                     [webhook.py L39]
  หาร้าน + ตรวจว่าเป็นแม่ค้าของร้าน                          [webhook.py L211–216]
  summary_text(session, shop.id, "day")                   [webhook.py L135]
    └─ store.summarize(session, shop_id, "day")           [store.py L86]
         PERIOD_TO_MODE["day"] = "daily"
         ref = วันนี้ตามเวลาไทย
         period_range → load_slips → build_summary
    └─ summary.summary_message(dict)                      [summary.py L153]
         "📊 ยอดวันที่ 21/11/2569\n฿350.00 จาก 2 สลิปที่ผ่าน\n⚠️ รอตรวจ 1 ใบ · ❌ ไม่ผ่าน 1 ใบ"
    error ใดๆ → "ยังดูยอดไม่ได้ตอนนี้ค่ะ..."
  reply_message(...) → แม่ค้าเห็นข้อความใน LINE
```

### 5.3 วัดเวลาสำหรับ A6

```
(ออกแบบไว้) แต่ละชั้น:  with timed("L2_qr"): check_qr(...)
   → finally: logger.info("TIMING layer=L2_qr ms=12.4")      [timing.py L57]
webhook ของแพรว (มีจริง): logger.info("slip_timing ... total_ms=1840")  [webhook.py L197]
                          │
docker compose logs api --no-log-prefix | ... python -m app.metrics.report
   main() → อ่าน stdin → parse()                              [report.py L35]
     บรรทัดตรง LINE_RE → result[ชั้น].append(ms)
     บรรทัดตรง TOTAL_RE → result["total"].append(ms)
   markdown_table() → p50/p95/max ต่อชั้น (nearest-rank)       [report.py L61]
   → พิมพ์ตาราง → copy ไป slide A6
```

> ⚠️ ในโค้ดบนเครื่องตอนนี้ **ยังไม่มีชั้นไหนเรียก `timed()`** ตารางจะมีแค่แถว `total` (ดูข้อ 8)

### 5.4 ใส่/ลบข้อมูลตัวอย่าง

```
python -m app.api.seed_demo [--days N] [--clear]
  main(): argparse → get_engine → create_tables (ถ้ายังไม่มี)
  Session:
    seed(): DELETE ... WHERE line_message_id LIKE 'DEMO-%'
            --clear → จบ
            sample_slips(days) → วนสร้าง Slip(line_message_id="DEMO-SAMPLE…", received_at=UTC, ...)
            flush()
    commit()   (ทั้งหมดหรือไม่มีเลย)
  พิมพ์ "ลบ … แถว · ใส่ใหม่ … แถว"
```

### 5.5 Pseudocode ของ logic หลัก (build_summary)

```
function build_summary(slips, mode, start, end):
    labels = ทุกช่องของกราฟตาม mode
    amounts[label] = 0, counts[label] = 0 สำหรับทุก label
    status_counts = {passed: 0, suspicious: 0, failed: 0}
    total = 0; passed = 0
    for slip in slips:
        day = วันที่ของ slip ตามเวลาไทย
        if day ไม่อยู่ใน [start, end): ข้าม
        status_counts[slip.verdict] += 1
        if slip.verdict != passed: ข้าม
        label = ช่องของ slip
        amounts[label] += slip.amount; counts[label] += 1
        total += slip.amount; passed += 1
    return {mode, start, end-1วัน, total (ข้อความ 2 ตำแหน่ง), passed, status_counts, series}
```

---

<a id="s6"></a>
## 6. Important Concepts — แนวคิดที่ใช้ในโค้ด

เฉพาะแนวคิดที่มีอยู่จริงในโค้ด พร้อมตำแหน่ง

### 6.1 Python / Backend

| แนวคิด | อธิบายสั้น | ตัวอย่างในโค้ด |
|---|---|---|
| **Pure function** | input เดิมได้ output เดิม ไม่แตะโลกภายนอก → เทสต์ง่าย | ทั้งไฟล์ `summary.py` |
| **Separation of concerns** | แยกคำนวณ / อ่านฐานข้อมูล / HTTP | summary.py / store.py / reports.py |
| **ช่วงครึ่งเปิด `[start, end)`** | รวมต้น ไม่รวมปลาย ไม่ต้องรู้จำนวนวันในเดือน ช่วงต่อกันไม่ทับ | `period_range`, `load_slips` (`>=` กับ `<`) |
| **Timezone / UTC** | ฐานข้อมูลเก็บเวลาจริงแบบ UTC แสดง/ตัดวันตามเวลาไทย | `BKK`, `_utc_midnight`, `astimezone(BKK)` |
| **Decimal สำหรับเงิน** | float ทศนิยมเพี้ยน Decimal ไม่เพี้ยน · ส่ง JSON เป็นข้อความ | `amount: Decimal`, `f"{total:.2f}"` |
| **dataclass(frozen)** | คลาสเก็บข้อมูลที่แก้ไม่ได้ | `SlipRecord` |
| **Type hints / Literal** | บอกชนิดข้อมูล FastAPI ใช้ตรวจค่าให้ | `Mode`, `Period`, `SlipSource` |
| **ORM (SQLAlchemy 2)** | แถวในตาราง ↔ object · สร้าง SQL จากโค้ด กัน SQL injection | `select(Slip).where(...)`, `delete(Slip)` |
| **Session / flush / commit** | flush = ส่งคำสั่งแต่ยังไม่ยืนยัน · commit = ยืนยัน · คนเรียกคุม transaction | `seed()` flush, `main()` commit |
| **Index** | ฐานข้อมูลหาแถวได้เร็วโดยไม่อ่านทั้งตาราง | `ix_slips_shop_received` ของวี ↔ `load_slips` |
| **Connection pool + cache** | ใช้ engine ตัวเดียวซ้ำ ไม่เปิดสายใหม่ทุก request | `@lru_cache _engine()` |
| **Dependency injection** | ฟังก์ชันรับของที่ต้องใช้จากภายนอก ตอนเทสต์สลับได้ | `Depends(get_source)`, `dependency_overrides` |
| **HTTP status code** | 200 สำเร็จ · 422 ค่าที่ส่งมาผิด · 503 ระบบยังไม่พร้อม | `HTTPException(422/503)` |
| **Exception handling** | จับ error แล้วแปลงเป็นคำตอบที่มีความหมาย · `from e` เก็บต้นเหตุ | reports.py L108–118 |
| **Context manager (`with`) + `yield` + `finally`** | เปิด/ปิดทรัพยากรอัตโนมัติ · ทำ finally เสมอแม้ error | `with Session(...)`, `timed()` |
| **Logging** | logger/handler/formatter/level/propagate | timing.py L28–37 |
| **Regular expression** | รูปแบบค้นหาข้อความ group จับค่า | `LINE_RE`, `TOTAL_RE` |
| **Percentile (nearest-rank)** | p50 ค่ากลาง p95 ตัวที่ 95% เร็วกว่า | `percentile()` |
| **Reproducibility (seed)** | สุ่มแบบกำหนด seed ได้ผลเดิมทุกครั้ง | `random.Random(463)` |
| **Comprehension** | สร้าง list/dict ในบรรทัดเดียว | `bucket_labels`, `build_summary` L114–116 |
| **`if __name__ == "__main__"`** | รันเฉพาะตอนเรียกไฟล์ตรง ไม่รันตอน import | seed_demo.py, report.py |
| **Data minimization** | ส่งออกเฉพาะข้อมูลที่จำเป็น | `recent_item` ไม่ส่ง line_user_id |

### 6.2 Frontend (Next.js / React / TypeScript / CSS)

| แนวคิด | อธิบายสั้น | ตัวอย่างในโค้ด |
|---|---|---|
| **Server Component** | รันใน server · async/fetch ได้ · ส่ง HTML ที่มีข้อมูลแล้ว | `page.tsx` ทั้งไฟล์ |
| **Client Component (`"use client"`)** | รันในเบราว์เซอร์ ใช้ hook/ตัวจับเวลาได้ | `AutoRefresh.tsx` |
| **Props** | ค่าที่ส่งให้ component | `children` (layout), `seconds` (AutoRefresh), `searchParams` (page) |
| **Hooks: `useEffect` + cleanup** | ทำงานหลัง render · คืนฟังก์ชันเก็บกวาด | AutoRefresh L12–17 |
| **State ใน URL (ไม่มี useState)** | โหมด/วันที่/ตัวกรอง อยู่ใน query string ปุ่มเป็นลิงก์ | `href()`, `Link` |
| **Asynchronous: async/await, Promise.all** | รอผลโดยไม่บล็อก · เรียกหลายคำขอพร้อมกัน | `getJson`, page.tsx L188 |
| **Error handling แบบไม่ให้หน้าพัง** | คืนข้อความแทน throw แล้วแสดงกล่อง error | `getJson`, page.tsx L235 |
| **TypeScript types** | union, generic `<T>`, `Record`, `as const`, type narrowing | `Mode`, `Summary`, `getJson<T>` |
| **Optional chaining / nullish** | `?.` ไม่ error ถ้า null · `??` ค่าสำรองเมื่อ null/undefined | `body?.detail ?? ...`, `r.bank ?? "–"` |
| **JSX: map + key, ternary, &&** | วนแสดงรายการ, if-else, แสดงตามเงื่อนไข | เมนู, การ์ด, ตาราง |
| **CSS variables / Flexbox / Grid / media query** | สีที่เดียว · เรียงแนวเดียว · ตาราง 2 มิติ · ปรับตามขนาดจอ | globals.css |

### 6.3 การทดสอบ

| แนวคิด | ตัวอย่าง |
|---|---|
| Unit test ของ pure function | test_reports.py ส่วนแรก |
| Fixture (เตรียม/เก็บกวาด) | `session` (SQLite ใหม่ทุกเทสต์), `tmp_path`, `capsys` |
| Parametrize (เทสต์เดียวหลายชุดข้อมูล) | `test_custom_range_rejects_bad_input` |
| Test double (ของปลอม) + dependency override | `fake_source`, `broken_source` |
| Integration test กับฐานข้อมูลจริง (in-memory) | test_reports_db.py |
| Regression test (กันบั๊กเดิมกลับมา) | `test_december_rolls_into_next_year`, `test_line_summary_command_reads_real_db` |

---

<a id="s7"></a>
## 7. My Contribution — ผลงานของแอ๊น

### 7.1 หลักฐานที่ใช้ยืนยัน (และที่ยืนยันไม่ได้)

| หลักฐาน | ยืนยันอะไร |
|---|---|
| แผนทีม `claude/slipguard-team-plan.md` ตาราง "ใครทำอะไร" | แอ๊นเป็นเจ้าของ `backend/app/api/` และ `frontend/` + หน้าที่ร่วมดูแล CI + A6 + รวม slide |
| handoff `claude/ann-session-handoff.md` | PR #25 (merge 20 พ.ย. 23:11, บีมรีวิว) มี: API สรุปยอด + `/recent`, `summarize()`, `summary_message()`, `timing.py` `timed()` + `report.py`, `seed_demo`, หน้าเว็บ, คอมเมนต์, `.gitignore` · และการแก้ import ใน `webhook.summary_text()` + เทสต์ `test_line_summary_command_reads_real_db` |
| หัวไฟล์ทุกไฟล์ในเอกสารนี้มีคำว่า "(แอ๊น)" | ตรงกับข้อ 2 แถวบน |
| **ยืนยันไม่ได้** | ประวัติ git (ใคร commit บรรทัดไหน) — เซสชันนี้ไม่ได้รัน `git log`/`git blame` บนเครื่องแอ๊น · handoff ระบุว่างานทำร่วมกับ Claude ในเซสชันก่อนๆ ตามกติกาของทีม เจ้าของต้องอธิบายได้ทุกบรรทัด (ซึ่งเป็นจุดประสงค์ของเอกสารนี้) · ไฟล์อื่นใน `frontend/` (`package.json`, `Dockerfile`, `tsconfig.json`) ไม่มีหลักฐานว่าแอ๊นเขียน จึงไม่ได้รวมไว้ |

### 7.2 สรุปส่วนที่แอ๊นพัฒนา และ logic สำคัญ

| ส่วน | รับผิดชอบอะไร | logic สำคัญที่ต้องอธิบายได้ |
|---|---|---|
| `summary.py` | คำนวณยอด + ข้อความ LINE | ช่วงครึ่งเปิด, ยอดนับเฉพาะ passed แต่สถานะนับทุกใบ, ตัดวันตามเวลาไทย, กราฟครบทุกช่อง, Decimal + ส่งเป็นข้อความ, ธันวาคม→มกราคมปีหน้า |
| `store.py` | อ่านตาราง slips + `summarize()` ให้แพรว | WHERE ใน SQL + index, `_utc_midnight`, ผลแม่ค้าชนะผลระบบ, NULL→0, received_at ไม่ใช่เวลาในสลิป, ไม่ส่งข้อมูลส่วนตัว |
| `reports.py` | API `/summary`, `/recent` | dependency injection เพื่อเทสต์, 422 vs 503, engine ตัวเดียว (lru_cache), จำกัด limit 1–50 และ custom ≤ 366 วัน |
| `sample_data.py` + `seed_demo.py` | ข้อมูลทดสอบ | seed คงที่, ลบเฉพาะ DEMO-, flush/commit, รันซ้ำได้ |
| `timing.py` + `report.py` | วัดเวลา A6 | context manager + finally, logger config, regex, nearest-rank percentile |
| `page.tsx` + `AutoRefresh.tsx` + `layout.tsx` + `globals.css` | หน้าเว็บสรุปยอด | Server vs Client Component, Promise.all, state ใน URL, ตรวจค่า URL, กันหารด้วย 0, ปฏิทิน (leadingBlanks + heat), CSS grid/flex/media query |
| เทสต์ 3 ไฟล์ | ยืนยันกติกาทั้งหมดข้างบน + CI | SQLite in-memory, dependency_overrides, regression tests |
| แก้ `webhook.summary_text()` (ไฟล์แพรว) | ให้คำสั่งยอดใน LINE ใช้ได้จริง | import จาก `app.api.store` + เทสต์กันบั๊กกลับมา |

### 7.3 เชื่อมโยงกันอย่างไร (เล่าแบบ 30 วินาที)

> "ส่วนของหนูเริ่มทำงานหลังจากวีบันทึกสลิปลงตาราง `slips` แล้วค่ะ `store.py` อ่านเฉพาะสลิปของร้านในช่วงวันที่ต้องการ โดยตัดวันตามเวลาไทย แล้วส่งให้ `summary.py` รวมยอด ซึ่งนับเงินเฉพาะสลิปที่ผ่าน แต่นับจำนวนทุกสถานะ ผลเดียวกันนี้ใช้ 2 ทาง คือ API ที่หน้าเว็บเรียก กับ `summarize()` ที่ webhook ของแพรวเรียกตอนแม่ค้าพิมพ์ 'ยอดวันนี้' ตัวเลขสองที่จึงตรงกันเสมอ หน้าเว็บเป็น Server Component เรียก API 2 ตัวพร้อมกันแล้วรีเฟรชเองทุก 15 วินาที ส่วน `timed()` กับ `report.py` ใช้เก็บเวลาแต่ละชั้นจาก log มาทำตาราง p50/p95 ใน A6 ทุกกติกามีเทสต์คุมอยู่ใน CI ค่ะ"

---

<a id="s8"></a>
## 8. ข้อสังเกตและจุดที่ยังไม่สามารถยืนยันได้

1. **`timed()` ยังไม่มีใครเรียกใช้** — ค้นทั้ง `backend/app/` บนเครื่องแอ๊นแล้ว ไม่มีไฟล์ไหนเรียก `timed(` (มีแค่ `slip_timing ... total_ms` ของแพรวใน webhook.py L197) → ถ้ารัน report.py ตอนนี้จะได้แค่แถว `total` · ถ้าต้องการเวลารายชั้นใน A6 ต้องให้บีม/ชิชาครอบ L1–L5 ใน `run_pipeline()` และวีครอบ L6/L7 · handoff ระบุว่ามี PR ของวี `feature/verify-timing` ซึ่งอาจเพิ่มแล้วใน main (ยังไม่ได้ตรวจ) → `git pull origin main` แล้วค้นอีกครั้ง · ถ้าไม่ทัน ใช้เวลา p50/p95 ของ L1–L3 จาก `eval_vision.py` ของบีมแทน และเขียนกำกับใน slide
2. **ชื่อชั้น L6 ไม่ตรงกัน** — docstring timing.py เขียน `L6_bank` แต่คอมเมนต์ใน `contracts.py` เขียน `L6_transaction` (handoff: เคยแนะนำวีในรีวิว) ตอนทำตารางให้ดูชื่อที่ออกใน log จริง
3. **ยอดรับขึ้นกับ L6** — `decide()` ของวีไม่ให้ ✅ ถ้า L6 ไม่ยืนยัน ถ้า EasySlip ใช้ไม่ได้ ทุกสลิปเป็น ⚠️ "ยอดรับ" บนเว็บ = 0 ตอน demo (store.py รองรับ `merchant_verdict` แล้ว แต่ยังไม่มีปุ่มให้แม่ค้ากด)
4. **ไม่มีระบบล็อกอิน** — ใครเปิด URL ได้ก็เห็นยอดร้าน (เขียนไว้เองใน reports.py L19) ควรพูดเป็นข้อจำกัดใน A5
5. **ตัวกรองสลิปล่าสุดกรองจาก 50 ใบล่าสุดฝั่งหน้าเว็บ** — ถ้า ❌ ใบล่าสุดเก่ากว่า 50 ใบ จะขึ้น "ไม่มีสลิปแบบนี้ในรายการล่าสุด" ทั้งที่มีในฐานข้อมูล
6. **p95 เมื่อ n น้อย** — n < 20 ค่า p95 = ค่าช้าสุด ต้องใส่ n ทุกแถว
7. **สลิปผ่านแต่อ่านยอดไม่ได้** นับเป็น 0 บาท ยอดบนเว็บอาจต่ำกว่าเงินจริง
8. **เวอร์ชันของโค้ด** — เอกสารนี้อ่านจากไฟล์บนเครื่องแอ๊น ณ 21 พ.ย. ~15:45 ยืนยันไม่ได้ว่าตรงกับ `main` บน GitHub ล่าสุด (ไม่ได้รัน git) ถ้า pull แล้วไฟล์เปลี่ยน เลขบรรทัดอาจเลื่อน
9. **ก่อน demo** — `python -m app.api.seed_demo --clear` และหน้าเว็บต้องเปิดบนเครื่องเดียวกับที่รัน webhook (ฐานข้อมูลเดียวกัน)
