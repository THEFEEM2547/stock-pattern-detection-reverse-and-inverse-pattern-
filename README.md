# Stock Pattern Detection (Reverse & Inverse) — Project Proposal

> A pattern detection model for technical analysis that identifies key trend reversals and inverse patterns in stock/crypto time series.

## 1) Problem & Motivation
- นักเทรด/นักลงทุนต้องการ “สัญญาณกลับตัว” (reversal) และ “แพทเทิร์นผกผัน/ผันกลับ” (inverse patterns) ที่แม่นยำและตีความได้ง่าย
- งานนี้ต้องการโมเดลที่:
  1) ตรวจจับแพทเทิร์นฮาร์โมนิก (เช่น Gartley, Bat, Butterfly, Crab, Shark, Cypher) และ
  2) แพทเทิร์นกลับตัวคลาสสิก/เชิงโครงสร้าง (เช่น Double Top/Bottom, Head & Shoulders, Rectangle Break)
- ใช้ได้กับหุ้น/คริปโต, ทำซ้ำได้ (reproducible), และต่อยอดเป็นสัญญาณเชิงกลยุทธ์ได้

## 2) Objectives
- [O1] สกัดจุด Pivot/Swings และวาด XABCD สำหรับฮาร์โมนิก → ตรวจสอบ Fibonacci Ratios
- [O2] ตรวจจับแพทเทิร์นกลับตัวแบบคลาสสิกโดยอิงกฎรูปทรง + เงื่อนไขยืนยัน (volume/volatility/RSI)
- [O3] สร้าง Scoring/Confidence ต่อแพทเทิร์น → คัดกรองสัญญาณคุณภาพ
- [O4] Backtest สัญญาณกับกลยุทธ์ง่าย ๆ (entry/exit/SL-TP) เทียบ Benchmark
- [O5] สร้างโน้ตบุ๊ก/สคริปต์สำหรับรีรัน, บันทึกผล, และรายงาน

## 3) Scope (In) / (Out)
**In-scope**
- Timeframe: Daily (เริ่มต้น) และรองรับ Intraday ภายหลัง
- Markets: หุ้น/คริปโต (ระบุแหล่งข้อมูลปลั๊กอินได้)
- Detection: Harmonic + Classic Reversal/Inverse patterns
- Backtesting: Vectorized backtest และสถิติเบื้องต้น (win rate, PF, max DD)

**Out-of-scope (ระยะแรก)**
- กลยุทธ์ขั้นสูงหลายพอร์ต, Portfolio optimization
- Execution/Trading API จริง

## 4) Data & Sources
- แหล่งข้อมูลยืดหยุ่น: `yfinance` / CSV ภายใน / อื่น ๆ ตามสิทธิ์การใช้งาน
- ตารางชื่อสินทรัพย์: ใช้ไฟล์ชื่อหุ้นที่มีอยู่ (`stocth_names.xlsx`) เพื่อสร้าง watchlist
- จัดวางดาต้าใน `db/` และ/หรือ `halal_stock/` (สอดคล้องโครงสร้างเดิม)

> โครงสร้างรีโปปัจจุบัน (ณ เวลาจัดทำ proposal): มี `Technical_data.ipynb`, `model.ipynb`, โฟลเดอร์ `db/`, `halal_stock/` และไฟล์ `stocth_names.xlsx`.  
> แหล่งอ้างอิง: ดูหน้ารีโปเดียวกันบน GitHub.  

## 5) Methodology

### 5.1 Swing/Pivot Extraction
- ใช้ ZigZag/peak-valley ด้วยพารามิเตอร์ `pct_threshold` หรือ `std`-based
- Clean pivots (ลบจุดซ้อน/ระยะสั้นเกินไป) → สร้างลำดับ X-A-B-C-D

### 5.2 Harmonic Patterns
- คำนวณอัตราส่วน Fib ระหว่างช่วง (XA, AB, BC, CD)
- ตรวจสอบกฎเฉพาะแพทเทิร์น เช่น:
  - **Gartley**: AB ≈ 0.618 XA, BC ≈ 0.382–0.886 AB, CD ≈ 1.27–1.618 BC, D ใกล้ 0.786 XA
  - **Bat, Butterfly, Crab, Shark, Cypher**: ปรับกฎ Fib ตามนิยาม
- ให้ **score** ต่อแพทเทิร์น: ระยะเบี่ยงเบนจากค่ากลาง Fib + โครงสร้าง/มุมเอียง + ความยาวคลื่น

### 5.3 Classic Reversal / Inverse Patterns
- **Double Top/Bottom**: สองยอด/สองฐานใกล้ระดับเดียว + neckline break
- **Head & Shoulders**: โครง H-L-HH-L-H (หรือกลับด้าน) + neckline break
- วัด **quality features**: symmetry, width, volume change, volatility regime

### 5.4 Signal Confirmation
- Indicator filters (เลือกใช้): RSI divergence, ATR regime, Volume expansion
- Confidence = f(harmonic_score, structure_score, confirmation_signals)

### 5.5 Backtesting
- Entry: ที่จุดยืนยัน (เช่น neckline break หรือ D-completion + confirmation)
- Exit: SL/TP แบบสัดส่วน (เช่น 1R/2R) หรือ trailing ATR
- Metrics: Win rate, Expectancy, Profit Factor, Max Drawdown, Sharpe (เบื้องต้น)

## 6) Deliverables
- `pattern_core.py` : โมดูลตรวจจับแพทเทิร์น + scoring
- `features.py` : ตัวสกัด pivots/ratios/shape-features
- `backtest.py` : ตัวทดสอบกลยุทธ์พื้นฐาน + report
- `notebooks/` : 
  - `Technical_data.ipynb` (E2E data→detect→plot)
  - `model.ipynb` (ทดลอง/เทียบอัลกอฯ)
- `reports/` : HTML/MD สรุปผล backtest ต่อสินทรัพย์/ช่วงเวลา
- `examples/plots/` : รูปประกอบแพทเทิร์น (อัตโนมัติจากโค้ด)


## 7) Evaluation Plan
- **Detection metrics** (บนชุด label เล็ก ๆ ที่เราทำมือ): precision/recall per pattern
- **Strategy metrics**: win rate, PF, max DD เทียบ buy&hold หรือ random entries
- **Latency**: เวลาคำนวนต่อสินทรัพย์/ต่อวันข้อมูล

## 8) Risks & Mitigations
- **ข้อมูลไม่สะอาด / gaps** → data validation + forward-fill แบบควบคุม
- **Overfitting กับ fib tolerance** → cross-validation ช่วงเวลา/สินทรัพย์
- **False positive สูง** → เพิ่ม confirmation และ threshold โดยอิง backtest

## 9) Timeline & Milestones (Target)
- **W1**: Pivot/zigzag + harmonic rules & scoring (M1)
- **W2**: Classic patterns + confirmation filters (M2)
- **W3**: Backtest v1 + รายงานผลเบื้องต้น (M3)
- **W4**: Refactor src/, unit tests, docs + release v0.1 (M4)

## 10) License
- โปรดเลือกใบอนุญาต (เช่น MIT/Apache-2.0) ให้สอดคล้อง dependency/data source

## 11) How to Run (Draft)
```bash
# 1) Install
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 2) Run detection demo
python -m src.pattern_core --symbol SET:BBL --tf 1d --plot --save examples/plots/

# 3) Backtest quick
python -m src.backtest --universe stocth_names.xlsx --tf 1d --report reports/
