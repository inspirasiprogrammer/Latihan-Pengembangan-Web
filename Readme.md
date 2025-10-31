# 📸 Aplikasi Galeri Foto dengan Peringkat  

---

## 🧭 Deskripsi Proyek
Mahasiswa diminta membangun aplikasi web sederhana untuk **menampilkan galeri foto** dan **memberikan peringkat (rating)** pada setiap foto.  
Foto diambil dari **API eksternal** `https://jsonplaceholder.typicode.com/photos`, sementara data peringkat disimpan pada **database lokal (MySQL)** dan diakses melalui **API backend** berbasis Express.js.

Aplikasi ini terdiri atas dua bagian utama:
1. **Backend & Database (50%)** – menggunakan Node.js, Express, dan MySQL.  
2. **Frontend (50%)** – menggunakan HTML, CSS, dan JavaScript dengan Fetch API dan DOM manipulation.

---

## 🎯 Tujuan Pembelajaran
Setelah menyelesaikan proyek ini, mahasiswa diharapkan mampu:

- Membuat server REST API sederhana menggunakan Express.js.  
- Menghubungkan Node.js dengan database MySQL menggunakan mysql2.  
- Mengimplementasikan operasi `GET` dan `POST` dalam API.  
- Mengambil dan menampilkan data dari API eksternal menggunakan Fetch API.  
- Melakukan manipulasi DOM untuk membuat elemen HTML secara dinamis.  
- Mengintegrasikan frontend dengan backend secara asynchronous.  

---

## 🧰 Tools dan Teknologi
| Tools / Teknologi | Fungsi |
|--------------------|--------|
| **Node.js v18+** | Runtime JavaScript untuk backend |
| **Express.js** | Framework backend REST API |
| **MySQL2** | Library konektor database MySQL |
| **HTML5, CSS3, JavaScript (ES6+)** | Teknologi frontend |
| **Postman** | Uji coba endpoint API |
| **Browser (Chrome/Edge)** | Tampilan hasil aplikasi |

---

## 🗂️ Struktur Folder
```
/Praktikum
│
├── /public
│   ├── index.html
│   ├── style.css
│   └── script.js
│
├── index.js
└── package.json
```

---

## ⚙️ Bagian 1: Backend & Database (50 poin)

### 1️⃣ Membuat Database
```sql
CREATE DATABASE db_galeri_foto;
USE db_galeri_foto;

CREATE TABLE peringkat_foto (
  id INT PRIMARY KEY AUTO_INCREMENT,
  photo_id INT NOT NULL,
  nilai INT NOT NULL
);
```

---

### 2️⃣ Inisialisasi Server Node.js
```bash
npm init -y
npm install express mysql2
```

Buat file `index.js` dan isi sebagai berikut:
```js
const express = require("express");
const mysql = require("mysql2");
const app = express();
const PORT = 3000;

app.use(express.json());
app.use(express.static("public"));

const db = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "",
  database: "db_galeri_foto"
});

db.connect(err => {
  if (err) throw err;
  console.log("✅ Koneksi ke database berhasil!");
});

// GET rata-rata peringkat per foto
app.get("/api/peringkat/:photo_id", (req, res) => {
  const photoId = req.params.photo_id;
  db.query(
    "SELECT SUM(nilai) AS sum_nilai FROM peringkat_foto WHERE photo_id = ?",
    [photoId],
    (err, results) => {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ sum_nilai: results[0].sum_nilai || 0 });
    }
  );
});

// POST tambah peringkat baru
app.post("/api/peringkat", (req, res) => {
  const { photo_id, nilai } = req.body;
  db.query(
    "INSERT INTO peringkat_foto (photo_id, nilai) VALUES (?, ?)",
    [photo_id, nilai],
    err => {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ message: "Peringkat berhasil disimpan" });
    }
  );
});

app.listen(PORT, () => console.log(`🚀 Server berjalan di http://localhost:${PORT}`));
```

---

## 💻 Bagian 2: Frontend (50 poin)

### 1️⃣ Struktur Halaman (`index.html`)
```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>Galeri Foto Publik</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <h1>Galeri Foto Publik</h1>
  <div id="foto-container"></div>
  <script src="script.js"></script>
</body>
</html>
```

---

### 2️⃣ Logika & Interaktivitas (`script.js`)
```js
async function muatFoto() {
  const res = await fetch("https://jsonplaceholder.typicode.com/photos");
  const data = await res.json();
  const fotoContainer = document.getElementById("foto-container");

  data.slice(0, 20).forEach(foto => {
    const divFoto = document.createElement("div");
    divFoto.classList.add("foto-item");

    divFoto.innerHTML = `
      <img src="${foto.thumbnailUrl}" alt="${foto.title}">
      <h3>${foto.title}</h3>
      <div id="rating-${foto.id}" class="rating">Memuat...</div>
      <form id="form-${foto.id}">
        <select name="nilai">
          <option value="1">1</option><option value="2">2</option>
          <option value="3">3</option><option value="4">4</option>
          <option value="5">5</option>
        </select>
        <button type="submit">Kirim</button>
      </form>
    `;

    fotoContainer.appendChild(divFoto);

    muatPeringkatFoto(foto.id);
    const form = document.getElementById(`form-${foto.id}`);
    form.addEventListener("submit", e => kirimPeringkat(e, foto.id));
  });
}

async function muatPeringkatFoto(photoId) {
  const res = await fetch(`/api/peringkat/${photoId}`);
  const data = await res.json();
  const el = document.getElementById(`rating-${photoId}`);
  el.innerText = `Rata-rata peringkat: ${data.sum_nilai.toFixed(1)}`;
}

async function kirimPeringkat(event, photoId) {
  event.preventDefault();
  const nilai = event.target.nilai.value;
  await fetch("/api/peringkat", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ photo_id: photoId, nilai: parseInt(nilai) })
  });
  muatPeringkatFoto(photoId);
}

muatFoto();
```

---

### 3️⃣ Style Dasar (`style.css`)
```css
body { font-family: Arial, sans-serif; background: #f4f4f4; text-align: center; }
h1 { color: #333; }
.foto-item { display: inline-block; margin: 10px; background: #fff; padding: 10px; border-radius: 8px; width: 180px; }
img { border-radius: 6px; width: 100%; }
.rating { margin-top: 6px; font-weight: bold; }
```

---

## 🧪 Kriteria Penilaian
| Aspek | Bobot | Deskripsi |
|--------|--------|------------|
| Fungsionalitas Backend | 25% | Rute API berfungsi sesuai spesifikasi |
| Fungsionalitas Frontend | 25% | Data foto & rating tampil dan interaktif |
| Integrasi | 20% | Fetch API GET & POST berjalan lancar |
| Manipulasi DOM | 15% | Elemen HTML dinamis dan rapi |
| Kualitas Kode | 15% | Struktur dan sintaks terorganisir |

---

## 📚 Ringkasan
Mahasiswa akan memahami bagaimana:
- Mengambil data dari API eksternal.
- Mengirim dan membaca data dari database lokal.
- Menggabungkan backend dan frontend dalam aplikasi asinkron.
- Mengimplementasikan konsep **CRUD dasar** (read + create).

---

**Selamat mengerjakan dan berkreasi! 🚀**  
_"Integrasi API adalah seni membuat dua dunia bicara dalam bahasa yang sama."_
