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

### Kardinalitas Relasi (Relationship Cardinality)

#### **1. ONE-TO-ONE (1:1)**
**Definisi:** Satu entitas di tabel A berhubungan dengan **tepat satu** entitas di tabel B, dan sebaliknya.

**Contoh Kasus:** 
- 1 Karyawan memiliki 1 Laptop Kantor
- 1 Negara memiliki 1 Ibu Kota
- 1 Mahasiswa memiliki 1 Kartu Mahasiswa

**Diagram ERD:**
```
[KARYAWAN] -------- 1:1 -------- [LAPTOP]
    |                                |
  PK: ID_Karyawan                PK: ID_Laptop
    Nama                          FK: ID_Karyawan (UNIQUE)
    Jabatan                         Merk
                                    Serial_Number
```

**Implementasi SQL:**
```sql
CREATE TABLE karyawan (
    id_karyawan CHAR(6) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,
    jabatan VARCHAR2(30)
);

CREATE TABLE laptop (
    id_laptop CHAR(6) PRIMARY KEY,
    id_karyawan CHAR(6) UNIQUE NOT NULL,  -- UNIQUE untuk 1:1
    merk VARCHAR2(30),
    serial_number VARCHAR2(50),
    CONSTRAINT laptop_fk FOREIGN KEY (id_karyawan) 
        REFERENCES karyawan(id_karyawan) 
        ON DELETE CASCADE
);
```

**Ciri Khas:** Foreign Key di salah satu tabel + constraint **UNIQUE**

---

#### **2. ONE-TO-MANY (1:N)**
**Definisi:** Satu entitas di tabel A berhubungan dengan **banyak** entitas di tabel B, tapi satu entitas di B hanya berhubungan dengan satu entitas di A.

**Contoh Kasus:**
- 1 Dosen membimbing N Mahasiswa
- 1 Jurusan memiliki N Mahasiswa
- 1 Kategori memiliki N Produk
- 1 Departemen memiliki N Karyawan

**Diagram ERD:**
```
[JURUSAN] -------- 1:N -------- [MAHASISWA]
    |                                |
  PK: Kode_Jurusan              PK: NIM
    Nama_Jurusan                  Nama
    Akreditasi                    FK: Kode_Jurusan
                                    Alamat
                                    IPK
```

**Implementasi SQL:**
```sql
CREATE TABLE jurusan (
    kode_jurusan CHAR(3) PRIMARY KEY,
    nama_jurusan VARCHAR2(50) NOT NULL,
    akreditasi CHAR(1)
);

CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,
    kode_jurusan CHAR(3) NOT NULL,  -- FK tanpa UNIQUE
    alamat VARCHAR2(100),
    ipk NUMBER(3,2),
    CONSTRAINT mhs_fk_jurusan FOREIGN KEY (kode_jurusan) 
        REFERENCES jurusan(kode_jurusan) 
        ON DELETE CASCADE
);
```

**Ciri Khas:** Foreign Key di tabel **"many"** (N), **tanpa** constraint UNIQUE

---

#### **3. MANY-TO-MANY (N:M)**
**Definisi:** Satu entitas di tabel A berhubungan dengan **banyak** entitas di tabel B, dan sebaliknya.

**Contoh Kasus:**
- N Mahasiswa mengambil M Mata Kuliah
- N Aktor bermain di M Film
- N Dokter menangani M Pasien
- N Penulis menulis M Buku

**Diagram ERD:**
```
[MAHASISWA] -----< mendaftar >----- [MATA_KULIAH]
    |                                      |
  PK: NIM                              PK: Kode_MK
    Nama                                  Nama_MK
    Alamat                                SKS
                                          Semester

         [PENDAFTARAN] (Tabel Relasi/Junction Table)
              |
            PK: ID_Pendaftaran
            FK: NIM
            FK: Kode_MK
              Tanggal_Daftar
              Nilai
```

**Implementasi SQL:**
```sql
-- Tabel Entitas 1
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,
    alamat VARCHAR2(100)
);

-- Tabel Entitas 2
CREATE TABLE mata_kuliah (
    kode_mk CHAR(6) PRIMARY KEY,
    nama_mk VARCHAR2(50) NOT NULL,
    sks NUMBER(1),
    semester NUMBER(1)
);

-- Tabel Relasi/Junction/Bridge Table
CREATE TABLE pendaftaran (
    id_pendaftaran CHAR(8) PRIMARY KEY,
    nim CHAR(10) NOT NULL,
    kode_mk CHAR(6) NOT NULL,
    tanggal_daftar DATE DEFAULT SYSDATE,
    nilai NUMBER(5,2),
    CONSTRAINT pend_fk_mhs FOREIGN KEY (nim) 
        REFERENCES mahasiswa(nim) ON DELETE CASCADE,
    CONSTRAINT pend_fk_mk FOREIGN KEY (kode_mk) 
        REFERENCES mata_kuliah(kode_mk) ON DELETE CASCADE,
    CONSTRAINT pend_unique UNIQUE (nim, kode_mk)  -- Hindari duplikasi
);
```

**Ciri Khas:** Butuh **tabel ketiga** (junction/bridge table) yang berisi 2 Foreign Key

---

### Ringkasan Kardinalitas

| Kardinalitas | FK Lokasi | Constraint UNIQUE | Tabel Tambahan |
|--------------|-----------|-------------------|----------------|
| **1:1** | Salah satu tabel | ✅ YA | ❌ TIDAK |
| **1:N** | Tabel "Many" | ❌ TIDAK | ❌ TIDAK |
| **N:M** | Tabel junction | ❌ TIDAK | ✅ YA (Junction) |

---

### Contoh ERD Lengkap dengan Berbagai Kardinalitas

```
         [DOSEN] 1:1 [RUANG_KERJA]
            |
            | 1:N (membimbing)
            |
    [MAHASISWA] N:M (mengambil) [MATA_KULIAH]
            |                         |
            | 1:N                     | 1:N
            |                         |
      [JURUSAN]                   [DOSEN]
                                (mengajar)
```

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

---

### CONSTRAINTS (Batasan)

#### **PRIMARY KEY**
```sql
-- Method 1: Inline
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50)
);

-- Method 2: Table-level (RECOMMENDED)
CREATE TABLE mahasiswa (
    nim CHAR(10),
    nama VARCHAR2(50),
    CONSTRAINT mhs_pk PRIMARY KEY (nim)
);

-- Method 3: Composite Primary Key
CREATE TABLE nilai (
    nim CHAR(10),
    kode_mk CHAR(6),
    nilai NUMBER(5,2),
    CONSTRAINT nilai_pk PRIMARY KEY (nim, kode_mk)
);
```

**Aturan Primary Key:**
- ✅ Harus UNIK
- ✅ Tidak boleh NULL
- ✅ Hanya 1 per tabel
- ✅ Bisa composite (gabungan kolom)

---

#### **FOREIGN KEY** ⭐⭐⭐

**Definisi:** Kolom yang mereferensikan PRIMARY KEY/UNIQUE KEY di tabel lain untuk menjaga referential integrity.

**Sintaks Dasar:**
```sql
CONSTRAINT nama_fk FOREIGN KEY (kolom_lokal)
    REFERENCES tabel_induk(kolom_pk)
    [ON DELETE action]
    [ON UPDATE action]  -- Oracle tidak support, auto cascade
```

**ON DELETE Actions:**

| Action | Deskripsi | Contoh |
|--------|-----------|--------|
| `CASCADE` | Hapus child jika parent dihapus | Hapus mahasiswa → hapus nilai |
| `SET NULL` | Set NULL jika parent dihapus | Hapus dosen → set NULL di mahasiswa |
| `NO ACTION` | Tolak penghapusan parent (default) | Tidak bisa hapus jurusan jika ada mahasiswa |
| `SET DEFAULT` | Set ke default value | - |

**Contoh Lengkap:**

**1. ON DELETE CASCADE**
```sql
-- Parent Table
CREATE TABLE jurusan (
    kode_jurusan CHAR(3) PRIMARY KEY,
    nama_jurusan VARCHAR2(50)
);

-- Child Table
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50),
    kode_jurusan CHAR(3),
    CONSTRAINT mhs_fk_jurusan FOREIGN KEY (kode_jurusan)
        REFERENCES jurusan(kode_jurusan)
        ON DELETE CASCADE  -- Hapus mahasiswa jika jurusan dihapus
);

-- Jika: DELETE FROM jurusan WHERE kode_jurusan = 'IF';
-- Maka: Semua mahasiswa dengan kode_jurusan 'IF' juga terhapus
```

**2. ON DELETE SET NULL**
```sql
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50),
    id_dosen_wali CHAR(6),
    CONSTRAINT mhs_fk_dosen FOREIGN KEY (id_dosen_wali)
        REFERENCES dosen(id_dosen)
        ON DELETE SET NULL  -- Set NULL jika dosen dihapus
);

-- Jika: DELETE FROM dosen WHERE id_dosen = 'D001';
-- Maka: id_dosen_wali mahasiswa jadi NULL (mahasiswa tidak terhapus)
```

**3. ON DELETE NO ACTION / RESTRICT (Default)**
```sql
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50),
    kode_jurusan CHAR(3),
    CONSTRAINT mhs_fk_jurusan FOREIGN KEY (kode_jurusan)
        REFERENCES jurusan(kode_jurusan)
    -- Tidak ada ON DELETE = NO ACTION (default)
);

-- Jika: DELETE FROM jurusan WHERE kode_jurusan = 'IF';
-- Maka: ERROR! Tidak bisa hapus karena ada mahasiswa yang masih referensi
```

**4. Multiple Foreign Keys**
```sql
CREATE TABLE pendaftaran (
    id_pendaftaran CHAR(8) PRIMARY KEY,
    nim CHAR(10) NOT NULL,
    kode_mk CHAR(6) NOT NULL,
    id_dosen CHAR(6),
    semester NUMBER(1),
    tahun_ajaran VARCHAR2(9),
    nilai NUMBER(5,2),
    
    -- FK ke Mahasiswa
    CONSTRAINT pend_fk_mhs FOREIGN KEY (nim)
        REFERENCES mahasiswa(nim) ON DELETE CASCADE,
    
    -- FK ke Mata Kuliah
    CONSTRAINT pend_fk_mk FOREIGN KEY (kode_mk)
        REFERENCES mata_kuliah(kode_mk) ON DELETE CASCADE,
    
    -- FK ke Dosen
    CONSTRAINT pend_fk_dosen FOREIGN KEY (id_dosen)
        REFERENCES dosen(id_dosen) ON DELETE SET NULL
);
```

**5. Self-Referencing FK (Recursive)**
```sql
CREATE TABLE pegawai (
    id_pegawai CHAR(6) PRIMARY KEY,
    nama VARCHAR2(50),
    id_atasan CHAR(6),  -- Referensi ke pegawai lain
    CONSTRAINT peg_fk_atasan FOREIGN KEY (id_atasan)
        REFERENCES pegawai(id_pegawai)
        ON DELETE SET NULL
);
```

**Cara Menambah FK ke Tabel Existing:**
```sql
ALTER TABLE mahasiswa
ADD CONSTRAINT mhs_fk_jurusan FOREIGN KEY (kode_jurusan)
    REFERENCES jurusan(kode_jurusan) ON DELETE CASCADE;
```

**Cara Hapus FK:**
```sql
ALTER TABLE mahasiswa DROP CONSTRAINT mhs_fk_jurusan;
```

**Enable/Disable FK:**
```sql
-- Disable (untuk bulk insert cepat)
ALTER TABLE mahasiswa DISABLE CONSTRAINT mhs_fk_jurusan;

-- Enable kembali
ALTER TABLE mahasiswa ENABLE CONSTRAINT mhs_fk_jurusan;
```

---

#### **UNIQUE Constraint**
```sql
-- Inline
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    email VARCHAR2(50) UNIQUE
);

-- Table-level
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    email VARCHAR2(50),
    no_telp VARCHAR2(15),
    CONSTRAINT mhs_uk_email UNIQUE (email),
    CONSTRAINT mhs_uk_telp UNIQUE (no_telp)
);
```

**Perbedaan UNIQUE vs PRIMARY KEY:**
| | PRIMARY KEY | UNIQUE |
|---|-------------|---------|
| NULL | ❌ Tidak boleh | ✅ Boleh (1x) |
| Jumlah | 1 per tabel | Banyak per tabel |
| Tujuan | Identifikasi baris | Hindari duplikasi |

---

#### **NOT NULL Constraint**
```sql
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,  -- Wajib diisi
    email VARCHAR2(50)           -- Boleh kosong
);
```

---

#### **CHECK Constraint**
```sql
CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,
    ipk NUMBER(3,2),
    semester NUMBER(2),
    
    -- Check constraints
    CONSTRAINT mhs_ck_ipk CHECK (ipk BETWEEN 0 AND 4.0),
    CONSTRAINT mhs_ck_semester CHECK (semester >= 1 AND semester <= 14)
);

-- Contoh lain
CREATE TABLE produk (
    id_produk CHAR(6) PRIMARY KEY,
    nama_produk VARCHAR2(50),
    harga NUMBER(10,2),
    stok NUMBER(5),
    
    CONSTRAINT prd_ck_harga CHECK (harga > 0),
    CONSTRAINT prd_ck_stok CHECK (stok >= 0)
);
```

---

#### **DEFAULT Constraint**
```sql
CREATE TABLE transaksi (
    id_transaksi CHAR(8) PRIMARY KEY,
    tanggal DATE DEFAULT SYSDATE,
    status VARCHAR2(20) DEFAULT 'PENDING',
    diskon NUMBER(5,2) DEFAULT 0
);
```

---

### Contoh Tabel dengan Semua Constraints

```sql
CREATE TABLE mahasiswa (
    nim CHAR(10),
    nama VARCHAR2(50) NOT NULL,
    email VARCHAR2(50),
    no_telp VARCHAR2(15),
    kode_jurusan CHAR(3) NOT NULL,
    id_dosen_wali CHAR(6),
    ipk NUMBER(3,2) DEFAULT 0,
    semester NUMBER(2) DEFAULT 1,
    tanggal_daftar DATE DEFAULT SYSDATE,
    status VARCHAR2(10) DEFAULT 'AKTIF',
    
    -- Primary Key
    CONSTRAINT mhs_pk PRIMARY KEY (nim),
    
    -- Foreign Keys
    CONSTRAINT mhs_fk_jurusan FOREIGN KEY (kode_jurusan)
        REFERENCES jurusan(kode_jurusan) ON DELETE CASCADE,
    CONSTRAINT mhs_fk_dosen FOREIGN KEY (id_dosen_wali)
        REFERENCES dosen(id_dosen) ON DELETE SET NULL,
    
    -- Unique
    CONSTRAINT mhs_uk_email UNIQUE (email),
    CONSTRAINT mhs_uk_telp UNIQUE (no_telp),
    
    -- Check
    CONSTRAINT mhs_ck_ipk CHECK (ipk BETWEEN 0 AND 4.0),
    CONSTRAINT mhs_ck_semester CHECK (semester BETWEEN 1 AND 14),
    CONSTRAINT mhs_ck_status CHECK (status IN ('AKTIF', 'CUTI', 'LULUS', 'DO'))
);
```

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

### Kardinalitas & Foreign Key - Visual Guide

```
1:1 → FK di salah satu tabel + UNIQUE
┌─────────┐         ┌──────────┐
│ PEGAWAI │────1:1──│  LAPTOP  │
│  (PK)   │         │  (PK,FK) │ ← FK + UNIQUE
└─────────┘         └──────────┘

1:N → FK di tabel "Many" (N)
┌─────────┐         ┌───────────┐
│ JURUSAN │────1:N──│ MAHASISWA │
│  (PK)   │         │  (PK,FK)  │ ← FK tanpa UNIQUE
└─────────┘         └───────────┘
    1                    MANY

N:M → Butuh tabel junction dengan 2 FK
┌───────────┐         ┌─────────────┐         ┌──────────┐
│ MAHASISWA │────N────│ PENDAFTARAN │────M────│ MATA_MK  │
│   (PK)    │         │  (PK,FK,FK) │         │   (PK)   │
└───────────┘         └─────────────┘         └──────────┘
                       ↑ Junction Table
                       berisi 2 FK
```

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

### Soal 1: ERD & Kardinalitas
Buatlah ERD untuk sistem perpustakaan dengan entitas:
- Anggota (ID, Nama, Alamat, Telp)
- Buku (ISBN, Judul, Pengarang, Tahun)
- Peminjaman (tanggal_pinjam, tanggal_kembali)

**Relasi:** 
- 1 Anggota bisa pinjam banyak Buku (1:N)
- 1 Buku bisa dipinjam berkali-kali (N:M dengan Anggota)

<details>
<summary>Jawaban</summary>

```
[ANGGOTA] ────N:M──── [BUKU]
    |                    |
  PK: ID              PK: ISBN
                        
        [PEMINJAMAN] (Junction Table)
              |
          PK: ID_Pinjam
          FK: ID_Anggota
          FK: ISBN
            tanggal_pinjam
            tanggal_kembali
```

**Kardinalitas:** N:M → Butuh tabel junction (PEMINJAMAN)
</details>

---

### Soal 2: Foreign Key Implementation
Buat DDL untuk kasus berikut:
- Tabel DOSEN (id_dosen PK, nama, nip UNIQUE)
- Tabel MAHASISWA (nim PK, nama, id_dosen_wali FK)
- Jika dosen dihapus, id_dosen_wali mahasiswa jadi NULL

<details>
<summary>Jawaban</summary>

```sql
CREATE TABLE dosen (
    id_dosen CHAR(6) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,
    nip CHAR(18),
    CONSTRAINT dosen_uk_nip UNIQUE (nip)
);

CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    nama VARCHAR2(50) NOT NULL,
    id_dosen_wali CHAR(6),
    CONSTRAINT mhs_fk_dosen FOREIGN KEY (id_dosen_wali)
        REFERENCES dosen(id_dosen)
        ON DELETE SET NULL  -- ← Key answer
);
```
</details>

---

### Soal 3: Query Join
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

---

### Soal 4: Subquery
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

### Soal 5: Kardinalitas Challenge
Identifikasi kardinalitas untuk kasus berikut:

1. **1 Pasien bisa konsultasi dengan banyak Dokter, 1 Dokter menangani banyak Pasien**
   <details><summary>Jawaban</summary>N:M → Butuh tabel KONSULTASI</details>

2. **1 Karyawan punya 1 Badge ID, 1 Badge hanya untuk 1 Karyawan**
   <details><summary>Jawaban</summary>1:1 → FK + UNIQUE di salah satu tabel</details>

3. **1 Kategori punya banyak Produk, 1 Produk hanya 1 Kategori**
   <details><summary>Jawaban</summary>1:N → FK di tabel Produk</details>

4. **1 Siswa bisa ikut banyak Ekskul, 1 Ekskul punya banyak Siswa**
   <details><summary>Jawaban</summary>N:M → Butuh tabel PENDAFTARAN_EKSKUL</details>

---

### Soal 6: Complex Query
Buat query untuk kasus:
- Tampilkan nama jurusan dan jumlah mahasiswa per jurusan
- Hanya jurusan dengan > 10 mahasiswa
- Urutkan dari yang terbanyak

<details>
<summary>Jawaban</summary>

```sql
SELECT j.nama_jurusan, COUNT(m.nim) AS jumlah_mhs
FROM jurusan j
LEFT JOIN mahasiswa m ON j.kode_jurusan = m.kode_jurusan
GROUP BY j.nama_jurusan
HAVING COUNT(m.nim) > 10
ORDER BY jumlah_mhs DESC;
```
</details>

---

### Soal 7: ON DELETE Action
Apa yang terjadi pada data child jika parent dihapus?

**Skenario:**
```sql
CREATE TABLE jurusan (
    kode_jurusan CHAR(3) PRIMARY KEY,
    nama_jurusan VARCHAR2(50)
);

CREATE TABLE mahasiswa (
    nim CHAR(10) PRIMARY KEY,
    kode_jurusan CHAR(3),
    CONSTRAINT mhs_fk FOREIGN KEY (kode_jurusan)
        REFERENCES jurusan(kode_jurusan)
        ON DELETE CASCADE
);
```

**Jika dijalankan:**
```sql
DELETE FROM jurusan WHERE kode_jurusan = 'IF';
```

<details>
<summary>Jawaban</summary>

Semua mahasiswa dengan `kode_jurusan = 'IF'` akan **terhapus** juga karena `ON DELETE CASCADE`.

Jika ingin mahasiswa tidak terhapus, gunakan:
- `ON DELETE SET NULL` → kode_jurusan jadi NULL
- Atau tidak pakai ON DELETE → Error, tidak bisa hapus jurusan
</details>

---

**SEMOGA SUKSES! 🚀**

*Catatan: Cheatsheet ini berdasarkan Oracle 11g. Beberapa sintaks mungkin berbeda di DBMS lain.*
