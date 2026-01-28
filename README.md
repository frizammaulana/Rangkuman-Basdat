# RANGKUMAN BASIS DATA

## 📋 DAFTAR ISI
1. [ERD (Entity Relationship Diagram)](#1-erd-entity-relationship-diagram)
2. [SQL Query Basic](#2-sql-query-basic)
3. [SQL Query Advanced](#3-sql-query-advanced)
4. [Subquery](#4-subquery)
5. [Tips & Tricks](#5-tips--tricks)

---

## 1. ERD (Entity Relationship Diagram)

### Komponen Utama ERD

#### **Entity (Entitas)**
- Representasi objek/konsep dalam sistem
- Digambarkan dengan **persegi panjang**
- Contoh: Mahasiswa, Dosen, Mata Kuliah

#### **Attribute (Atribut)**
- Karakteristik dari entitas
- Digambarkan dengan **oval/elips**
- **Jenis-jenis Atribut:**
  - **Primary Key (PK)**: Atribut unik, digambarkan dengan **oval + garis bawah**
  - **Composite**: Atribut yang terdiri dari beberapa atribut (contoh: Nama → Nama_Depan, Nama_Belakang)
  - **Multivalued**: Atribut dengan banyak nilai, digambarkan dengan **oval ganda**
  - **Derived**: Atribut yang dihitung dari atribut lain, digambarkan dengan **oval putus-putus**

#### **Relationship (Relasi)**
- Hubungan antar entitas
- Digambarkan dengan **belah ketupat (diamond)**
- **Kardinalitas:**
  - **1:1** (One-to-One)
  - **1:N** (One-to-Many) 
  - **N:M** (Many-to-Many)

#### **Participation Constraint**
- **Total Participation** (garis ganda): Setiap instance harus berpartisipasi
- **Partial Participation** (garis tunggal): Boleh tidak berpartisipasi

### Contoh ERD Sederhana

```
[MAHASISWA] -----< mendaftar >----- [MATA_KULIAH]
    |                                      |
  PK: NIM                              PK: Kode_MK
    Nama                                  Nama_MK
    Alamat                                SKS
    Email                                 Semester
```

**Kardinalitas:** 1 Mahasiswa bisa mendaftar N Mata Kuliah (1:N)

---

## 2. SQL Query Basic

### DDL (Data Definition Language)

#### **CREATE TABLE**
```sql
CREATE TABLE nama_tabel (
    kolom1 TIPE_DATA [CONSTRAINT],
    kolom2 TIPE_DATA [CONSTRAINT],
    CONSTRAINT nama_pk PRIMARY KEY (kolom1),
    CONSTRAINT nama_fk FOREIGN KEY (kolom2) 
        REFERENCES tabel_lain(kolom_pk) ON DELETE CASCADE
);
```

**Contoh:**
```sql
CREATE TABLE mahasiswa (
    nim CHAR(10) NOT NULL,
    nama VARCHAR2(50) NOT NULL,
    tgl_lahir DATE,
    ipk NUMBER(3,2),
    CONSTRAINT mhs_pk PRIMARY KEY (nim)
);
```

#### **Tipe Data Oracle**
| Tipe Data | Deskripsi | Contoh |
|-----------|-----------|--------|
| `CHAR(n)` | String panjang tetap | `CHAR(10)` |
| `VARCHAR2(n)` | String panjang variabel | `VARCHAR2(100)` |
| `NUMBER(p,s)` | Angka dengan presisi p, skala s | `NUMBER(10,2)` |
| `DATE` | Tanggal & waktu | `01-JAN-2024` |
| `TIMESTAMP` | Tanggal & waktu detail | - |
| `CLOB` | Text besar (4GB) | - |
| `BLOB` | Binary besar (4GB) | - |

#### **ALTER TABLE**
```sql
-- Tambah kolom
ALTER TABLE mahasiswa ADD email VARCHAR2(50);

-- Ubah kolom
ALTER TABLE mahasiswa MODIFY email VARCHAR2(100);

-- Hapus kolom
ALTER TABLE mahasiswa DROP COLUMN email;

-- Tambah constraint
ALTER TABLE mahasiswa ADD CONSTRAINT mhs_ck CHECK (ipk <= 4.0);
```

#### **DROP & TRUNCATE**
```sql
DROP TABLE mahasiswa CASCADE CONSTRAINT;  -- Hapus tabel
TRUNCATE TABLE mahasiswa;                 -- Hapus semua data
```

---

### DML (Data Manipulation Language)

#### **INSERT**
```sql
-- Insert single row
INSERT INTO mahasiswa (nim, nama, ipk) 
VALUES ('1234567890', 'Budi', 3.5);

-- Insert multiple rows
INSERT INTO mahasiswa 
SELECT * FROM mahasiswa_backup;
```

#### **UPDATE**
```sql
UPDATE mahasiswa 
SET ipk = 3.75, email = 'budi@email.com'
WHERE nim = '1234567890';
```

#### **DELETE**
```sql
DELETE FROM mahasiswa 
WHERE ipk < 2.0;
```

#### **MERGE** (Upsert)
```sql
MERGE INTO mahasiswa m
USING mahasiswa_baru mb
ON (m.nim = mb.nim)
WHEN MATCHED THEN
    UPDATE SET m.nama = mb.nama
WHEN NOT MATCHED THEN
    INSERT (nim, nama) VALUES (mb.nim, mb.nama);
```

---

### SELECT Query (DQL)

#### **Sintaks Dasar**
```sql
SELECT [DISTINCT] kolom1, kolom2, ...
FROM tabel
WHERE kondisi
GROUP BY kolom
HAVING kondisi_agregat
ORDER BY kolom [ASC|DESC];
```

#### **Operator WHERE**
```sql
=, !=, <>, >, <, >=, <=          -- Perbandingan
AND, OR, NOT                      -- Logika
BETWEEN x AND y                   -- Range
IN (nilai1, nilai2, ...)          -- List
LIKE 'pattern'                    -- Pattern matching
IS NULL, IS NOT NULL              -- Null check
```

#### **LIKE Pattern**
```sql
'A%'     -- Dimulai dengan A
'%A'     -- Diakhiri dengan A
'%A%'    -- Mengandung A
'A_'     -- A + 1 karakter
'____'   -- Tepat 4 karakter
```

#### **Fungsi String**
```sql
CONCAT(str1, str2)           -- Gabung string
LENGTH(str)                  -- Panjang string
SUBSTR(str, pos, len)        -- Potong string
UPPER(str), LOWER(str)       -- Ubah case
TRIM(str)                    -- Hapus spasi
REPLACE(str, old, new)       -- Ganti string
LPAD(str, len, 'x')          -- Padding kiri
RPAD(str, len, 'x')          -- Padding kanan
```

#### **Fungsi Numerik**
```sql
ROUND(n, d)      -- Pembulatan
TRUNC(n, d)      -- Pemotongan
CEIL(n)          -- Pembulatan ke atas
FLOOR(n)         -- Pembulatan ke bawah
MOD(m, n)        -- Modulo
ABS(n)           -- Nilai absolut
POWER(m, n)      -- Pangkat
SQRT(n)          -- Akar kuadrat
```

#### **Fungsi Tanggal**
```sql
SYSDATE                          -- Tanggal sekarang
ADD_MONTHS(date, n)              -- Tambah bulan
MONTHS_BETWEEN(date1, date2)     -- Selisih bulan
LAST_DAY(date)                   -- Hari terakhir bulan
NEXT_DAY(date, 'DAY')            -- Hari berikutnya
TO_CHAR(date, 'format')          -- Date ke string
TO_DATE(str, 'format')           -- String ke date
```

**Format Tanggal:**
```sql
'DD-MON-YYYY'    -- 01-JAN-2024
'DD/MM/YYYY'     -- 01/01/2024
'YYYY-MM-DD'     -- 2024-01-01
```

#### **Fungsi Agregat**
```sql
COUNT(*)             -- Hitung baris
COUNT(DISTINCT x)    -- Hitung nilai unik
SUM(kolom)          -- Total
AVG(kolom)          -- Rata-rata
MAX(kolom)          -- Maksimum
MIN(kolom)          -- Minimum
```

#### **GROUP BY & HAVING**
```sql
SELECT jurusan, COUNT(*) AS jumlah, AVG(ipk) AS rata_ipk
FROM mahasiswa
GROUP BY jurusan
HAVING AVG(ipk) > 3.0
ORDER BY rata_ipk DESC;
```

**ATURAN PENTING:**
- Kolom di SELECT (non-agregat) **HARUS** ada di GROUP BY
- WHERE filter sebelum GROUP BY
- HAVING filter setelah GROUP BY
- ORDER BY paling akhir

---

## 3. SQL Query Advanced

### JOIN Operations

#### **CROSS JOIN** (Cartesian Product)
```sql
SELECT * FROM tabel1 CROSS JOIN tabel2;
-- ATAU
SELECT * FROM tabel1, tabel2;
```

#### **INNER JOIN**

**Natural Join:**
```sql
SELECT * 
FROM mahasiswa NATURAL JOIN jurusan;
-- Otomatis join kolom dengan nama sama
```

**JOIN USING:**
```sql
SELECT * 
FROM mahasiswa JOIN jurusan USING(kode_jurusan);
-- Spesifik kolom join
```

**JOIN ON:**
```sql
SELECT m.nim, m.nama, j.nama_jurusan
FROM mahasiswa m JOIN jurusan j 
ON m.kode_jurusan = j.kode_jurusan;
-- Paling fleksibel
```

#### **OUTER JOIN**

**LEFT OUTER JOIN:**
```sql
SELECT m.*, j.nama_jurusan
FROM mahasiswa m 
LEFT OUTER JOIN jurusan j ON m.kode_jurusan = j.kode_jurusan;
-- Tampilkan semua mahasiswa, NULL jika tidak ada jurusan
```

**RIGHT OUTER JOIN:**
```sql
SELECT m.*, j.nama_jurusan
FROM mahasiswa m 
RIGHT OUTER JOIN jurusan j ON m.kode_jurusan = j.kode_jurusan;
-- Tampilkan semua jurusan, NULL jika tidak ada mahasiswa
```

**FULL OUTER JOIN:**
```sql
SELECT m.*, j.nama_jurusan
FROM mahasiswa m 
FULL OUTER JOIN jurusan j ON m.kode_jurusan = j.kode_jurusan;
-- Tampilkan semua dari kedua tabel
```

#### **SELF JOIN**
```sql
SELECT e1.nama AS karyawan, e2.nama AS manager
FROM pegawai e1 JOIN pegawai e2 
ON e1.id_manager = e2.id_pegawai;
```

#### **Multiple JOIN**
```sql
SELECT m.nama, mk.nama_mk, n.nilai
FROM mahasiswa m
JOIN pendaftaran p ON m.nim = p.nim
JOIN mata_kuliah mk ON p.kode_mk = mk.kode_mk
JOIN nilai n ON p.id_pendaftaran = n.id_pendaftaran
WHERE n.nilai >= 80;
```

---

## 4. SUBQUERY

### Jenis Subquery

#### **Single-Row Subquery**
Mengembalikan **1 baris**

**Operator:** `=, !=, <>, >, <, >=, <=`

```sql
-- Cari mahasiswa dengan IPK tertinggi
SELECT nama, ipk
FROM mahasiswa
WHERE ipk = (SELECT MAX(ipk) FROM mahasiswa);

-- Cari mahasiswa dengan IPK > rata-rata
SELECT nama, ipk
FROM mahasiswa
WHERE ipk > (SELECT AVG(ipk) FROM mahasiswa);
```

#### **Multiple-Row Subquery**
Mengembalikan **> 1 baris**

**Operator:** `IN, ANY, ALL`

**IN:**
```sql
-- Mahasiswa yang ambil mata kuliah tertentu
SELECT nama
FROM mahasiswa
WHERE nim IN (
    SELECT nim 
    FROM pendaftaran 
    WHERE kode_mk IN ('CS101', 'CS102')
);
```

**ANY:**
```sql
-- Mahasiswa dengan IPK > IPK minimal jurusan IF
SELECT nama, ipk
FROM mahasiswa
WHERE ipk > ANY (
    SELECT ipk 
    FROM mahasiswa 
    WHERE jurusan = 'IF'
);
-- > ANY = lebih besar dari nilai TERKECIL
```

**ALL:**
```sql
-- Mahasiswa dengan IPK > semua IPK jurusan IF
SELECT nama, ipk
FROM mahasiswa
WHERE ipk > ALL (
    SELECT ipk 
    FROM mahasiswa 
    WHERE jurusan = 'IF'
);
-- > ALL = lebih besar dari nilai TERBESAR
```

#### **Multiple-Column Subquery**
```sql
SELECT nim, nama
FROM mahasiswa
WHERE (jurusan, ipk) IN (
    SELECT jurusan, MAX(ipk)
    FROM mahasiswa
    GROUP BY jurusan
);
-- Mahasiswa dengan IPK tertinggi per jurusan
```

#### **Correlated Subquery**
Subquery yang **bergantung** pada outer query

```sql
-- Mahasiswa dengan IPK > rata-rata jurusannya
SELECT m1.nama, m1.ipk, m1.jurusan
FROM mahasiswa m1
WHERE m1.ipk > (
    SELECT AVG(m2.ipk)
    FROM mahasiswa m2
    WHERE m2.jurusan = m1.jurusan
);
```

#### **Subquery di FROM (Inline View)**
```sql
SELECT j.nama_jurusan, stats.rata_ipk
FROM jurusan j
JOIN (
    SELECT jurusan, AVG(ipk) AS rata_ipk
    FROM mahasiswa
    GROUP BY jurusan
) stats ON j.kode_jurusan = stats.jurusan
WHERE stats.rata_ipk > 3.0;
```

#### **Subquery di SELECT**
```sql
SELECT 
    nama,
    ipk,
    (SELECT AVG(ipk) FROM mahasiswa) AS rata_ipk_kampus,
    ipk - (SELECT AVG(ipk) FROM mahasiswa) AS selisih
FROM mahasiswa;
```

### Subquery dalam DML

#### **INSERT dengan Subquery**
```sql
INSERT INTO mahasiswa_berprestasi
SELECT nim, nama, ipk
FROM mahasiswa
WHERE ipk >= 3.5;
```

#### **UPDATE dengan Subquery**
```sql
UPDATE mahasiswa
SET ipk = (
    SELECT AVG(nilai)
    FROM nilai
    WHERE nilai.nim = mahasiswa.nim
)
WHERE nim IN (SELECT nim FROM nilai);
```

#### **DELETE dengan Subquery**
```sql
DELETE FROM mahasiswa
WHERE ipk < (
    SELECT AVG(ipk) 
    FROM mahasiswa
);
```

---

## 5. TIPS & TRICKS

### Checklist Query
1. ✅ Identifikasi tabel yang dibutuhkan
2. ✅ Tentukan jenis JOIN (jika > 1 tabel)
3. ✅ Pilih kolom yang ditampilkan
4. ✅ Tambahkan kondisi WHERE
5. ✅ Gunakan GROUP BY jika ada agregat
6. ✅ Filter agregat dengan HAVING
7. ✅ Urutkan dengan ORDER BY

### Kesalahan Umum

❌ **GROUP BY tanpa agregat di SELECT**
```sql
-- SALAH
SELECT jurusan, nama, COUNT(*)
FROM mahasiswa
GROUP BY jurusan;

-- BENAR
SELECT jurusan, COUNT(*)
FROM mahasiswa
GROUP BY jurusan;
```

❌ **WHERE untuk fungsi agregat**
```sql
-- SALAH
SELECT jurusan
FROM mahasiswa
WHERE AVG(ipk) > 3.0
GROUP BY jurusan;

-- BENAR
SELECT jurusan
FROM mahasiswa
GROUP BY jurusan
HAVING AVG(ipk) > 3.0;
```

❌ **Subquery mengembalikan > 1 row dengan operator `=`**
```sql
-- SALAH
SELECT nama
FROM mahasiswa
WHERE ipk = (SELECT ipk FROM mahasiswa WHERE jurusan = 'IF');

-- BENAR
SELECT nama
FROM mahasiswa
WHERE ipk IN (SELECT ipk FROM mahasiswa WHERE jurusan = 'IF');
```

### Query Performance Tips

1. **Gunakan INDEX** pada kolom yang sering di-WHERE/JOIN
   ```sql
   CREATE INDEX idx_nama ON mahasiswa(nama);
   ```

2. **Hindari SELECT *** jika tidak perlu semua kolom

3. **Gunakan EXISTS** daripada IN untuk subquery besar
   ```sql
   SELECT nama
   FROM mahasiswa m
   WHERE EXISTS (
       SELECT 1 
       FROM pendaftaran p 
       WHERE p.nim = m.nim
   );
   ```

4. **Batasi hasil dengan ROWNUM**
   ```sql
   SELECT * FROM mahasiswa WHERE ROWNUM <= 10;
   ```

### Contoh Kasus Lengkap

**Soal:** Tampilkan nama mahasiswa, jurusan, dan IPK yang:
- IPK di atas rata-rata jurusannya
- Terdaftar di lebih dari 3 mata kuliah
- Urutkan berdasarkan IPK tertinggi

```sql
SELECT 
    m.nama,
    j.nama_jurusan,
    m.ipk,
    COUNT(p.kode_mk) AS jumlah_mk
FROM mahasiswa m
JOIN jurusan j ON m.kode_jurusan = j.kode_jurusan
JOIN pendaftaran p ON m.nim = p.nim
WHERE m.ipk > (
    SELECT AVG(ipk)
    FROM mahasiswa
    WHERE kode_jurusan = m.kode_jurusan
)
GROUP BY m.nama, j.nama_jurusan, m.ipk
HAVING COUNT(p.kode_mk) > 3
ORDER BY m.ipk DESC;
```

---

## 🎯 RANGKUMAN AKHIR

### ERD
- **Entitas** = Persegi panjang
- **Atribut** = Oval (PK = garis bawah)
- **Relasi** = Belah ketupat
- **Kardinalitas** = 1:1, 1:N, N:M

### Query
- **DDL**: CREATE, ALTER, DROP, TRUNCATE
- **DML**: INSERT, UPDATE, DELETE, MERGE
- **DQL**: SELECT dengan WHERE, GROUP BY, HAVING, ORDER BY
- **JOIN**: INNER, LEFT, RIGHT, FULL, CROSS, SELF

### Subquery
- **Single-row**: `=, >, <, >=, <=`
- **Multiple-row**: `IN, ANY, ALL`
- **Correlated**: Bergantung outer query
- **Inline View**: Subquery di FROM
- Bisa di: SELECT, FROM, WHERE, HAVING, DML

### Formula Sukses
1. **Baca soal 2x** sebelum coding
2. **Gambar ERD** untuk soal kompleks
3. **Tulis pseudocode** SQL sebelum query
4. **Test step-by-step** (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY)
5. **Cek hasil** dengan ROWNUM atau COUNT

---

## 📚 LATIHAN SOAL

### Soal 1: ERD
Buatlah ERD untuk sistem perpustakaan dengan entitas:
- Anggota (ID, Nama, Alamat, Telp)
- Buku (ISBN, Judul, Pengarang, Tahun)
- Peminjaman (tanggal_pinjam, tanggal_kembali)
- Relasi: 1 Anggota bisa pinjam N Buku, 1 Buku bisa dipinjam N kali

### Soal 2: Query Join
```sql
-- Tampilkan nama mahasiswa dan jumlah mata kuliah yang diambil
-- hanya yang ambil > 5 mata kuliah
```

<details>
<summary>Jawaban</summary>

```sql
SELECT m.nama, COUNT(p.kode_mk) AS jumlah_mk
FROM mahasiswa m
JOIN pendaftaran p ON m.nim = p.nim
GROUP BY m.nama
HAVING COUNT(p.kode_mk) > 5;
```
</details>

### Soal 3: Subquery
```sql
-- Tampilkan mahasiswa yang IPK-nya lebih tinggi 
-- dari rata-rata IPK seluruh mahasiswa
```

<details>
<summary>Jawaban</summary>

```sql
SELECT nim, nama, ipk
FROM mahasiswa
WHERE ipk > (SELECT AVG(ipk) FROM mahasiswa);
```
</details>

---

**SEMOGA SUKSES! 🚀**

*Catatan: Cheatsheet ini berdasarkan Oracle 11g. Beberapa sintaks mungkin berbeda di DBMS lain.*
