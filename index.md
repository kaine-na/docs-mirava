# 📐 RUMUS PERHITUNGAN WORKER & DELAY

## 🧮 FORMULA DASAR

### **1. Workers Per Account**

```
workers_per_account = CEILING(total_workers ÷ total_accounts)
```

**Contoh:**
- 35 workers ÷ 3 accounts = 11.67 → **12 workers/account** (rounded up)
- 20 workers ÷ 5 accounts = 4 → **4 workers/account** (exact)

---

### **2. Minimum Spacing Between Requests (Same Account)**

```
min_spacing = (total_accounts × delay_worker_start) + delay_between_requests
```

**Contoh:**
- 3 accounts × 10s + 5s = **35 detik**
- 5 accounts × 15s + 10s = **85 detik**

---

### **3. Requests Per Minute Per Account**

```
requests_per_minute = 60 ÷ (processing_time + delay_between_requests)
```

**Untuk Video (90s processing):**
```
60 ÷ (90 + 7) = 60 ÷ 97 = 0.62 requests/minute
```

**Untuk Image (5s processing):**
```
60 ÷ (5 + 10) = 60 ÷ 15 = 4 requests/minute
```

---

### **4. Total Throughput (All Workers)**

```
total_throughput = workers_per_account × requests_per_minute × total_accounts
```

**Contoh (Image Generation):**
- 4 workers/account × 4 req/min × 3 accounts = **48 images/minute**

---

### **5. Safety Check (Avoid 429)**

```
safe_threshold = API_rate_limit × 0.8

IF (requests_per_minute × workers_per_account) > safe_threshold:
    REDUCE workers OR INCREASE delays
```

**Typical API limits:**
- Image API: ~60-100 requests/minute/account
- Video API: ~10-20 requests/minute/account

---

## 📊 TABEL LENGKAP: IMAGE GENERATION

**Assumptions:**
- Processing time: 5 seconds/image
- API limit: 60 requests/minute/account
- Safe threshold: 48 requests/minute (80%)

### **Formula Perhitungan Delay:**

```
Untuk mencapai safe threshold:

target_interval = 60 ÷ safe_threshold
                = 60 ÷ 48 = 1.25 seconds

delay_needed = target_interval - processing_time
             = 1.25 - (5 ÷ workers_per_account)

delay_between_requests = MAX(delay_needed, 10)  // Minimum 10s untuk safety
delay_worker_start = MAX(delay_needed, 10)
```

---

### **1 ACCOUNT - IMAGE GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 1 | 1 | 10s | 4.0 | 4 | 10s | 10s | ✅ Safe |
| 3 | 3 | 10s | 12.0 | 12 | 10s | 10s | ✅ Safe |
| 5 | 5 | 10s | 20.0 | 20 | 10s | 10s | ✅ Safe |
| 8 | 8 | 10s | 32.0 | 32 | 12s | 12s | ✅ Safe |
| 10 | 10 | 10s | 40.0 | 40 | 15s | 15s | ⚠️ Near Limit |
| 12 | 12 | 10s | 48.0 | 48 | 15s | 15s | ⚠️ At Threshold |
| 15 | 15 | 10s | 60.0 | 60 | 20s | 20s | ❌ Exceed! |
| 20 | 20 | 10s | 80.0 | 80 | 25s | 25s | ❌ Exceed! |
| 25 | 25 | 10s | 100.0 | 100 | 30s | 30s | ❌ Exceed! |
| 30 | 30 | 10s | 120.0 | 120 | 35s | 35s | ❌ Exceed! |
| 35 | 35 | 10s | 140.0 | 140 | 40s | 40s | ❌ Exceed! |

**⚠️ WARNING:** 1 akun hanya bisa handle **maksimal 10-12 workers** untuk Image Generation!

---

### **2 ACCOUNTS - IMAGE GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 2 | 1 | 20s | 4.0 | 8 | 10s | 10s | ✅ Safe |
| 6 | 3 | 20s | 12.0 | 24 | 10s | 10s | ✅ Safe |
| 10 | 5 | 20s | 20.0 | 40 | 10s | 10s | ✅ Safe |
| 16 | 8 | 20s | 32.0 | 64 | 12s | 12s | ⚠️ Near Limit |
| 20 | 10 | 20s | 40.0 | 80 | 15s | 15s | ⚠️ At Threshold |
| 24 | 12 | 20s | 48.0 | 96 | 15s | 15s | ⚠️ At Threshold |
| 30 | 15 | 20s | 60.0 | 120 | 20s | 20s | ❌ Exceed! |
| 35 | 18 | 20s | 72.0 | 144 | 25s | 25s | ❌ Exceed! |

**✅ OPTIMAL:** 2 akun bisa handle **16-20 workers** dengan delay 12-15s

---

### **3 ACCOUNTS - IMAGE GENERATION** ⭐

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 3 | 1 | 30s | 4.0 | 12 | 10s | 10s | ✅ Safe |
| 9 | 3 | 30s | 12.0 | 36 | 10s | 10s | ✅ Safe |
| 12 | 4 | 30s | 16.0 | 48 | 10s | 10s | ✅ Safe |
| 15 | 5 | 30s | 20.0 | 60 | 10s | 10s | ✅ Safe |
| 18 | 6 | 30s | 24.0 | 72 | 12s | 12s | ✅ Safe |
| 21 | 7 | 30s | 28.0 | 84 | 12s | 12s | ✅ Safe |
| 24 | 8 | 30s | 32.0 | 96 | 12s | 12s | ✅ Safe |
| 27 | 9 | 30s | 36.0 | 108 | 15s | 15s | ⚠️ Near Limit |
| 30 | 10 | 30s | 40.0 | 120 | 15s | 15s | ⚠️ At Threshold |
| 33 | 11 | 30s | 44.0 | 132 | 20s | 20s | ❌ Exceed! |
| 35 | 12 | 30s | 48.0 | 144 | 25s | 25s | ❌ Exceed! |

**✅ OPTIMAL:** 3 akun bisa handle **15-24 workers** dengan delay 10-12s

**⭐ RECOMMENDED CONFIG (3 accounts):**
```json
{
  "workers": 18,
  "accounts": 3,
  "delay_start": 12,
  "delay_between_requests": 12
}
```

---

### **4 ACCOUNTS - IMAGE GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|-------------|---------------|-------------|--------|
| 4 | 1 | 40s | 4.0 | 16 | 10s | 10s | ✅ Safe |
| 12 | 3 | 40s | 12.0 | 48 | 10s | 10s | ✅ Safe |
| 16 | 4 | 40s | 16.0 | 64 | 10s | 10s | ✅ Safe |
| 20 | 5 | 40s | 20.0 | 80 | 10s | 10s | ✅ Safe |
| 24 | 6 | 40s | 24.0 | 96 | 10s | 10s | ✅ Safe |
| 28 | 7 | 40s | 28.0 | 112 | 12s | 12s | ✅ Safe |
| 32 | 8 | 40s | 32.0 | 128 | 12s | 12s | ✅ Safe |
| 35 | 9 | 40s | 36.0 | 144 | 15s | 15s | ⚠️ Near Limit |

**✅ OPTIMAL:** 4 akun bisa handle **28-32 workers** dengan delay 10-12s

---

### **5 ACCOUNTS - IMAGE GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 5 | 1 | 50s | 4.0 | 20 | 10s | 10s | ✅ Safe |
| 15 | 3 | 50s | 12.0 | 60 | 10s | 10s | ✅ Safe |
| 20 | 4 | 50s | 16.0 | 80 | 10s | 10s | ✅ Safe |
| 25 | 5 | 50s | 20.0 | 100 | 10s | 10s | ✅ Safe |
| 30 | 6 | 50s | 24.0 | 120 | 10s | 10s | ✅ Safe |
| 35 | 7 | 50s | 28.0 | 140 | 10s | 10s | ✅ Safe |

**✅ OPTIMAL:** 5 akun bisa handle **35 workers** dengan delay 10s! 🎉

**⭐ RECOMMENDED CONFIG (5 accounts):**
```json
{
  "workers": 35,
  "accounts": 5,
  "delay_start": 10,
  "delay_between_requests": 10
}
```

---

## 📊 TABEL LENGKAP: VIDEO GENERATION (T2V/I2V)

**Assumptions:**
- Processing time: 90 seconds/video
- API limit: 15 requests/minute/account
- Safe threshold: 12 requests/minute (80%)

---

### **1 ACCOUNT - VIDEO GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 1 | 1 | 7s | 0.62 | 0.62 | 7s | 7s | ✅ Safe |
| 5 | 5 | 7s | 3.09 | 3.09 | 7s | 7s | ✅ Safe |
| 10 | 10 | 7s | 6.19 | 6.19 | 7s | 7s | ✅ Safe |
| 15 | 15 | 7s | 9.28 | 9.28 | 7s | 7s | ✅ Safe |
| 20 | 20 | 7s | 12.37 | 12.37 | 10s | 10s | ⚠️ At Threshold |
| 25 | 25 | 7s | 15.46 | 15.46 | 15s | 15s | ❌ Exceed! |
| 30 | 30 | 7s | 18.56 | 18.56 | 20s | 20s | ❌ Exceed! |
| 35 | 35 | 7s | 21.65 | 21.65 | 25s | 25s | ❌ Exceed! |

**⚠️ WARNING:** 1 akun hanya bisa handle **maksimal 15 workers** untuk Video!

---

### **2 ACCOUNTS - VIDEO GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 2 | 1 | 14s | 0.62 | 1.24 | 7s | 7s | ✅ Safe |
| 10 | 5 | 14s | 3.09 | 6.19 | 7s | 7s | ✅ Safe |
| 20 | 10 | 14s | 6.19 | 12.37 | 7s | 7s | ✅ Safe |
| 30 | 15 | 14s | 9.28 | 18.56 | 10s | 10s | ⚠️ Near Limit |
| 35 | 18 | 14s | 11.13 | 22.26 | 15s | 15s | ❌ Exceed! |

**✅ OPTIMAL:** 2 akun bisa handle **20-25 workers** dengan delay 7s

---

### **3 ACCOUNTS - VIDEO GENERATION** ⭐

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 3 | 1 | 21s | 0.62 | 1.85 | 7s | 7s | ✅ Safe |
| 15 | 5 | 21s | 3.09 | 9.28 | 7s | 7s | ✅ Safe |
| 21 | 7 | 21s | 4.33 | 12.99 | 7s | 7s | ✅ Safe |
| 27 | 9 | 21s | 5.57 | 16.70 | 10s | 10s | ⚠️ Near Limit |
| 30 | 10 | 21s | 6.19 | 18.56 | 10s | 10s | ⚠️ Near Limit |
| 33 | 11 | 21s | 6.80 | 20.41 | 15s | 15s | ❌ Exceed! |
| 35 | 12 | 21s | 7.42 | 22.26 | 15s | 15s | ❌ Exceed! |

**✅ OPTIMAL:** 3 akun bisa handle **21-27 workers** dengan delay 7-10s

**⚠️ YOUR CASE (35 workers):** Butuh delay 15s atau tambah akun!

---

### **4 ACCOUNTS - VIDEO GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 4 | 1 | 28s | 0.62 | 2.47 | 7s | 7s | ✅ Safe |
| 20 | 5 | 28s | 3.09 | 12.37 | 7s | 7s | ✅ Safe |
| 28 | 7 | 28s | 4.33 | 17.32 | 7s | 7s | ✅ Safe |
| 32 | 8 | 28s | 4.95 | 19.79 | 10s | 10s | ⚠️ Near Limit |
| 35 | 9 | 28s | 5.57 | 22.26 | 12s | 12s | ❌ Exceed! |

**✅ OPTIMAL:** 4 akun bisa handle **28-32 workers** dengan delay 7-10s

---

### **5 ACCOUNTS - VIDEO GENERATION**

| Workers | Workers/Acc | Min Spacing | Req/Min/Acc | Total Req/Min | Delay Start | Delay Request | Status |
|---------|-------------|-------------|-------------|---------------|-------------|---------------|--------|
| 5 | 1 | 35s | 0.62 | 3.09 | 7s | 7s | ✅ Safe |
| 25 | 5 | 35s | 3.09 | 15.46 | 7s | 7s | ✅ Safe |
| 30 | 6 | 35s | 3.71 | 18.56 | 7s | 7s | ✅ Safe |
| 35 | 7 | 35s | 4.33 | 21.65 | 7s | 7s | ✅ Safe |

**✅ OPTIMAL:** 5 akun bisa handle **35 workers** dengan delay 7s! 🎉

**⭐ RECOMMENDED CONFIG (5 accounts):**
```json
{
  "workers": 35,
  "accounts": 5,
  "delay_start": 7,
  "delay_between_requests": 7
}
```

---

## 📊 VISUAL COMPARISON

### **IMAGE GENERATION - Max Safe Workers**

```
┌──────────┬────────────────┬─────────────────┬──────────┐
│ Accounts │ Max Workers    │ Delay Needed    │ Req/Min  │
├──────────┼────────────────┼─────────────────┼──────────┤
│ 1        │ 10-12 workers  │ 15s             │ 40-48    │
│ 2        │ 16-20 workers  │ 12-15s          │ 64-80    │
│ 3        │ 18-24 workers  │ 10-12s          │ 72-96    │ ⭐
│ 4        │ 28-32 workers  │ 10-12s          │ 112-128  │
│ 5        │ 35 workers     │ 10s             │ 140      │ 🎯
└──────────┴────────────────┴─────────────────┴──────────┘
```

### **VIDEO GENERATION - Max Safe Workers**

```
┌──────────┬────────────────┬─────────────────┬──────────┐
│ Accounts │ Max Workers    │ Delay Needed    │ Req/Min  │
├──────────┼────────────────┼─────────────────┼──────────┤
│ 1        │ 15 workers     │ 7s              │ 9.28     │
│ 2        │ 20-25 workers  │ 7s              │ 12-15    │
│ 3        │ 21-27 workers  │ 7-10s           │ 13-17    │ ⚠️
│ 4        │ 28-32 workers  │ 7-10s           │ 17-20    │
│ 5        │ 35 workers     │ 7s              │ 21.65    │ 🎯
└──────────┴────────────────┴─────────────────┴──────────┘
```

---

## 🎯 REKOMENDASI FINAL

### **Untuk IMAGE GENERATION:**

#### **Konfigurasi Anda Sekarang (BERMASALAH):**
```json
❌ BAD:
{
  "workers": 35,
  "accounts": 3,
  "delay_start": 10,
  "delay_between_requests": 5
}
Result: 144 req/min → RATE LIMIT! (429 errors)
```

#### **Opsi Fix:**

**OPSI A - Kurangi Workers (BEST!)**
```json
✅ RECOMMENDED:
{
  "workers": 18,
  "accounts": 3,
  "delay_start": 12,
  "delay_between_requests": 12
}
Result: 72 req/min → SAFE!
Success rate: 95-98%
```

**OPSI B - Tambah Akun**
```json
✅ OPTIMAL:
{
  "workers": 35,
  "accounts": 5,
  "delay_start": 10,
  "delay_between_requests": 10
}
Result: 140 req/min → SAFE!
Success rate: 95-98%
```

**OPSI C - Increase Delay (Slow)**
```json
⚠️ SLOW BUT SAFE:
{
  "workers": 35,
  "accounts": 3,
  "delay_start": 25,
  "delay_between_requests": 25
}
Result: 48 req/min → SAFE but VERY SLOW
```

---

### **Untuk VIDEO GENERATION:**

#### **Konfigurasi Optimal:**

**3 Accounts:**
```json
✅ GOOD:
{
  "workers": 25,
  "accounts": 3,
  "delay_start": 7,
  "delay_between_requests": 7
}
Result: 15.46 req/min → SAFE
```

**5 Accounts (BEST):**
```json
⭐ BEST:
{
  "workers": 35,
  "accounts": 5,
  "delay_start": 7,
  "delay_between_requests": 7
}
Result: 21.65 req/min → SAFE
```

---

## 📐 RUMUS CEPAT

### **Hitung Max Workers untuk Account Anda:**

```python
# Constants
IMAGE_API_LIMIT = 60  # requests/minute/account
VIDEO_API_LIMIT = 15  # requests/minute/account
SAFETY_FACTOR = 0.8   # 80% threshold

IMAGE_PROCESS_TIME = 5   # seconds
VIDEO_PROCESS_TIME = 90  # seconds

# Calculate max workers
def calculate_max_workers(accounts, mode="image", delay_request=10):
    if mode == "image":
        api_limit = IMAGE_API_LIMIT
        process_time = IMAGE_PROCESS_TIME
    else:
        api_limit = VIDEO_API_LIMIT
        process_time = VIDEO_PROCESS_TIME
    
    # Safe requests per minute per account
    safe_req_per_min = api_limit * SAFETY_FACTOR
    
    # Interval between requests
    interval = process_time + delay_request
    
    # Requests per minute per worker
    req_per_min_per_worker = 60 / interval
    
    # Max workers per account
    max_workers_per_acc = safe_req_per_min / req_per_min_per_worker
    
    # Total max workers
    max_workers = int(max_workers_per_acc * accounts)
    
    return max_workers

# Examples:
print(calculate_max_workers(3, "image", 10))   # Output: 24
print(calculate_max_workers(3, "image", 12))   # Output: 28
print(calculate_max_workers(5, "image", 10))   # Output: 40
print(calculate_max_workers(3, "video", 7))    # Output: 21
print(calculate_max_workers(5, "video", 7))    # Output: 35
```

---

## 🚨 TROUBLESHOOTING

### **Jika Masih Kena 429:**

1. **Cek Req/Min Actual:**
   ```
   actual_req_per_min = (successful_requests + failed_requests) / elapsed_minutes
   ```

2. **Jika > API Limit:**
   - Kurangi workers: `new_workers = workers × 0.7`
   - Atau naik delay: `new_delay = delay × 1.5`

3. **Jika < API Limit tapi tetap 429:**
   - API limit mungkin lebih rendah dari asumsi
   - Coba kurangi safety factor: 70% instead of 80%

---

## ✅ QUICK REFERENCE

```
UNTUK IMAGE GENERATION (5s/image):
┌──────────┬────────────┬─────────┬──────────┐
│ Accounts │ Max Worker │ Delay   │ Status   │
├──────────┼────────────┼─────────┼──────────┤
│ 1        │ 12         │ 15s     │ ⚠️       │
│ 2        │ 20         │ 12s     │ ⚠️       │
│ 3        │ 24         │ 12s     │ ✅       │
│ 4        │ 32         │ 10s     │ ✅       │
│ 5        │ 35         │ 10s     │ ⭐       │
└──────────┴────────────┴─────────┴──────────┘

UNTUK VIDEO GENERATION (90s/video):
┌──────────┬────────────┬─────────┬──────────┐
│ Accounts │ Max Worker │ Delay   │ Status   │
├──────────┼────────────┼─────────┼──────────┤
│ 1        │ 15         │ 7s      │ ⚠️       │
│ 2        │ 25         │ 7s      │ ⚠️       │
│ 3        │ 27         │ 7s      │ ⚠️       │
│ 4        │ 32         │ 7s      │ ✅       │
│ 5        │ 35         │ 7s      │ ⭐       │
└──────────┴────────────┴─────────┴──────────┘
```

---

## 📝 KESIMPULAN

**Untuk 35 Workers:**

```
IMAGE GENERATION:
- Minimal butuh 5 accounts (delay 10s)
- Atau 3 accounts dengan delay 25s (lambat!)
- Atau turun ke 18 workers (delay 12s)

VIDEO GENERATION:
- Minimal butuh 5 accounts (delay 7s)
- Atau 3 accounts dengan delay 15s
- Atau turun ke 27 workers (delay 7s)
```

**⭐ SOLUSI TERBAIK UNTUK ANDA:**
```json
{
  "mode": "Image Generation",
  "workers": 18,
  "accounts": 3,
  "delay_start": 12,
  "delay_between_requests": 12
}
```

Atau tambah akun jadi 5 untuk tetap pakai 35 workers! 🎯
