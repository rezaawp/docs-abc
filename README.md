# docs-abc

### **Penjelasan Cara Kerja Query case when dengan group by**

Query ini menghitung total nilai **`brg_keluar`** berdasarkan kategori tertentu dalam kolom **`jenis`**, lalu mengelompokkan hasilnya berdasarkan **`kodebarang`**.  

---

## **1️⃣ Grouping dengan `GROUP BY`**
```sql
GROUP BY invt_kartustok.kodebarang
```
👉 **Ini berarti setiap hasil akan dikelompokkan berdasarkan `kodebarang`**.  
👉 Semua agregasi (`SUM`) akan dihitung **per kodebarang**.  

---

## **2️⃣ Bagaimana `CASE WHEN` Masih Bisa Berjalan?**
👉 **Karena `SUM(CASE WHEN ... THEN ... ELSE ... END)` adalah bentuk agregasi!**  
👉 PostgreSQL **menjalankan `CASE WHEN` di setiap baris**, kemudian `SUM()` menjumlahkannya berdasarkan grup.  

---

## **3️⃣ Cara PostgreSQL Mengeksekusi `SUM(CASE WHEN ...)`**
Lihat bagian pertama dari query:
```sql
SUM(
  CASE 
    WHEN invt_kartustok.jenis :: text = ANY (ARRAY['4', '6']) 
    THEN invt_kartustok.brg_keluar 
    ELSE 0 
  END
) AS total_brg_keluar_a
```
📌 **Langkah Eksekusi per Baris:**
- PostgreSQL memeriksa nilai **`jenis`**.  
- Jika nilai **`jenis`** ada di dalam **`ARRAY['4', '6']`**, ambil **`brg_keluar`**.  
- Jika tidak, berikan nilai `0`.  
- PostgreSQL menjumlahkan semua hasil ini **per `kodebarang`**.

Contoh data:  
| kodebarang | jenis | brg_keluar |
|------------|------|------------|
| A001       | 4    | 10         |
| A001       | 6    | 5          |
| A001       | 8    | 7          |
| A001       | 9    | 3          |
| A002       | 4    | 8          |
| A002       | 9    | 6          |

📌 **Hasil setelah `SUM(CASE WHEN ...)`**
| kodebarang | total_brg_keluar_a (4 & 6) | total_brg_keluar_b (8 & 9) |
|------------|---------------------------|---------------------------|
| A001       | 10 + 5 = **15**            | 7 + 3 = **10**           |
| A002       | 8                           | 6                         |

---

## **4️⃣ Filter Data dengan `WHERE`**
```sql
WHERE invt_kartustok.tgl_input >= (CURRENT_DATE - '30 days'::interval)
AND invt_kartustok.jenis::text = ANY (ARRAY['4', '8', '9', '6'])
```
🔹 **Membatasi hanya data dalam 30 hari terakhir**  
🔹 **Memfilter hanya `jenis` yang termasuk dalam ['4', '6', '8', '9']**  

---

### **Kesimpulan**
✔ PostgreSQL menjalankan `CASE WHEN` pada setiap **baris data** sebelum proses `GROUP BY`.  
✔ **Setelah `CASE WHEN` selesai, hasilnya dijumlahkan (`SUM()`) berdasarkan `kodebarang`**.  
✔ Hasil akhir adalah **total `brg_keluar`** yang dihitung secara terpisah berdasarkan kategori **`jenis`**.  

🚀 **Jadi, meskipun sudah `GROUP BY kodebarang`, PostgreSQL masih bisa menggunakan `CASE WHEN` untuk melakukan perhitungan agregasi yang spesifik per kategori!**
