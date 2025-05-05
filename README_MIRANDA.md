
## 🗃 Kalkulator Android – Storyboard Database Developer

Dokumentasi ini menjelaskan peran, struktur, dan alur kerja modul database dalam aplikasi kalkulator Android. Modul ini digunakan untuk *menyimpan, membaca, dan menghapus histori kalkulasi* secara lokal menggunakan SQLite.

---

### 📖 *Storyboard Fungsional Database*

#### 1. *Inisialisasi Database*

*Komponen Terkait:*

* DatabaseHelper.kt (extends SQLiteOpenHelper)

*Kode Utama:*

kotlin
class DatabaseHelper(context: Context) :
    SQLiteOpenHelper(context, "CalculatorDB", null, 1) {

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(
            "CREATE TABLE history (" +
            "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
            "expression TEXT, " +
            "result TEXT)"
        )
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        db.execSQL("DROP TABLE IF EXISTS history")
        onCreate(db)
    }
}


📌 Tujuan: Membuat tabel history saat database pertama kali diakses.

---

#### 2. *Fungsi Tambah Riwayat*

*Metode:* addHistoryItem(expression: String, result: String)

kotlin
fun addHistoryItem(expression: String, result: String) {
    val db = writableDatabase
    val values = ContentValues().apply {
        put("expression", expression)
        put("result", result)
    }
    db.insert("history", null, values)
    db.close()
}


📌 Tujuan: Menyimpan ekspresi dan hasil kalkulasi ke dalam database.

---

#### 3. *Fungsi Ambil Semua Riwayat*

*Metode:* getAllHistory(): List<HistoryItem>

kotlin
fun getAllHistory(): List<HistoryItem> {
    val list = mutableListOf<HistoryItem>()
    val db = readableDatabase
    val cursor = db.rawQuery("SELECT * FROM history ORDER BY id DESC", null)

    if (cursor.moveToFirst()) {
        do {
            val item = HistoryItem(
                id = cursor.getInt(0),
                expression = cursor.getString(1),
                result = cursor.getString(2)
            )
            list.add(item)
        } while (cursor.moveToNext())
    }

    cursor.close()
    db.close()
    return list
}


📌 Tujuan: Menampilkan daftar riwayat kalkulasi terbaru ke tampilan pengguna.

---

#### 4. *Fungsi Hapus Satu Riwayat*

*Metode:* deleteHistoryItem(id: Int)

kotlin
fun deleteHistoryItem(id: Int) {
    val db = writableDatabase
    db.delete("history", "id = ?", arrayOf(id.toString()))
    db.close()
}


📌 Tujuan: Menghapus riwayat kalkulasi tertentu berdasarkan ID.

---

#### 5. *Fungsi Hapus Semua Riwayat*

*Metode:* clearAllHistory()

kotlin
fun clearAllHistory() {
    val db = writableDatabase
    db.delete("history", null, null)
    db.close()
}


📌 Tujuan: Menghapus seluruh histori kalkulasi pengguna.

---

### 🔄 *Alur Logika Database*


Evaluasi selesai → Simpan ke DB → Ambil semua data → Tampilkan di RecyclerView
Tombol hapus item → deleteHistoryItem(id)
Tombol "Clear All" → clearAllHistory()


---

### 🧱 Struktur Tabel Database

| Kolom      | Tipe Data | Keterangan                  |
| ---------- | --------- | --------------------------- |
| id         | INTEGER   | Primary Key, auto increment |
| expression | TEXT      | Ekspresi yang dihitung      |
| result     | TEXT      | Hasil dari ekspresi         |

---

### ⚙ Tips Pengembangan

* Gunakan try-catch untuk menghindari crash saat query database.
* Pastikan Cursor selalu ditutup setelah digunakan.
* Jangan simpan data sensitif — riwayat hanya menyimpan ekspresi dan hasil kalkulasi.
* Batasi jumlah entri jika diperlukan (misalnya hanya tampilkan 50 entri terbaru).

---

### 📦 Integrasi dengan Komponen Lain

* *Adapter*: HistoryAdapter mengambil data dari getAllHistory().
* *View*: RecyclerView menampilkan ekspresi dan hasil.
* *Event*: onClickListener tombol "=" → memanggil addHistoryItem().

---
