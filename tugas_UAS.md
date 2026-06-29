# Tugas Proyek Akhir: Representasi Tahapan Kompilasi

## 📌 Deskripsi Tugas
Tugas ini bertujuan untuk menguji pemahaman Anda mengenai tahapan utama dalam proses kompilasi. Anda diminta untuk membuat representasi dari suatu konstruksi bahasa pemrograman yang melewati tahapan **Analisis Leksikal, Sintaksis, Semantik**, hingga **Generasi Kode Antara / *Three-Address Code* (TAC)**.

## 📝 Ketentuan Pengerjaan
1. **Pemilihan Konstruksi:** Pilih **satu** konstruksi sintaksis untuk diimplementasikan. Contoh pilihan:
   * Perulangan (*Looping*: `for`, `while`, atau `do-while`)
   * Percabangan/Kondisi (*Conditional*: `if-else` atau `switch-case`)
   * Penanganan Error (*Error handler*: `try-catch`)
   * Deklarasi fungsi/metode.
2. **Pembuatan *Pattern* (Pola):** Buatlah pola tata bahasa (*grammar*/Regex/BNF) yang mendefinisikan aturan sintaksis dari konstruksi yang Anda pilih.
3. **Implementasi Kode:** Buatlah program (menggunakan bahasa pemrograman bebas, misalnya Python, Java, C++, dll) yang dapat menyimulasikan proses kompilasi konstruksi tersebut. Program harus merepresentasikan tahapan:
   * **Leksikal:** Memecah *input* menjadi *token*.
   * **Sintaksis:** Membentuk *Abstract Syntax Tree* (AST) sederhana.
   * **Semantik:** Melakukan pengecekan dasar (misal: validasi tipe data atau keberadaan variabel).
   * **Generasi Kode (TAC):** Menghasilkan *Three-Address Code* dari AST yang terbentuk.
4. **Penjelasan & Dokumentasi:** Buatlah penjelasan lengkap dari implementasi program yang telah Anda buat. Dokumen penjelasan wajib diunggah dalam format **PDF** atau **Markdown (.md)**.

## 📅 Batas Waktu Pengumpulan
Seluruh *file* (*source code* dan dokumen penjelasan) wajib diunggah/dikumpulkan **selambat-lambatnya pada hari terakhir di pekan Ujian Akhir Semester (UAS)**.

***

# 💡 Contoh Pengerjaan Tugas (Sebagai Referensi)

Berikut adalah contoh bagaimana tugas ini dikerjakan agar Anda memiliki gambaran yang jelas.

### 1. Pilihan Konstruksi
**Konstruksi yang dipilih:** Percabangan / Kondisi (`If-Then-Else`)

### 2. *Pattern* (Pola Sintaks)
Pola didefinisikan menggunakan pendekatan *Backus-Naur Form* (BNF) sederhana:
```text
<if_stmt> ::= "if" "(" <condition> ")" "{" <statements> "}" "else" "{" <statements> "}"
<condition> ::= <identifier> <operator> <value>
<statements> ::= <identifier> "=" <value>
```

### 3. Implementasi Program
*(Contoh di bawah menggunakan Python untuk menyimulasikan pembuatan TAC dari sebuah struktur percabangan)*

```python
class IfElseCompiler:
    def __init__(self, source_code):
        self.source_code = source_code
        self.label_counter = 1
        self.temp_counter = 1

    def new_label(self):
        lbl = f"L{self.label_counter}"
        self.label_counter += 1
        return lbl

    def lexical_analysis(self):
        # Simulasi pemecahan token sederhana
        tokens = self.source_code.replace('(', ' ( ').replace(')', ' ) ').replace('{', ' { ').replace('}', ' } ').split()
        return tokens

    def syntax_semantic_analysis(self, tokens):
        # Simulasi validasi sintaksis dan semantik
        if 'if' in tokens and 'else' in tokens:
            condition = " ".join(tokens[tokens.index('(')+1 : tokens.index(')')])
            if_body = " ".join(tokens[tokens.index('{')+1 : tokens.index('}')])
            
            # Mencari blok else
            else_idx = tokens.index('else')
            else_body = " ".join(tokens[else_idx+2 : -1]) # mengambil isi di dalam {}
            
            return condition, if_body, else_body
        else:
            raise SyntaxError("Struktur If-Else tidak valid.")

    def generate_tac(self):
        tokens = self.lexical_analysis()
        condition, if_body, else_body = self.syntax_semantic_analysis(tokens)

        label_else = self.new_label()
        label_end = self.new_label()

        tac = []
        tac.append(f"ifFalse {condition} goto {label_else}")
        tac.append(f"{if_body}")
        tac.append(f"goto {label_end}")
        tac.append(f"{label_else}:")
        tac.append(f"{else_body}")
        tac.append(f"{label_end}:")
        
        return "\n".join(tac)

# --- Contoh Penggunaan ---
source = "if ( x > 5 ) { y = 1 } else { y = 0 }"
compiler = IfElseCompiler(source)

print("--- Hasil Analisis Leksikal (Tokens) ---")
print(compiler.lexical_analysis())

print("\n--- Generasi Three-Address Code (TAC) ---")
print(compiler.generate_tac())
```
