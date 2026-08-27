# BOT MUSIC MINER
===========================


Panduan langkah demi langkah untuk menginstal dan menjalankan bot otomatis Music Mining ($MB) di Ubuntu Server:

## Langkah 1: Mengambil Token Autentikasi

Sebelum masuk ke server, ambil kredensial akun dari browser komputer:

Buka Telegram Web di browser (Chrome/Brave) dan login.

Tekan tombol `F12` untuk membuka Developer Tools, lalu pilih tab `Network` dan klik filter `Fetch/XHR`.

Buka Mini App MusicMining / Mint Block dan klik tombol Claim.

Lakukan aksi di dalam aplikasi (misalnya klik tombol Claim, Start/Mine, atau buka menu utama).

Cari request bernama claim di panel kiri, lalu klik tab Headers di panel kanan.

Salin nilai dari baris Authorization (contoh: `Bearer 746056|YO7vUDwy...`).


## Langkah 2: Persiapan Sistem di Ubuntu Server

Buka terminal Ubuntu Server Anda, lalu jalankan perintah berikut untuk memperbarui paket sistem, menginstal dependensi, dan membuat folder kerja:

```Bash
# 1. Update repositori sistem
sudo apt update

# 2. Install Python dan modul HTTP requests
sudo apt install python3 python3-pip python3-requests -y

# 3. Buat folder proyek baru dan masuk ke dalamnya
mkdir musicmb-bot
cd musicmb-bot
```

## Langkah 3: Membuat File Script Bot (main.py)

Salin dan tempel perintah di bawah ini ke terminal untuk membuat file script secara otomatis. Pastikan nilai variabel TOKEN disesuaikan dengan token akun yang disalin pada Langkah 1:

```Bash

cat << 'EOF' > main.py
import time
import random
import requests
from datetime import datetime

URL_CLAIM = "https://api.musicmb.site/api/mining/claim"
URL_AD_BOOST = "https://api.musicmb.site/api/mining/ad-boost/complete"

# Ganti nilai di bawah ini dengan token Authorization milik Anda
TOKEN = "Bearer 746056|YO7vUDwy5nbpckRjfmktUh5HgX9bLOM7Nyp180fi79da7ae0"

HEADERS = {
    "Accept": "application/json",
    "Content-Type": "application/json",
    "Authorization": TOKEN,
    "Origin": "https://musicmb.site",
    "Referer": "https://musicmb.site/",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36"
}

def log(msg):
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"[{now}] {msg}")

def execute_claim():
    try:
        res = requests.post(URL_CLAIM, headers=HEADERS, timeout=15)
        if res.status_code == 200:
            data = res.json()
            user = data.get("user", {})
            mining = data.get("mining", {})
            
            earned = data.get("earned", 0)
            balance = user.get("balance", "N/A")
            remaining = mining.get("remaining_seconds", 28800)
            
            log(f"[CLAIM] Sukses | Dapat: +{earned} MB | Saldo: {balance} MB")
            return remaining
        elif res.status_code == 401:
            log("[AUTH] Error 401: Token Bearer kedaluwarsa. Silakan perbarui TOKEN di main.py.")
            return 3600
        else:
            log(f"[CLAIM] Gagal: HTTP {res.status_code} - {res.text}")
            return 600
    except Exception as e:
        log(f"[CLAIM] Request error: {e}")
        return 300

def execute_turbo_boost(total_ads=10):
    log(f"[TURBO] Menjalankan {total_ads} iklan Turbo Boost...")
    success_count = 0
    for i in range(1, total_ads + 1):
        try:
            res = requests.post(URL_AD_BOOST, headers=HEADERS, timeout=15)
            if res.status_code == 200:
                success_count += 1
                log(f"[TURBO] Iklan {i}/{total_ads} berhasil diklaim.")
            elif res.status_code == 429:
                log("[TURBO] Terkena limit sementara, menunggu...")
                time.sleep(15)
            else:
                log(f"[TURBO] Iklan {i} gagal: HTTP {res.status_code}")
        except Exception as e:
            log(f"[TURBO] Error iklan {i}: {e}")
            
        if i < total_ads:
            time.sleep(random.randint(5, 9))
            
    log(f"[TURBO] Selesai: {success_count}/{total_ads} boost aktif.\n")

def main():
    print("==================================================")
    print("  Music Mining ($MB) - Auto Claim & Turbo Boost   ")
    print("==================================================")
    
    while True:
        remaining_seconds = execute_claim()
        execute_turbo_boost(total_ads=10)
        
        jitter = random.randint(60, 300)
        total_wait = remaining_seconds + jitter
        
        hours = total_wait // 3600
        minutes = (total_wait % 3600) // 60
        log(f"[SLEEP] Menunggu {hours} jam {minutes} menit ({total_wait} detik) ke siklus berikutnya...\n")
        
        time.sleep(total_wait)

if __name__ == "__main__":
    main()
EOF
```

### Jalankan bot dengan perintah:

```Bash
python3 main.py
```

---

## Langkah 4: Menjalankan Bot di Background (24/7) optional

Jalankan perintah ini agar bot tetap bekerja di server meskipun koneksi SSH atau aplikasi terminal ditutup:

```Bash
nohup python3 main.py > bot.log 2>&1 &
```

## Langkah 5: Perintah Pemantauan & Manajemen

Melihat status bot berjalan secara live:

```Bash
tail -f bot.log
```
(Tekan Ctrl + C untuk keluar dari tampilan log tanpa mematikan bot).

Memeriksa ID Proses (PID) bot:

```Bash
ps aux | grep "python3 main.py"
```

Menghentikan bot:

```Bash
kill <NOMOR_PID>
```
