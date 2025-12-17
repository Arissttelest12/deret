# Logika Algoritma Game Deret Aritmatika

## 📋 Penjelasan Logika Algoritma

### 1. **Struktur Data Utama**

```javascript
class ArithmeticSequenceGame {
    level = 1;              // Level saat ini
    nyawa = 5;              // Jumlah nyawa pemain
    deret = [];             // Array berisi deret aritmatika lengkap
    emptyPositions = [];    // Array berisi index posisi yang kosong
    filledAnswers = {};     // Object untuk menyimpan jawaban yang sudah benar
    totalSlots = 10;        // Total kotak dalam deret
    emptySlots = 3;         // Jumlah kotak yang harus diisi
    
    levelConfig = {         // Konfigurasi setiap level
        1: { beda: 2, startRange: [1, 10] },
        2: { beda: 3, startRange: [1, 15] },
        ...
    };
}
```

### 2. **Algoritma Generate Deret Aritmatika**

**Logika:**
1. Dapatkan konfigurasi level (beda dan range angka mulai)
2. Generate angka mulai secara acak dalam range yang ditentukan
3. Buat deret dengan panjang 10 (totalSlots)
4. Setiap iterasi tambahkan beda ke angka sebelumnya

**Keunggulan:**
- Konsisten secara matematika
- Tidak ada ambiguitas
- Level scaling yang seimbang

### 3. **Algoritma Generate Posisi Kosong Secara Acak**

**Logika:**
1. Buat array berisi index 0-9 (untuk 10 slot)
2. Acak array menggunakan Fisher-Yates shuffle
3. Ambil 3 index pertama sebagai posisi kosong
4. Pastikan posisi kosong tersebar acak

**Keunggulan:**
- Tidak selalu di awal/tengah/akhir
- Setiap permainan berbeda
- Fair untuk pemain

### 4. **Algoritma Validasi Jawaban**

**Logika:**
1. Saat pemain menjawab, cek apakah angka sesuai dengan deret di posisi tersebut
2. Jika benar: tandai sebagai jawaban benar, update progress
3. Jika salah: kurangi nyawa, berikan feedback
4. Cek kemenangan jika semua jawaban sudah benar

### 5. **Algoritma Generate Pilihan Angka**

**Logika:**
1. Pastikan semua angka yang dibutuhkan (untuk mengisi kotak kosong) ada di pilihan
2. Tambahkan distraktor (angka salah) yang mirip dengan jawaban
   - ±beda dari jawaban
   - ±1, ±2 dari jawaban
3. Acak urutan pilihan
4. Batasi jumlah pilihan agar tidak terlalu banyak

**Keunggulan:**
- Semua jawaban yang dibutuhkan tersedia
- Distraktor yang challenging
- Tidak membingungkan pemain

### 6. **Algoritma Level Progression**

**Logika:**
1. Setiap level naik, beda bertambah 1
2. Range angka mulai semakin besar
3. Bonus nyawa (max 5)
4. Reset game dengan konfigurasi level baru

### 7. **Algoritma Feedback dan Animasi**

**Logika:**
1. Jawaban benar: animasi success, box menghilang
2. Jawaban salah: animasi shake, nyawa berkurang
3. Level selesai: animasi celebration
4. Game over: animasi defeat

---

## ✅ Kelebihan Algoritma Ini

1. **Konsisten Matematika**: Deret selalu mengikuti pola aritmatika yang benar
2. **Tidak Ambigu**: Setiap soal memiliki satu jawaban yang benar
3. **Acak yang Fair**: Posisi kosong acak, tapi tetap solvable
4. **Scalable**: Mudah ditambahkan level baru
5. **Challenging**: Distraktor yang cerdas membuat game menantang
6. **Educational**: Membantu pemahaman deret aritmatika
7. **Good UX**: Feedback jelas, animasi smooth

## 🔧 Implementasi Teknis

- **OOP Design**: Menggunakan class untuk encapsulation
- **Event-driven**: Drag & drop dengan proper event handling
- **Responsive**: Animasi smooth dengan Anime.js
- **User-friendly**: SweetAlert untuk feedback
- **Modular**: Setiap fungsi memiliki tanggung jawab yang jelas