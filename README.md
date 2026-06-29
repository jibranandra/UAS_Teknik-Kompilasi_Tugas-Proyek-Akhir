# UAS_Teknik-Kompilasi_Tugas-Proyek-Akhir
---
# Dokumentasi Proyek Akhir: Representasi Tahapan Kompilasi

## 1. Pilihan Konstruksi

**Konstruksi yang dipilih:** Perulangan (*Looping*: `while`)

Konstruksi `while` dipilih karena strukturnya membutuhkan penanganan *label* yang menarik pada tahap *Three-Address Code* (TAC), di mana program harus melompat kembali ke awal untuk mengevaluasi kondisi, dan melompat ke akhir blok jika kondisi bernilai *false*.

## 2. *Pattern* (Pola Sintaks)

Pola tata bahasa didefinisikan menggunakan pendekatan *Backus-Naur Form* (BNF) sederhana:

```text
<while_stmt> ::= "while" "(" <condition> ")" "{" <statements> "}"
<condition>  ::= <identifier> <operator> <value>
<operator>   ::= "<" | ">" | "==" | "!=" | "<=" | ">="
<statements> ::= <assignment> | <assignment> <statements>
<assignment> ::= <identifier> "=" <expression> ";"
<expression> ::= <identifier> | <value> | <identifier> <arithmetic_op> <value>

```

## 3. Implementasi Program

Program di bawah ini diimplementasikan menggunakan bahasa **Python**. Program ini menyimulasikan proses kompilasi mulai dari memecah *token* (Leksikal), memvalidasi struktur dan variabel (Sintaksis & Semantik), hingga menghasilkan kode antara (TAC).

```python
class WhileLoopCompiler:
    def __init__(self, source_code):
        self.source_code = source_code
        self.label_counter = 1
        # Tabel simbol sederhana (Mock Symbol Table) untuk Analisis Semantik
        self.symbol_table = {"x": "int", "y": "int", "total": "int"}

    def new_label(self):
        """Menghasilkan label baru untuk TAC."""
        lbl = f"L{self.label_counter}"
        self.label_counter += 1
        return lbl

    def lexical_analysis(self):
        """Tahap 1: Memecah string input menjadi token."""
        # Menambahkan spasi pada simbol agar mudah dipisah (split)
        padded_code = (self.source_code.replace('(', ' ( ')
                                       .replace(')', ' ) ')
                                       .replace('{', ' { ')
                                       .replace('}', ' } ')
                                       .replace(';', ' ; '))
        tokens = padded_code.split()
        return tokens

    def syntax_semantic_analysis(self, tokens):
        """Tahap 2 & 3: Validasi sintaksis dan pengecekan semantik sederhana."""
        if not tokens or tokens[0] != 'while':
            raise SyntaxError("Struktur harus dimulai dengan keyword 'while'.")

        try:
            # Pengecekan Sintaksis Dasar (Keberadaan kurung dan kurawal)
            idx_open_paren = tokens.index('(')
            idx_close_paren = tokens.index(')')
            idx_open_brace = tokens.index('{')
            idx_close_brace = tokens.index('}')
        except ValueError:
            raise SyntaxError("Sintaks tidak valid: Kurung atau kurawal tidak lengkap.")

        # Ekstraksi bagian kondisi dan isi (body) perulangan
        condition_tokens = tokens[idx_open_paren+1 : idx_close_paren]
        body_tokens = tokens[idx_open_brace+1 : idx_close_brace]

        # Tahap Analisis Semantik: Memeriksa apakah variabel dalam kondisi sudah dideklarasikan
        if len(condition_tokens) >= 3:
            var_name = condition_tokens[0]
            if var_name not in self.symbol_table and not var_name.isnumeric():
                raise NameError(f"Semantic Error: Variabel '{var_name}' tidak terdefinisi (belum dideklarasikan).")
        else:
            raise SyntaxError("Format kondisi tidak valid.")

        condition = " ".join(condition_tokens)
        
        # Merapikan kembali spasi sebelum titik koma pada body
        body = " ".join(body_tokens).replace(" ;", ";")

        return condition, body

    def generate_tac(self):
        """Tahap 4: Menghasilkan Three-Address Code (TAC)."""
        tokens = self.lexical_analysis()
        condition, body = self.syntax_semantic_analysis(tokens)

        # Membuat label untuk awal perulangan dan akhir perulangan
        label_start = self.new_label()
        label_end = self.new_label()

        tac = []
        tac.append(f"{label_start}:")
        tac.append(f"ifFalse {condition} goto {label_end}")
        
        # Memproses isi statement di dalam blok while
        statements = [stmt.strip() for stmt in body.split(';') if stmt.strip()]
        for stmt in statements:
            tac.append(f"{stmt}")
            
        tac.append(f"goto {label_start}")
        tac.append(f"{label_end}:")

        return "\n".join(tac)


# ==========================================
# --- Contoh Penggunaan dan Eksekusi ---
# ==========================================
if __name__ == "__main__":
    # Source code yang akan dikompilasi
    source = "while ( x < 10 ) { x = x + 1 ; total = total + x ; }"
    
    print("Kode Sumber:")
    print(source)
    print("-" * 40)
    
    compiler = WhileLoopCompiler(source)

    # Menampilkan hasil Analisis Leksikal
    print("1. Hasil Analisis Leksikal (Tokens):")
    tokens = compiler.lexical_analysis()
    print(tokens)
    print("-" * 40)

    # Menampilkan hasil Generasi TAC (Telah melewati Sintaksis & Semantik)
    print("2. Hasil Generasi Three-Address Code (TAC):")
    try:
        tac_output = compiler.generate_tac()
        print(tac_output)
    except Exception as e:
        print(f"Kompilasi Gagal: {e}")

```

## 4. Penjelasan Tahapan Implementasi

Berikut adalah rincian dari setiap tahapan kompilasi yang terjadi dalam program di atas:

### A. Analisis Leksikal (*Lexical Analysis*)

Tahap ini dilakukan oleh *method* `lexical_analysis()`.
Program membaca teks *source code* mentah dan mengubahnya menjadi deretan *token*. Proses ini disimulasikan dengan memberikan spasi tambahan di sekitar simbol-simbol khusus seperti `(`, `)`, `{`, `}`, dan `;` agar metode `.split()` bawaan Python dapat mengenali kata kunci, operator, dan pengenal (*identifier*) sebagai entitas terpisah.

* **Input:** `"while ( x < 10 ) { x = x + 1 ; }"`
* **Output:** `['while', '(', 'x', '<', '10', ')', '{', 'x', '=', 'x', '+', '1', ';', '}']`

### B. Analisis Sintaksis (*Syntax Analysis*)

Tahap ini digabung ke dalam *method* `syntax_semantic_analysis()`.
Di sini, *array token* dievaluasi letaknya untuk memastikan instruksi mematuhi aturan pola tata bahasa. Program memvalidasi:

* Apakah kata pertama adalah `while`.
* Keberadaan pasangan kurung buka `(` dan kurung tutup `)` untuk kondisi.
* Keberadaan kurung kurawal `{` dan `}` untuk membatasi ruang lingkup blok pernyataan (*statement block*).
Jika ada simbol yang tidak lengkap atau susunannya salah, program akan melempar `SyntaxError`.

### C. Analisis Semantik (*Semantic Analysis*)

Tahap ini juga ada di dalam *method* `syntax_semantic_analysis()`.
Pada tahap ini, *compiler* tidak sekadar mengecek bentuk, tetapi "makna" dasar. Dalam program, ini disimulasikan dengan fitur **Mock Symbol Table** (`self.symbol_table`).

* Program membaca nama variabel pertama yang ada di dalam token kondisi (contoh: `x`).
* Program mengecek apakah `x` sudah terdaftar di *Symbol Table*.
* Jika variabel tidak ada di *Symbol Table* dan bukan merupakan angka, program akan melempar `NameError` (Semantic Error), mencegah program dikompilasi ke tahap selanjutnya karena ada pengenal yang tidak valid.

### D. Generasi Kode Antara (*TAC Generation*)

Tahap akhir dieksekusi oleh *method* `generate_tac()`.
Untuk instruksi `while`, struktur kontrol sangat mengandalkan "lompatan" (*jump* / `goto`). Program menghasilkan TAC dengan skema berikut:

1. **Menetapkan Label Awal (`L1:`):** Sebagai titik kembalinya perulangan.
2. **Evaluasi Kondisi:** Membuat instruksi kondisional terbalik. Jika kondisi bernilai **salah** (`ifFalse`), maka langsung lompat keluar ke Label Akhir (`goto L2`).
3. **Eksekusi Isi Perulangan (*Body*):** Menerjemahkan setiap *statement* (yang dipisahkan oleh `;`) ke dalam baris-baris TAC independen.
4. **Instruksi Ulang (`goto L1`):** Memaksa program kembali mengevaluasi kondisi di atas.
5. **Menetapkan Label Akhir (`L2:`):** Titik tempat program melanjutkan eksekusi jika perulangan selesai.
