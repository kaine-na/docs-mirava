# 📐 PANDUAN LENGKAP RATE LIMITING VEOGEN V3.2

> **🎯 Sistem Terbaru:** Staggered Start + Parallel Execution untuk performa maksimal!

---

## 📚 DAFTAR ISI

1. [Apa itu Rate Limit?](#-apa-itu-rate-limit)
2. [Sistem Baru v3.2: Staggered Start](#-sistem-baru-v32-staggered-start)
3. [2 Mode Rate Limiting](#-2-mode-rate-limiting)
4. [Rekomendasi Formula by Sistem](#-rekomendasi-formula-by-sistem)
5. [Cara Kerja Worker Rotation](#-cara-kerja-worker-rotation)
6. [Simulasi Lengkap 5 Skenario](#-simulasi-lengkap-5-skenario)
7. [Error dan Solusinya](#-error-dan-solusinya)
8. [Perkiraan Waktu](#-perkiraan-waktu)
9. [Panduan Penggunaan Lengkap](#-panduan-penggunaan-lengkap)
10. [Checklist Sebelum Generate](#-checklist-sebelum-generate)
11. [Tips Pro](#-tips-pro)
12. [Troubleshooting Cepat](#-troubleshooting-cepat)

---

## 🚨 APA ITU RATE LIMIT?

**Rate Limit** adalah batasan jumlah request yang bisa dilakukan ke server dalam waktu tertentu.

### **Analogi Sederhana: Restoran Cepat Saji**

Bayangkan restoran McDonald's:
- **Kasir** = API Server (Google/GenAI)
- **Pelanggan** = Worker (program generate video)
- **Rate Limit** = "Maksimal 10 pelanggan per menit per kasir"

### **Skenario Tanpa Rate Limiting:**
```
❌ 20 pelanggan datang BERSAMAAN (burst)
   → Kasir kewalahan
   → Server tolak request
   → Error 429 "Too Many Requests"
   → Generate gagal!
```

### **Skenario Dengan Rate Limiting (v3.2):**
```
✅ Pelanggan 1 → Masuk antrian
   Tunggu 15 detik...
✅ Pelanggan 2 → Masuk antrian
   Tunggu 15 detik...
✅ Pelanggan 3 → Masuk antrian
   Tunggu 15 detik...
✅ Pelanggan 4 → Masuk antrian
   
→ Kasir melayani dengan tenang
→ Tidak ada yang ditolak
→ Semua generate sukses!
```

---

## 🚀 SISTEM BARU V3.2: STAGGERED START

### **Apa itu Staggered Start?**

**Staggered Start** = Workers dimulai secara bertahap dengan delay antar start, tapi setelah start, semua jalan **PARALLEL** (bersamaan).

### **Konsep Utama:**

```
┌─────────────────────────────────────────────────────┐
│  STAGGERED START (Mencegah Burst Request)          │
├─────────────────────────────────────────────────────┤
│                                                      │
│  t=0s:   Worker 1 → START ✅                        │
│          Main: Tunggu 15 detik... 😴                │
│                                                      │
│  t=15s:  Worker 2 → START ✅                        │
│          Main: Tunggu 15 detik... 😴                │
│                                                      │
│  t=30s:  Worker 3 → START ✅                        │
│          Main: Tunggu 15 detik... 😴                │
│                                                      │
│  t=45s:  Worker 4 → START ✅                        │
│          Main: Selesai! 🎉                          │
│                                                      │
├─────────────────────────────────────────────────────┤
│  PARALLEL EXECUTION (Setelah Semua Start)          │
├─────────────────────────────────────────────────────┤
│                                                      │
│  Worker 1: Generating... (parallel) 🟢              │
│  Worker 2: Generating... (parallel) 🟢              │
│  Worker 3: Generating... (parallel) 🟢              │
│  Worker 4: Generating... (parallel) 🟢              │
│                                                      │
│  ✅ Semua workers jalan BERSAMAAN!                  │
│  ✅ TIDAK ada yang tunggu worker lain selesai!      │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### **Perbedaan dengan Sistem Lama:**

| Aspek | Sistem Lama (v3.1) | Sistem Baru (v3.2) |
|-------|-------------------|-------------------|
| **Start Workers** | Bersamaan (burst) ❌ | Bertahap (staggered) ✅ |
| **Blocking** | Ada queue blocking ❌ | Tidak ada blocking ✅ |
| **Execution** | Serial (satu-satu) ❌ | Parallel (bersamaan) ✅ |
| **Performa** | Lambat 🐢 | Cepat 🚀 |
| **Rate Limit** | Sering error 429 ❌ | Jarang error ✅ |

---

## 🎛️ 2 MODE RATE LIMITING

Aplikasi menyediakan **2 mode sederhana** untuk mengatur rate limiting:

### **1️⃣ Formula by Sistem (Recommended) 🤖**

**Mode Otomatis** - Sistem menghitung optimal settings berdasarkan jumlah akun yang terdeteksi.

✅ **Keuntungan:**
- Tidak perlu mikir pengaturan
- Sudah ditest optimal
- Auto-adjust berdasarkan akun
- Recommended untuk pemula

❌ **Kekurangan:**
- Tidak bisa custom
- Fixed formula

**Cara Pakai:**
1. Centang radio button **"Formula by Sistem"**
2. Klik **Start**
3. Sistem auto-detect jumlah akun
4. Sistem auto-set optimal workers & delay
5. Done! ✅

---

### **2️⃣ Setting Your Rate Limiting (Manual) ⚙️**

**Mode Manual** - Kamu atur sendiri max worker per akun dan delay.

✅ **Keuntungan:**
- Full control
- Bisa eksperimen
- Cocok untuk advanced user

❌ **Kekurangan:**
- Perlu paham konsep
- Salah setting bisa rate limit
- Butuh trial & error

**Cara Pakai:**
1. Centang radio button **"Setting Your Rate Limiting"**
2. Atur **Max Worker Per Akun** (default: 4, recommended: 4)
3. Atur **Delay Antar Worker** (default: 15s, recommended: 10-20s)
4. Atur delay lainnya (Generate/Status/Upload/Upscale)
5. Klik **Start**

**Settings Manual:**
```
┌─────────────────────────────────────────────────┐
│ Max Worker Per Akun: [4] (rekomendasi: 4)      │
│ Delay Antar Worker: [15] detik (10-20s)        │
│ Delay After Generate: [3] detik                │
│ Delay After Status: [3] detik                  │
│ Delay After Upload: [3] detik (I2V)            │
│ Delay After Upscale: [3] detik (4K)            │
└─────────────────────────────────────────────────┘

Info:
ℹ️ Terdeteksi: 3 akun
   Total Workers: 12 (4 per akun × 3)
```

---

## 📊 REKOMENDASI FORMULA BY SISTEM

Berikut **formula optimal** yang digunakan sistem otomatis:

### **Tabel Rekomendasi:**

| Jumlah Akun | Max Worker/Akun | Total Workers | Delay Antar Worker | Delay Generate/Status |
|-------------|-----------------|---------------|-------------------|----------------------|
| **1 akun** | 4 | 4 | 15 detik | 3 detik |
| **2 akun** | 4 | 8 | 12 detik | 3 detik |
| **3 akun** | 4 | 12 | 7 detik | 3 detik |
| **4-5 akun** | 4 | 16-20 | 5 detik | 3 detik |
| **6+ akun** | 4 | 24+ | 3 detik | 3 detik |

### **Penjelasan Formula:**

**1️⃣ Single Account (1 Akun):**
```
Max Worker: 4
Delay: 15 detik antar start
Total: 4 workers

Timeline:
t=0s:  Worker 1 START → Generate prompt #1
t=15s: Worker 2 START → Generate prompt #2
t=30s: Worker 3 START → Generate prompt #3
t=45s: Worker 4 START → Generate prompt #4

Kenapa 4 workers?
✅ Aman untuk 1 akun (tidak overload)
✅ Delay 15s mencegah burst request
✅ Performa cukup baik untuk 1 akun
```

**2️⃣ Multiple Accounts (2-3 Akun):**
```
2 Akun:
- Max Worker/Akun: 4
- Total: 8 workers (4×2)
- Delay: 12 detik

3 Akun:
- Max Worker/Akun: 4  
- Total: 12 workers (4×3)
- Delay: 7 detik

Worker Distribution:
W1 → Akun A
W2 → Akun B
W3 → Akun C (jika ada)
W4 → Akun A (rotate)
W5 → Akun B (rotate)
...

Kenapa delay lebih kecil?
✅ Load terbagi ke multiple akun
✅ Rate limit per akun lebih aman
✅ Total throughput lebih tinggi
```

**3️⃣ Enterprise Setup (6+ Akun):**
```
6+ Akun:
- Max Worker/Akun: 4
- Total: 24+ workers
- Delay: 3 detik (fastest!)

Performa:
✅ Maximum throughput
✅ Load distribution optimal
✅ Minimal rate limit risk
✅ Production-ready setup
```

---

## 🔄 CARA KERJA WORKER ROTATION

### **Apa itu Worker Rotation?**

**Worker Rotation** adalah sistem dimana:
1. Worker selesai generate → tunggu cooldown → generate lagi
2. Masing-masing worker punya cooldown sendiri (TIDAK tunggu worker lain)
3. Workers jalan PARALLEL (bersamaan)

### **Timeline Lengkap (4 Workers, 1 Akun):**

```
═══════════════════════════════════════════════════════════════════
                    STAGGERED START PHASE
═══════════════════════════════════════════════════════════════════

t=0s:   [Worker-1] START ✅
        Status: Generating prompt #1...
        Main Thread: Sleep 15 detik...

t=15s:  [Worker-2] START ✅
        Status: Generating prompt #2...
        [Worker-1] masih generating... (parallel)
        Main Thread: Sleep 15 detik...

t=30s:  [Worker-3] START ✅
        Status: Generating prompt #3...
        [Worker-1] masih generating... (parallel)
        [Worker-2] masih generating... (parallel)
        Main Thread: Sleep 15 detik...

t=45s:  [Worker-4] START ✅
        Status: Generating prompt #4...
        [Worker-1] masih generating... (parallel)
        [Worker-2] masih generating... (parallel)
        [Worker-3] masih generating... (parallel)

═══════════════════════════════════════════════════════════════════
                    PARALLEL EXECUTION PHASE
═══════════════════════════════════════════════════════════════════

t=60s:  [Worker-1] SELESAI ✅ (video #1 done!)
        [Worker-1] Action: Tunggu cooldown 15s...
        [Worker-2] masih generating...
        [Worker-3] masih generating...
        [Worker-4] masih generating...

t=70s:  [Worker-2] SELESAI ✅ (video #2 done!)
        [Worker-2] Action: Tunggu cooldown 15s...
        [Worker-1] cooldown... (5s remaining)
        [Worker-3] masih generating...
        [Worker-4] masih generating...

t=75s:  [Worker-1] RESTART! ✅
        [Worker-1] Status: Generating prompt #5...
        [Worker-2] cooldown... (10s remaining)
        [Worker-3] masih generating...
        [Worker-4] masih generating...

t=80s:  [Worker-3] SELESAI ✅ (video #3 done!)
        [Worker-3] Action: Tunggu cooldown 15s...
        [Worker-1] generating prompt #5...
        [Worker-2] cooldown... (5s remaining)
        [Worker-4] masih generating...

t=85s:  [Worker-2] RESTART! ✅
        [Worker-2] Status: Generating prompt #6...
        [Worker-1] masih generating #5...
        [Worker-3] cooldown... (10s remaining)
        [Worker-4] masih generating...

t=90s:  [Worker-4] SELESAI ✅ (video #4 done!)
        [Worker-4] Action: Tunggu cooldown 15s...
        [Worker-1] masih generating #5...
        [Worker-2] masih generating #6...
        [Worker-3] cooldown... (5s remaining)

t=95s:  [Worker-3] RESTART! ✅
        [Worker-3] Status: Generating prompt #7...
        [Worker-1] masih generating #5...
        [Worker-2] masih generating #6...
        [Worker-4] cooldown... (10s remaining)

═══════════════════════════════════════════════════════════════════
                    CONTINUOUS ROTATION
═══════════════════════════════════════════════════════════════════

Dan seterusnya...
Semua workers continue rotate dengan cooldown masing-masing!

KEY POINTS:
✅ Workers TIDAK tunggu satu sama lain
✅ Masing-masing punya cooldown sendiri
✅ Execution 100% PARALLEL
✅ Maximum throughput!
```

### **Visualisasi Grafik:**

```
Time →
0s    15s   30s   45s   60s   75s   90s   105s  120s

W1: |START|=====GENERATE=====|COOL|START|====GEN====|
W2:        |START|=====GENERATE=====|COOL|START|====
W3:               |START|=====GENERATE=====|COOL|STA
W4:                      |START|=====GENERATE=====|CO

Legend:
START = Worker dimulai (staggered)
GEN   = Generating (parallel)
COOL  = Cooldown delay
```

---

## 🎬 SIMULASI LENGKAP 5 SKENARIO

### **Skenario 1: Single Account - SALAH ❌**

**Setup:**
```
Akun: 1
Workers: 12
Delay: 3 detik
Mode: Manual (salah setting!)
```

**Yang Terjadi:**
```
t=0s:   12 workers START bersamaan (BURST!)
t=0s:   W1-W12 request API bersamaan
t=1s:   Server Google: "Too many requests!"
        Error 429 Rate Limit Exceeded
        Generate GAGAL SEMUA! ❌
```

**Kenapa Gagal?**
- ❌ Terlalu banyak workers untuk 1 akun
- ❌ Delay terlalu kecil (burst request)
- ❌ Server langsung reject

**Solusi:**
```
✅ Gunakan "Formula by Sistem"
✅ Atau manual: 4 workers, delay 15s
```

---

### **Skenario 2: Single Account - BENAR ✅**

**Setup:**
```
Akun: 1
Workers: 4 (auto dari formula)
Delay: 15 detik
Mode: Formula by Sistem ✅
```

**Timeline:**
```
[12:00:00] ℹ️ Starting generation (ultra - t2v)
[12:00:00] ℹ️ Prompts: 100, Workers: 4
[12:00:00] ℹ️ Formula Sistem: 1 Akun → 4 Workers, Delay 15s
[12:00:00] ✅ Generation started successfully!

[12:00:00] ℹ️ Worker 1 started, waiting 15s...
[12:00:00] ℹ️ [Worker-1] Starting generation (seed: 7600)
[12:00:07] ℹ️ [Worker-1] Waiting 3s after generate...
[12:00:10] ℹ️ [Worker-1] Checking status... (attempt 1/30)

[12:00:15] ℹ️ Worker 2 started, waiting 15s...
[12:00:15] ℹ️ [Worker-2] Starting generation (seed: 5432)

[12:00:30] ℹ️ Worker 3 started, waiting 15s...
[12:00:30] ℹ️ [Worker-3] Starting generation (seed: 8901)

[12:00:45] ℹ️ Worker 4 started
[12:00:45] ℹ️ [Worker-4] Starting generation (seed: 2345)

[12:02:30] ✅ [Worker-1] Video saved: prompt_001.mp4
[12:02:30] ℹ️ [Worker-1] Cooldown 15s before restart...

[12:02:45] ℹ️ [Worker-1] Starting generation (seed: 6789)

... (continue rotation)

[12:45:30] ✅ Generation completed!
           Total: 100, Success: 100, Failed: 0
           Duration: 45 menit 30 detik
```

**Kenapa Sukses?**
- ✅ 4 workers optimal untuk 1 akun
- ✅ Delay 15s mencegah burst
- ✅ Parallel execution after staggered start
- ✅ Tidak ada rate limit error!

**Performa:**
```
100 prompts ÷ 4 workers = 25 prompts per worker
Avg time per video: ~6-8 menit
Total time: ~45 menit

Throughput: ~2.2 video per menit
```

---

### **Skenario 3: Multiple Accounts - POWER MODE 🚀**

**Setup:**
```
Akun: 3
Workers: 12 (auto dari formula)
Delay: 7 detik
Mode: Formula by Sistem ✅
```

**Account Distribution:**
```
Worker 1  → Akun A (akun1@example.com)
Worker 2  → Akun B (akun2@example.com)
Worker 3  → Akun C (akun3@example.com)
Worker 4  → Akun A (rotate)
Worker 5  → Akun B (rotate)
Worker 6  → Akun C (rotate)
Worker 7  → Akun A (rotate)
Worker 8  → Akun B (rotate)
Worker 9  → Akun C (rotate)
Worker 10 → Akun A (rotate)
Worker 11 → Akun B (rotate)
Worker 12 → Akun C (rotate)

Result: 4 workers per akun ✅
```

**Timeline:**
```
[12:00:00] ℹ️ Starting generation (ultra - t2v)
[12:00:00] ℹ️ Prompts: 100, Workers: 12
[12:00:00] ℹ️ Formula Sistem: 3 Akun → 4 Workers per akun, Delay 7s
[12:00:00] ℹ️ Loaded 3 Google Ultra account(s)
[12:00:00] ✅ Generation started successfully!

[12:00:00] ℹ️ Worker 1 started (Akun A), waiting 7s...
[12:00:07] ℹ️ Worker 2 started (Akun B), waiting 7s...
[12:00:14] ℹ️ Worker 3 started (Akun C), waiting 7s...
[12:00:21] ℹ️ Worker 4 started (Akun A), waiting 7s...
[12:00:28] ℹ️ Worker 5 started (Akun B), waiting 7s...
[12:00:35] ℹ️ Worker 6 started (Akun C), waiting 7s...
[12:00:42] ℹ️ Worker 7 started (Akun A), waiting 7s...
[12:00:49] ℹ️ Worker 8 started (Akun B), waiting 7s...
[12:00:56] ℹ️ Worker 9 started (Akun C), waiting 7s...
[12:01:03] ℹ️ Worker 10 started (Akun A), waiting 7s...
[12:01:10] ℹ️ Worker 11 started (Akun B), waiting 7s...
[12:01:17] ℹ️ Worker 12 started (Akun C)

[12:01:17] ✅ All 12 workers running in PARALLEL!

... (workers generating in parallel)

[12:16:30] ✅ Generation completed!
           Total: 100, Success: 100, Failed: 0
           Duration: 16 menit 30 detik
```

**Kenapa Super Cepat?**
- ✅ 12 workers PARALLEL (3× faster than 1 akun)
- ✅ Load terbagi ke 3 akun (4 workers per akun aman)
- ✅ Delay lebih kecil (7s) karena multiple akun
- ✅ Maximum throughput!

**Performa:**
```
100 prompts ÷ 12 workers = 8-9 prompts per worker
Avg time per video: ~6-8 menit
Total time: ~16 menit (3× faster!)

Throughput: ~6 video per menit 🚀
```

---

### **Skenario 4: Enterprise Setup (10 Akun) 💎**

**Setup:**
```
Akun: 10
Workers: 40 (manual: 4 per akun)
Delay: 3 detik (fastest!)
Mode: Manual (advanced user)
```

**Account Distribution:**
```
40 workers ÷ 10 akun = 4 workers per akun ✅

Load per akun: SANGAT RINGAN
Rate limit risk: MINIMAL
```

**Timeline:**
```
[12:00:00] ℹ️ Starting generation (ultra - t2v)
[12:00:00] ℹ️ Prompts: 1000, Workers: 40
[12:00:00] ℹ️ Manual mode: 10 Akun → 4 Workers per akun, Delay 3s
[12:00:00] ✅ Generation started successfully!

[12:00:00-12:02:00] Starting 40 workers dengan delay 3s antar start
[12:02:00] ✅ All 40 workers running in PARALLEL!

... (massive parallel processing)

[12:14:30] ✅ Generation completed!
           Total: 1000, Success: 1000, Failed: 0
           Duration: 14 menit 30 detik
```

**Performa INSANE:**
```
1000 prompts ÷ 40 workers = 25 prompts per worker
Avg time per video: ~6-8 menit
Total time: ~14 menit

Throughput: ~69 video per menit! 💎
```

**Use Case:**
- ✅ Production content agency
- ✅ Mass video generation
- ✅ Enterprise deployment
- ✅ Time-critical projects

---

### **Skenario 5: DISASTER - Jangan Ditiru! ☠️**

**Setup (SALAH TOTAL):**
```
Akun: 1
Workers: 35 (max system)
Delay: 0 detik (no delay!)
Mode: Manual (salah parah!)
```

**Yang Terjadi:**
```
[12:00:00] ℹ️ Starting generation (ultra - t2v)
[12:00:00] ℹ️ Prompts: 100, Workers: 35
[12:00:00] ❌ BURST REQUEST 35 WORKERS BERSAMAAN!

[12:00:01] ❌ [Worker-1] Error 429 Rate Limit Exceeded
[12:00:01] ❌ [Worker-2] Error 429 Rate Limit Exceeded
[12:00:01] ❌ [Worker-3] Error 429 Rate Limit Exceeded
... (30 more errors)

[12:00:05] ❌ [Worker-4] Error 429 Rate Limit Exceeded
[12:00:05] ❌ [Worker-5] Error 429 Rate Limit Exceeded

[12:00:10] ⚠️ Semua workers kena rate limit!
[12:00:10] ⚠️ Retry 3×... GAGAL SEMUA!

[12:05:00] ❌ Generation completed!
           Total: 100, Success: 0, Failed: 100
           Duration: 5 menit
           
           TOTAL DISASTER! ☠️
```

**Kenapa Gagal Total?**
- ❌ 35 workers untuk 1 akun = OVERLOAD PARAH
- ❌ Delay 0 detik = BURST REQUEST (semua bersamaan)
- ❌ Server langsung block semua request
- ❌ Tidak ada yang berhasil!

**Lesson Learned:**
```
🚫 JANGAN PERNAH:
   - Set workers > 6 untuk 1 akun
   - Set delay < 5 detik untuk single akun
   - Disable rate limiting
   - Ignore warning messages
```

---

## ⚠️ ERROR DAN SOLUSINYA

### **1. Error 429: Rate Limit Exceeded**

**Tampilan Error:**
```
❌ [Worker-3] Error 429 Rate Limit Exceeded
❌ Too many requests from this account
❌ Please wait and try again
```

**Arti:**
Anda mengirim terlalu banyak request dalam waktu singkat. Server menolak request untuk melindungi sistemnya.

**Penyebab:**
- Workers terlalu banyak untuk jumlah akun
- Delay antar worker terlalu kecil
- Burst request (semua worker start bersamaan)

**Solusi:**

**A. Segera (Emergency):**
```
1. Klik tombol STOP ⏹️
2. Tunggu 2-3 menit (cooldown server)
3. Aktifkan "Formula by Sistem"
4. Klik START lagi
```

**B. Setting Manual:**
```
Jika punya:
- 1 akun   → Max 4 workers, delay 15s
- 2 akun   → Max 8 workers, delay 12s
- 3 akun   → Max 12 workers, delay 7s
- 4+ akun  → Max 20 workers, delay 5s
```

**C. Tambah Akun (Best Solution):**
```
Rate limit = per akun

1 akun:
- Max safe: 4-6 workers
- Throughput: ~2-3 video/menit

3 akun:
- Max safe: 12-18 workers
- Throughput: ~6-9 video/menit (3× faster!)

💡 Tambah akun = Throughput naik linear!
```

---

### **2. Error 401: Unauthorized**

**Tampilan Error:**
```
❌ [Worker-1] Error 401 Unauthorized
❌ Cookie expired or invalid
❌ Please refresh account
```

**Arti:**
Cookie akun Anda sudah expired (kadaluarsa). Perlu login ulang.

**Penyebab:**
- Cookie expired (biasanya 7-30 hari)
- Session timeout
- Google/GenAI logout otomatis

**Solusi:**
```
1. Pergi ke tab "Accounts"
2. Cari akun yang error
3. Klik tombol 🔄 Refresh Cookie
4. Chrome akan terbuka otomatis
5. Login ulang (jika perlu)
6. Cookie akan update otomatis
7. Kembali ke tab generation
8. Klik START lagi
```

**Pencegahan:**
```
✅ Refresh cookie setiap 2-3 minggu
✅ Jangan logout dari browser
✅ Gunakan "Remember Me" saat login
```

---

### **3. Timeout: Generation Timeout**

**Tampilan Error:**
```
⏱️ [Worker-2] Generation timeout (exceeded 15 minutes)
❌ Request took too long
❌ Moving to next prompt
```

**Arti:**
Generate video memakan waktu terlalu lama (> 15 menit). Kemungkinan server overload atau prompt terlalu complex.

**Penyebab:**
- Server Google/GenAI sedang lambat
- Prompt terlalu kompleks (heavy rendering)
- Peak hours (jam sibuk)

**Solusi:**
```
✅ Wait & Retry: Biasanya prompt selanjutnya berhasil
✅ Simplify Prompt: Kurangi detail yang terlalu complex
✅ Off-Peak Hours: Generate di malam hari (sepi server)
✅ Check Failed Prompts: Review prompts_failed.txt
```

**Tidak Perlu Panik:**
- ✅ Worker otomatis lanjut ke prompt berikutnya
- ✅ Timeout prompt disave di "failed" folder
- ✅ Bisa retry manual nanti

---

### **4. Connection Error**

**Tampilan Error:**
```
❌ [Worker-4] Connection error
❌ Failed to connect to server
❌ Check your internet connection
```

**Arti:**
Tidak bisa connect ke server Google/GenAI. Problem network/internet.

**Penyebab:**
- Internet mati/putus
- DNS problem
- Firewall blocking
- VPN issue

**Solusi:**
```
1. Check Internet:
   - Buka browser → Google.com
   - Pastikan internet normal

2. Restart Network:
   - Matikan WiFi → Nyalakan lagi
   - Atau restart modem

3. Check Firewall:
   - Windows Defender → Allow app
   - Antivirus → Add exception

4. Retry:
   - Klik STOP
   - Tunggu 10 detik
   - Klik START lagi
```

---

### **5. Queue Timeout (v3.1 Only - Tidak Ada di v3.2)**

**Update:**
```
✅ Error ini TIDAK MUNGKIN terjadi di v3.2!
✅ Anti-Collision Queue sudah DISABLED
✅ No more queue blocking!

Jika Anda masih lihat error ini:
→ Update aplikasi ke v3.2 latest!
```

---

## ⏱️ PERKIRAAN WAKTU

### **Video Generation (T2V/I2V):**

Rata-rata waktu per video: **6-8 menit**
- Generate: 6-7 menit (Google processing)
- Download: 30-60 detik
- Upscale (optional): +2-3 menit
- Mute (optional): +5-10 detik

| Setup | Workers | Time/100 Prompts | Throughput |
|-------|---------|------------------|------------|
| **1 akun** | 4 | ~45 menit | ~2.2 video/min |
| **2 akun** | 8 | ~25 menit | ~4 video/min |
| **3 akun** | 12 | ~16 menit | ~6 video/min |
| **5 akun** | 20 | ~12 menit | ~8.3 video/min |
| **10 akun** | 40 | ~8 menit | ~12.5 video/min |

**Contoh Real:**
```
1 Akun + 4 Workers + 1000 Prompts:
= 1000 ÷ 4 = 250 prompts per worker
= 250 × 7 menit = 1750 menit
= ~29 jam (non-stop)
= ~1.2 hari

3 Akun + 12 Workers + 1000 Prompts:
= 1000 ÷ 12 = 84 prompts per worker
= 84 × 7 menit = 588 menit
= ~9.8 jam (non-stop)
= ~10 jam total (3× faster!)
```

---

### **Image Generation:**

Rata-rata waktu per image: **30-60 detik**
- Generate: 20-40 detik
- Download: 5-10 detik
- Upscale (optional): +15-20 detik

| Setup | Workers | Time/100 Prompts | Throughput |
|-------|---------|------------------|------------|
| **1 akun** | 4 | ~8 menit | ~12.5 img/min |
| **2 akun** | 8 | ~5 menit | ~20 img/min |
| **3 akun** | 12 | ~3.5 menit | ~28 img/min |
| **5 akun** | 20 | ~2.5 menit | ~40 img/min |
| **10 akun** | 40 | ~1.5 menit | ~66 img/min |

**Contoh Real:**
```
1 Akun + 4 Workers + 1000 Images:
= 1000 ÷ 4 = 250 images per worker
= 250 × 45 detik = 11250 detik
= ~187 menit = ~3.1 jam

3 Akun + 12 Workers + 1000 Images:
= 1000 ÷ 12 = 84 images per worker
= 84 × 45 detik = 3780 detik
= ~63 menit = ~1 jam (3× faster!)
```

---

## 📘 PANDUAN PENGGUNAAN LENGKAP

### **🎯 STEP 1: Setup Akun**

**A. Tambah Akun Google Ultra atau GenAI:**
```
1. Buka aplikasi Veogen
2. Klik tab "Accounts"
3. Klik "Add Data" untuk provider yang diinginkan
4. Chrome akan terbuka otomatis
5. Login dengan akun Google/GenAI Anda
6. Cookie akan diambil otomatis
7. Akun akan muncul di daftar

Tips:
✅ Gunakan akun berbeda (jangan sama)
✅ Recommended: 3-5 akun untuk performa optimal
✅ Bisa mix: 2 GenAI + 3 Ultra (total 5)
```

**B. Verifikasi Akun:**
```
Di tab "Accounts", pastikan:
✅ Status: Active (hijau)
✅ Cookie: Valid (tidak expired)
✅ Last Used: Recent

Jika ada yang merah:
❌ Klik 🔄 Refresh Cookie
```

---

### **🎯 STEP 2: Pilih Provider & Mode**

**A. Provider:**
```
GenAI Pro:
- Model: Veo 3.1 (Google AI Studio)
- Speed: Fast
- Quality: Excellent
- Best for: Professional video

Google Ultra:
- Model: Veo 3.1 (Google AI Console)
- Speed: Fast
- Quality: Excellent  
- Best for: Alternative account source
```

**B. Mode:**
```
T2V (Text-to-Video):
- Input: Text prompts
- Output: Video dari teks
- Best for: Storyboarding, concepts

I2V (Image-to-Video):
- Input: Images + prompts (optional)
- Output: Video dari gambar
- Best for: Animation, product demo
```

---

### **🎯 STEP 3: Setup Rate Limiting**

**A. Untuk Pemula (Recommended):**
```
1. Scroll ke section "🤖 Pengaturan Rate Limiting"
2. Pilih radio button: "Formula by Sistem"
3. Done! Sistem akan auto-detect & set optimal
```

**B. Untuk Advanced User:**
```
1. Pilih radio button: "Setting Your Rate Limiting"
2. Settings panel akan muncul
3. Atur:
   - Max Worker Per Akun: 4 (recommended)
   - Delay Antar Worker: 15 detik
   - Delay After Generate: 3 detik
   - Delay After Status: 3 detik
   - Delay After Upload: 3 detik
   - Delay After Upscale: 3 detik
4. Info box akan show:
   "Terdeteksi: X akun | Total Workers: Y"
```

**Settings Explanation:**

| Setting | Fungsi | Recommended |
|---------|--------|-------------|
| **Max Worker Per Akun** | Jumlah workers per akun | 4 |
| **Delay Antar Worker** | Jeda start antar worker (staggered) | 10-20 detik |
| **Delay After Generate** | Jeda setelah request generate | 3 detik |
| **Delay After Status** | Jeda setelah cek status | 3 detik |
| **Delay After Upload** | Jeda setelah upload image (I2V) | 3 detik |
| **Delay After Upscale** | Jeda setelah upscale 4K | 3 detik |

---

### **🎯 STEP 4: Prepare Prompts**

**A. Untuk Batch Video Generation:**
```
1. Buat file TXT dengan prompts (satu prompt per baris)
2. Contoh prompts.txt:
   
   A cinematic shot of a cat walking in rain
   Epic drone view of mountain sunset
   Close-up of coffee being poured
   
3. Di aplikasi, klik "Load Prompts"
4. Pilih file prompts.txt
5. Toast muncul: "Loaded 3 prompts from prompts.txt"
```

**B. Untuk Image-to-Video (I2V):**
```
1. Siapkan folder images:
   
   my_images/
   ├── image_001.jpg
   ├── image_002.jpg
   ├── image_003.png
   └── image_004.jpg

2. Di aplikasi, pilih mode "I2V"
3. Klik "Browse" folder images
4. Pilih folder my_images
5. Images akan load otomatis
6. Info: "Loaded 4 images"

7. Pilih prompt mode:
   - Global: Satu prompt untuk semua images
   - Multi: Beda prompt per image (load TXT)
```

---

### **🎯 STEP 5: Settings Tambahan (Optional)**

**A. Aspect Ratio:**
```
- Landscape (16:9): Default, widescreen
- Portrait (9:16): TikTok, Instagram Reels
- Square (1:1): Instagram Post
```

**B. Video Model:**
```
- Fast: Prioritas normal (recommended)
- Lower Priority: Lebih lambat, hemat quota
```

**C. Auto Features:**
```
✅ Auto-Mute: 
   - Hilangkan audio otomatis
   - Requires FFmpeg installed
   
✅ Auto-Upscale 1080p:
   - Upscale ke 1080p (free)
   - Topaz AI (if enabled)
   
✅ Auto-Upscale 4K:
   - Upscale ke 4K (premium)
   - Requires Topaz API key
   - Disable 1080p otomatis
```

**D. Output Directory:**
```
Default: video_download/

Struktur folder otomatis:
video_download/
├── sukses/
│   ├── prompt_001.mp4
│   ├── prompt_002.mp4
│   └── ...
├── failed/
│   └── (failed videos)
└── TrackingPrompt/
    └── batch_20251126_120000/
        ├── sukses/
        │   └── prompts_success.txt
        ├── gagal/
        │   └── prompts_failed.txt
        └── sisa/
            └── prompts_remaining.txt
```

---

### **🎯 STEP 6: Start Generation!**

**A. Final Check:**
```
✅ Akun: Terdeteksi & valid
✅ Prompts: Loaded (atau images loaded)
✅ Rate Limiting: Configured
✅ Settings: Reviewed
✅ Output Dir: Set
```

**B. Klik START Button:**
```
[12:00:00] ✅ Application started successfully
[12:00:00] ℹ️ Ready to generate videos
[12:00:00] ✅ Rate limiting: Staggered start enabled (v3.2)

[12:00:00] ℹ️ Starting generation (ultra - t2v)
[12:00:00] ℹ️ Prompts: 100, Workers: 12
[12:00:00] ℹ️ Formula Sistem: 3 Akun → 4 Workers per akun
[12:00:00] ✅ Generation started successfully!

[12:00:00] ℹ️ Worker 1 started, waiting 7s...
[12:00:07] ℹ️ Worker 2 started, waiting 7s...
...
```

**C. Monitor Progress:**
```
Tab kanan akan show:
┌─────────────────────────────┐
│ 📊 Statistics              │
├─────────────────────────────┤
│ Active Workers: 12          │
│ Completed: 45/100 (45%)     │
│ Failed: 2                   │
│ Remaining: 55               │
│                             │
│ Progress: [████████░░] 45%  │
└─────────────────────────────┘

Log Activity:
✅ [Worker-1] Video saved: prompt_001.mp4
✅ [Worker-2] Video saved: prompt_002.mp4
⏳ [Worker-3] Checking status... (attempt 3/30)
...
```

**D. Controls:**
```
⏸️ PAUSE: Pause semua workers (temporary)
⏹️ STOP: Stop generation (permanent)
🔄 RESUME: Continue dari pause
```

---

### **🎯 STEP 7: Review Results**

**A. Check Output Folder:**
```
video_download/sukses/
✅ prompt_001.mp4 (sukses)
✅ prompt_002.mp4 (sukses)
✅ prompt_003.mp4 (sukses)
...

video_download/failed/
❌ prompt_023.mp4 (jika ada yang gagal)
```

**B. Check Tracking:**
```
TrackingPrompt/batch_20251126_120000/sukses/prompts_success.txt:
A cinematic shot of a cat walking in rain
Epic drone view of mountain sunset
Close-up of coffee being poured
...

TrackingPrompt/batch_20251126_120000/gagal/prompts_failed.txt:
(prompts yang gagal - bisa retry manual)
```

**C. Statistics:**
```
Final Report:
✅ Total: 100
✅ Success: 98
❌ Failed: 2
⏱️ Duration: 16 menit 30 detik
🚀 Throughput: 6 video/menit

Success Rate: 98% ✅
```

---

### **🎯 STEP 8: Retry Failed (Jika Ada)**

**A. Manual Retry:**
```
1. Buka prompts_failed.txt
2. Copy prompts yang gagal
3. Paste ke prompt box di aplikasi
4. Klik START (retry with same settings)
```

**B. Adjust & Retry:**
```
Jika banyak yang failed:
1. Kurangi workers (e.g., 12 → 8)
2. Tambah delay (e.g., 7s → 10s)
3. Retry failed prompts
```

---

## ✅ CHECKLIST SEBELUM GENERATE

Print checklist ini sebelum batch generation besar:

```
□ AKUN
  □ Minimal 1 akun sudah ditambahkan
  □ Status akun: Active (hijau)
  □ Cookie valid (tidak expired)
  □ Jika multiple akun, pastikan BEDA akun (bukan duplikat)

□ PROMPTS/IMAGES
  □ File prompts.txt sudah disiapkan (T2V)
  □ Atau folder images sudah ready (I2V)
  □ Prompts sudah di-load ke aplikasi
  □ Jumlah prompts sesuai (check info box)

□ RATE LIMITING
  □ Mode dipilih: Formula by Sistem atau Manual
  □ Jika Manual: Settings sudah diatur optimal
  □ Info box show: "Terdeteksi: X akun | Total Workers: Y"
  □ Total workers masuk akal (tidak terlalu besar)

□ SETTINGS
  □ Provider dipilih (GenAI/Ultra)
  □ Mode dipilih (T2V/I2V)
  □ Aspect ratio sesuai kebutuhan
  □ Auto-mute/upscale sesuai kebutuhan
  □ Output directory set

□ SYSTEM
  □ Internet connection stabil
  □ Disk space cukup (min 10GB per 100 video)
  □ Aplikasi running smooth (no lag)
  □ FFmpeg installed (if auto-mute enabled)
  □ Topaz API key (if 4K upscale enabled)

□ FINAL CHECK
  □ Semua checkbox di atas ✅
  □ Siap monitor progress
  □ Tidak akan close aplikasi saat generate
  □ PC tidak akan sleep/hibernate

✅ ALL CLEAR! Klik START! 🚀
```

---

## 💡 TIPS PRO

### **1. Optimal Account Setup**

**Recommended Account Strategy:**
```
Pemula (Budget: $0):
- 1-2 akun gratis
- Workers: 4-8
- Throughput: 2-4 video/min
- Good for: Testing, small batches

Intermediate (Budget: Moderate):
- 3-5 akun
- Workers: 12-20
- Throughput: 6-8 video/min
- Good for: Regular production

Professional (Budget: High):
- 6-10 akun
- Workers: 24-40
- Throughput: 12-16 video/min
- Good for: Agency, mass production

Enterprise (Budget: Corporate):
- 10+ akun
- Workers: 40+
- Throughput: 16+ video/min
- Good for: Enterprise deployment
```

---

### **2. Prime Time vs Off-Peak**

**Server load berpengaruh pada speed:**

**Peak Hours (Lambat):**
```
🔴 08:00-12:00 WIB (Pagi)
🔴 13:00-17:00 WIB (Siang)
🔴 19:00-22:00 WIB (Malam)

Avg time: 8-10 menit per video
Rate limit risk: HIGHER
```

**Off-Peak Hours (Cepat):**
```
✅ 00:00-06:00 WIB (Dini hari)
✅ 12:00-13:00 WIB (Lunch break)
✅ 23:00-00:00 WIB (Late night)

Avg time: 5-7 menit per video
Rate limit risk: LOWER
```

**Pro Tip:**
```
Generate di malam hari (00:00-06:00):
- 30-40% lebih cepat
- Rate limit lebih longgar
- Bisa set workers lebih tinggi
```

---

### **3. Batch Size Strategy**

**Jangan generate semuanya sekaligus!**

**Small Batch First (Testing):**
```
1. Test batch: 10-20 prompts
2. Check results quality
3. Check error rate
4. Adjust settings if needed
5. Then scale up!
```

**Production Batch:**
```
Pemula: 50-100 prompts per batch
Intermediate: 100-300 prompts per batch
Professional: 300-1000 prompts per batch

Split besar batch:
1000 prompts → 10× batch @100
- Safer (jika error, tidak semua gagal)
- Easier to monitor
- Can adjust per batch
```

---

### **4. Prompt Quality Matters**

**Good Prompts = Higher Success Rate**

**Good Prompt Example:**
```
✅ "A cinematic shot of a red sports car driving through a winding mountain road at sunset, camera following from behind"

Why good:
- Clear subject (red sports car)
- Clear action (driving)
- Clear location (mountain road)
- Clear time (sunset)
- Clear camera (following from behind)
```

**Bad Prompt Example:**
```
❌ "car"

Why bad:
- Too vague
- No details
- AI confused
- Higher fail rate
```

**Pro Tips:**
```
✅ Be specific (colors, actions, camera angles)
✅ Use cinematic terms (drone shot, close-up, etc.)
✅ 10-30 words optimal length
✅ Avoid contradictions ("day and night")
✅ Test prompts individually first
```
---

### **5. Monitoring & Alerts**

**Watch These Metrics:**
```
✅ Active Workers: Should match settings
✅ Success Rate: Should be > 90%
✅ Failed Count: Should be minimal
✅ Progress %: Should increase steady

Red Flags:
❌ Workers stuck (tidak ada progress)
❌ Success rate < 70%
❌ Banyak error 429
❌ Connection errors bertubi-tubi
```

**What To Do:**
```
If Success Rate < 90%:
1. Click STOP
2. Review failed prompts log
3. Check error patterns
4. Adjust settings
5. Restart with smaller batch

If Error 429 Multiple Times:
1. Click STOP immediately
2. Wait 5 minutes (cooldown)
3. Reduce workers by 30-50%
4. Increase delays
5. Restart
```

---

## 🔧 TROUBLESHOOTING CEPAT

### **Problem: Workers Tidak Start**

**Gejala:**
```
[12:00:00] ✅ Generation started successfully!
(tidak ada log worker...)
```

**Cek:**
```
□ Prompts sudah diload? (check prompt box)
□ Images sudah diload? (I2V mode)
□ Akun ada & valid?
□ Internet connection OK?
```

**Solusi:**
```
1. Reload prompts/images
2. Check accounts tab
3. Restart aplikasi
4. Try again
```

---

### **Problem: Error 429 Terus-Menerus**

**Gejala:**
```
❌ [Worker-1] Error 429
❌ [Worker-2] Error 429
❌ [Worker-3] Error 429
(semua workers kena)
```

**Solusi Immediate:**
```
1. STOP generation
2. Wait 5 minutes
3. Pilih "Formula by Sistem"
4. START lagi

Jika masih error:
1. Kurangi Max Worker Per Akun: 4 → 3
2. Naikkan Delay: 15s → 20s
3. Try again
```

---

### **Problem: Generation Sangat Lambat**

**Gejala:**
```
1 video = 15-20 menit (harusnya 6-8 menit)
```

**Kemungkinan:**
```
□ Peak hours (server overload)
□ Internet lambat
□ Workers terlalu sedikit
□ Prompt terlalu complex
```

**Solusi:**
```
1. Check internet speed (speedtest.net)
2. Generate di off-peak hours (malam)
3. Tambah akun → tambah workers
4. Simplify prompts (kurangi detail)
```

---

### **Problem: Banyak Failed Videos**

**Gejala:**
```
✅ Success: 60
❌ Failed: 40
(40% fail rate!)
```

**Analisa:**
```
1. Buka prompts_failed.txt
2. Cek pattern error:
   - Semua timeout? → Server lambat
   - Semua error 429? → Rate limit
   - Semua error 401? → Cookie expired
   - Random? → Prompts issue
```

**Solusi:**
```
Pattern Timeout:
→ Generate di off-peak hours
→ Increase timeout setting

Pattern 429:
→ Reduce workers
→ Increase delays

Pattern 401:
→ Refresh cookie semua akun
→ Retry failed

Random:
→ Check prompt quality
→ Remove problematic prompts
→ Retry batch
```

---

### **Problem: Aplikasi Crash/Freeze**

**Gejala:**
```
Aplikasi not responding
Workers stuck
No log updates
```

**Emergency Steps:**
```
1. Task Manager → End Process
2. Restart aplikasi
3. Check logs:
   - TrackingPrompt/batch_xxx/sisa/
   - Ada remaining prompts?
4. Continue dari remaining prompts
```

**Pencegahan:**
```
✅ Jangan set workers > 35
✅ Jangan minimize saat generate
✅ Disk space cukup (min 20GB free)
✅ Close aplikasi lain (hemat RAM)
```

---

### **Problem: Cookie Expired Semua Akun**

**Gejala:**
```
❌ All workers: Error 401 Unauthorized
```

**Solusi Mass Refresh:**
```
1. Tab "Accounts"
2. Klik 🔄 untuk akun pertama
3. Login → Cookie updated
4. Ulangi untuk semua akun
5. Verifikasi semua status: Active
6. Back to generation tab
7. START lagi
```

**Scheduling:**
```
Set reminder:
□ Refresh cookie setiap 2 minggu
□ Check status sebelum batch besar
□ Keep browser logged in
```

---

## 🎓 PENUTUP

### **Key Takeaways:**

1. **Sistem v3.06 Beta = Staggered Start + Parallel Execution**
   - Workers start bertahap (delay 10-20s)
   - Setelah start, semua jalan parallel
   - Tidak ada queue blocking
   - Maximum throughput!

2. **2 Mode Sederhana:**
   - Formula by Sistem (auto, recommended)
   - Setting Your Rate Limiting (manual, advanced)

3. **Golden Rule:**
   - 1 akun = max 4-6 workers
   - Multiple akun = linear scalability
   - Delay 10-20 detik antar start
   - Tambah akun = fastest way to scale

4. **Error Prevention:**
   - Use "Formula by Sistem"
   - Don't overload (respect rate limit)
   - Generate di off-peak hours
   - Test small batch first

5. **Monitoring:**
   - Watch success rate (should be > 90%)
   - Stop immediately if error 429 multiple
   - Review logs regularly
   - Track remaining prompts

---

### **Support & Resources:**

**Documentation:**
- README.md (main documentation)
- CHANGELOG.md (version history)
- This file (rate limiting guide)

**Troubleshooting:**
- Check logs: TrackingPrompt/
- Review error messages
- Use troubleshooting section above

**Best Practices:**
- Start small, scale gradually
- Use Formula by Sistem first
- Monitor first batch carefully
- Keep cookies fresh
- Generate at off-peak hours

### **Credits:**

Developed with ❤️

---

**🎉 Selamat Generate! Semoga Sukses! 🚀**

---

*Last Updated: 2025-11-26*
*Version: 3.06 Beta*
*Document: RUMUS_WORKER.md*
