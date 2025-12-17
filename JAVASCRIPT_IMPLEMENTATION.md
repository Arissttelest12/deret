# Implementasi JavaScript - Game Deret Aritmatika

## 📁 File: `game_deret_aritmatika_fixed.html`

### 🏗️ Struktur Class

```javascript
class ArithmeticSequenceGame {
    // Constructor - inisialisasi game
    constructor() {
        this.level = 1;
        this.nyawa = 5;
        this.deret = [];
        this.emptyPositions = [];
        this.filledAnswers = {};
        this.totalSlots = 10;
        this.emptySlots = 3;
        
        // Level configuration
        this.levelConfig = {
            1: { beda: 2, startRange: [1, 10] },
            2: { beda: 3, startRange: [1, 15] },
            3: { beda: 4, startRange: [1, 20] },
            4: { beda: 5, startRange: [5, 25] },
            5: { beda: 6, startRange: [5, 30] },
            6: { beda: 7, startRange: [10, 35] },
            7: { beda: 8, startRange: [10, 40] },
            8: { beda: 9, startRange: [15, 45] },
            9: { beda: 10, startRange: [15, 50] },
            10: { beda: 11, startRange: [20, 55] }
        };
        
        this.init();
    }
}
```

### 🔢 Generate Deret Aritmatika

```javascript
// Generate deret aritmatika berdasarkan level
generateDeret() {
    const config = this.levelConfig[this.level] || { 
        beda: this.level + 1, 
        startRange: [20, 50 + this.level * 5] 
    };
    
    const beda = config.beda;
    const [minStart, maxStart] = config.startRange;
    const startNumber = Math.floor(Math.random() * (maxStart - minStart + 1)) + minStart;
    
    const deret = [];
    let current = startNumber;
    
    for (let i = 0; i < this.totalSlots; i++) {
        deret.push(current);
        current += beda;
    }
    
    return deret;
}

// Contoh output Level 1:
// beda = 2, startNumber = 3
// Output: [3, 5, 7, 9, 11, 13, 15, 17, 19, 21]
```

### 🎲 Generate Posisi Kosong Secara Acak

```javascript
// Generate posisi kosong secara acak
generateEmptyPositions() {
    const positions = [];
    const available = Array.from({length: this.totalSlots}, (_, i) => i);
    
    // Fisher-Yates shuffle
    for (let i = available.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [available[i], available[j]] = [available[j], available[i]];
    }
    
    // Ambil 3 posisi pertama
    return available.slice(0, this.emptySlots);
}

// Contoh output: [2, 7, 9] (index acak)
```

### 🎯 Setup Level

```javascript
// Setup game untuk level saat ini
setupLevel() {
    this.deret = this.generateDeret();
    this.emptyPositions = this.generateEmptyPositions();
    this.filledAnswers = {};
    
    console.log(`Level ${this.level} - Deret: [${this.deret.join(', ')}]`);
    console.log(`Posisi kosong: [${this.emptyPositions.join(', ')}]`);
}
```

### 🎨 Render Slots

```javascript
// Render slots ke DOM
renderSlots() {
    const slotsDiv = document.getElementById("slots");
    slotsDiv.innerHTML = '';
    
    for (let i = 0; i < this.totalSlots; i++) {
        const slot = document.createElement("div");
        
        if (this.emptyPositions.includes(i)) {
            // Slot yang harus diisi
            slot.className = "slot empty";
            slot.id = `slot-${i}`;
            slot.dataset.answer = this.deret[i];
            slot.innerText = '?';
            slot.addEventListener("dragover", this.dragOver.bind(this));
            slot.addEventListener("dragleave", this.dragLeave.bind(this));
            slot.addEventListener("drop", this.drop.bind(this));
        } else {
            // Slot fixed (sudah terisi)
            slot.className = "slot fixed";
            slot.innerText = this.deret[i];
        }
        
        slot.style.opacity = '0';
        slot.style.transform = 'scale(0.8) translateY(20px)';
        slotsDiv.appendChild(slot);
    }
}
```

### 🎲 Generate Pilihan Angka

```javascript
// Render pilihan angka
renderChoices() {
    const choicesDiv = document.getElementById("choices");
    choicesDiv.innerHTML = '';
    
    // Ambil angka yang harus diisi
    const neededNumbers = this.emptyPositions.map(pos => this.deret[pos]);
    
    // Tambahkan distraktor (angka salah tapi mirip)
    const distractors = this.generateDistractors(neededNumbers);
    
    // Gabungkan dan acak
    const allChoices = [...neededNumbers, ...distractors];
    allChoices.sort(() => Math.random() - 0.5);
    
    // Pastikan semua angka yang dibutuhkan ada
    const finalChoices = this.ensureAllNeededNumbers(neededNumbers, allChoices);
    
    finalChoices.forEach((num, index) => {
        const box = document.createElement("div");
        box.className = "box";
        box.innerText = num;
        box.draggable = true;
        box.id = `box-${num}-${index}`;
        box.addEventListener("dragstart", this.dragStart.bind(this));
        box.style.opacity = '0';
        box.style.transform = 'scale(0.8) translateY(20px)';
        choicesDiv.appendChild(box);
    });
}

// Generate distraktor yang mirip dengan jawaban
generateDistractors(neededNumbers) {
    const distractors = [];
    const beda = this.levelConfig[this.level]?.beda || (this.level + 1);
    
    neededNumbers.forEach(num => {
        // Tambahkan angka yang mirip (±beda, ±1, ±2)
        [-beda, -beda+1, -1, 1, beda-1, beda].forEach(offset => {
            const distractor = num + offset;
            if (distractor > 0 && !neededNumbers.includes(distractor) && !this.deret.includes(distractor)) {
                distractors.push(distractor);
            }
        });
    });
    
    // Hapus duplikat dan batasi jumlah
    return [...new Set(distractors)].slice(0, 6);
}

// Pastikan semua angka yang dibutuhkan ada di pilihan
ensureAllNeededNumbers(neededNumbers, allChoices) {
    const missing = neededNumbers.filter(num => !allChoices.includes(num));
    return [...allChoices, ...missing];
}
```

### 🖱️ Drag & Drop Handlers

```javascript
// Drag start handler
dragStart(e) {
    this.dragged = e.target;
    e.target.style.opacity = '0.5';
    e.target.style.transform = 'scale(0.9)';
}

// Drag over handler
dragOver(e) {
    e.preventDefault();
    e.target.classList.add('drag-over');
}

// Drag leave handler
dragLeave(e) {
    e.target.classList.remove('drag-over');
}

// Drop handler
drop(e) {
    const slot = e.target;
    slot.classList.remove('drag-over');
    
    const nilai = parseInt(this.dragged.innerText);
    const kunci = parseInt(slot.dataset.answer);
    
    // Validasi jawaban
    if (nilai !== kunci) {
        this.handleWrongAnswer(slot);
        return;
    }
    
    // Jawaban benar
    this.handleCorrectAnswer(slot, nilai);
}
```

### ✅ Validasi Jawaban

```javascript
// Handle jawaban salah
handleWrongAnswer(slot) {
    this.nyawa--;
    this.updateUI();
    
    // Animasi nyawa berkurang
    anime({
        targets: '.heart',
        scale: [1, 1.3, 1],
        duration: 300,
        easing: 'easeInOutQuad'
    });
    
    // Animasi slot salah
    anime({
        targets: slot,
        scale: [1, 1.1, 1],
        backgroundColor: ['rgba(255,255,255,0.2)', 'rgba(255,107,107,0.5)', 'rgba(255,255,255,0.2)'],
        duration: 600,
        easing: 'easeInOutQuad'
    });
    
    Swal.fire({
        title: "Salah!",
        text: `Nyawa berkurang! Sisa ${this.nyawa} nyawa`,
        icon: "error",
        confirmButtonColor: '#ff6b6b',
        background: 'rgba(255,255,255,0.95)',
        backdrop: 'rgba(0,0,0,0.3)'
    });
    
    if (this.nyawa === 0) {
        this.gameOver();
    }
}

// Handle jawaban benar
handleCorrectAnswer(slot, nilai) {
    slot.innerText = nilai;
    slot.classList.remove('empty');
    slot.classList.add('correct');
    
    // Animasi box menghilang
    anime({
        targets: this.dragged,
        scale: [1, 0],
        opacity: [1, 0],
        rotate: '1turn',
        duration: 500,
        easing: 'easeInBack',
        complete: () => {
            this.dragged.remove();
        }
    });
    
    // Catat jawaban yang benar
    this.filledAnswers[slot.id] = nilai;
    this.updateProgress();
    
    // Cek menang
    if (Object.keys(this.filledAnswers).length === this.emptySlots) {
        this.levelComplete();
    }
}
```

### 🎉 Game State Management

```javascript
// Update progress bar
updateProgress() {
    const progress = (Object.keys(this.filledAnswers).length / this.emptySlots) * 100;
    document.getElementById('progressFill').style.width = progress + '%';
}

// Level selesai
levelComplete() {
    // Animasi sukses
    const successDiv = document.createElement('div');
    successDiv.className = 'success-animation';
    successDiv.innerHTML = '🎉';
    document.body.appendChild(successDiv);
    
    anime({
        targets: successDiv,
        scale: [0, 1.5, 1],
        rotate: [0, 360],
        duration: 1000,
        easing: 'easeOutElastic(1, .8)',
        complete: () => {
            setTimeout(() => {
                successDiv.remove();
            }, 500);
        }
    });
    
    Swal.fire({
        title: "Hebat! 🎉",
        text: `Level ${this.level} selesai! Lanjut ke level berikutnya?`,
        icon: "success",
        confirmButtonColor: '#38ef7d',
        confirmButtonText: `Lanjut Level ${this.level + 1}`,
        background: 'rgba(255,255,255,0.95)',
        backdrop: 'rgba(0,0,0,0.3)'
    }).then((result) => {
        if (result.isConfirmed) {
            this.nextLevel();
        }
    });
}

// Level berikutnya
nextLevel() {
    this.level++;
    this.nyawa = Math.min(this.nyawa + 1, 5); // Bonus nyawa, max 5
    this.setupLevel();
    this.render();
}

// Game over
gameOver() {
    Swal.fire({
        title: "Game Over!",
        text: `Kamu mencapai level ${this.level}. Coba lagi?`,
        icon: "error",
        confirmButtonColor: '#ff6b6b',
        background: 'rgba(255,255,255,0.95)',
        backdrop: 'rgba(0,0,0,0.3)'
    }).then(() => {
        // Reset game
        this.level = 1;
        this.nyawa = 5;
        this.setupLevel();
        this.render();
    });
}
```

### 🎨 UI Updates

```javascript
// Update UI elements
updateUI() {
    document.getElementById('currentLevel').textContent = this.level;
    
    let hearts = "";
    for (let i = 1; i <= 5; i++) {
        hearts += `<span class="heart">${i <= this.nyawa ? "❤️" : "🖤"}</span>`;
    }
    document.getElementById("nyawa").innerHTML = hearts;
}

// Create floating particles
createParticles() {
    const particlesContainer = document.getElementById('particles');
    particlesContainer.innerHTML = '';
    
    for (let i = 0; i < 15; i++) {
        const particle = document.createElement('div');
        particle.className = 'particle';
        particle.style.left = Math.random() * 100 + '%';
        particle.style.top = Math.random() * 100 + '%';
        particle.style.width = (Math.random() * 10 + 5) + 'px';
        particle.style.height = particle.style.width;
        particle.style.animationDelay = Math.random() * 6 + 's';
        particle.style.animationDuration = (Math.random() * 3 + 4) + 's';
        particlesContainer.appendChild(particle);
    }
}
```

### 🚀 Initialization

```javascript
// Initialize game when DOM is loaded
document.addEventListener('DOMContentLoaded', () => {
    new ArithmeticSequenceGame();
});
```

## 📁 Struktur File

```
/mnt/okcomputer/output/
├── game_deret_aritmatika_fixed.html  # Main game file
├── ALGORITMA.md                       # Penjelasan logika algoritma
├── PSEUDOCODE.md                      # Pseudocode lengkap
└── JAVASCRIPT_IMPLEMENTATION.md       # Implementasi JavaScript
```

## 🎯 Cara Menggunakan

1. Buka file `game_deret_aritmatika_fixed.html` di browser
2. Game akan otomatis start di level 1
3. Drag angka dari pilihan ke kotak kosong yang sesuai
4. Jawaban benar akan menghilangkan angka dari pilihan
5. Jawaban salah akan mengurangi nyawa
6. Selesaikan semua kotak untuk naik level

## ✅ Features yang Sudah Diimplementasikan

- ✅ Deret aritmatika konsisten (beda +1 setiap level)
- ✅ 10 kotak total, 3 kotak kosong acak
- ✅ Pilihan angka hanya dari deret yang benar + distraktor
- ✅ Validasi jawaban dengan feedback
- ✅ Level progression dengan scaling difficulty
- ✅ Nyawa system dengan bonus
- ✅ Smooth animations dengan Anime.js
- ✅ Responsive design
- ✅ Progress tracking
- ✅ Game state management yang proper