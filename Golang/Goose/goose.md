# **Cara membuat migrations dengan Goose pada Golang**

Migrations adalah salah satu aspek yang penting dalam pengembangan perangkat lunak. Ini memungkinkan developer untuk mengelola perubahan struktural dalam database, seperti menambahkan kolom baru, mengubah nama tabel, atau membuat indeks baru dengan cara yang terkelola dan dapat direplikasi dengan mudah.

Goose adalah sebuah alat yang membantu dalam mengelola migrasi database dengan mudah dan efisien, terutama dalam proyek yang menggunakan bahasa pemrograman Go. Dalam artikel ini, kita akan membahas penggunaan Goose untuk melakukan migrasi database dalam proyek Go.

Apa itu Goose?

Goose adalah sebuah alat yang terinspirasi dari alat migrasi database Rails, yaitu ActiveRecord. Ini memungkinkan pengembang untuk mendefinisikan migrasi database sebagai file Go yang dapat dijalankan untuk memperbarui skema database. Goose menyediakan alat bantu yang efektif untuk melakukan perubahan struktural pada database dengan aman.

## **Langkah 1: Install Goose**

Pertama-tama, Anda perlu menginstal Goose. Anda dapat melakukannya dengan perintah:

```bash
go install github.com/pressly/goose/v3/cmd/goose@latest
```

Hal ini akan menginstall binary `goose` kedalam directory `$GOPATH/bin`

Untuk pengguna macOS Anda bisa lakukannya dengan perintah:

```bash
brew install goose
```

Lebih lengkap bisa baca dokumentasi [installation instructions](https://pressly.github.io/goose/installation/)

## **Langkah 2: Inisialisasi Struktur Project**

Buatlah folder `goose-migration`.

```bash
mkdir goose-migration \
    cd goose-migration
```

Kemudian inisiasi mod module.

```bash
go mod init goose-migration
```

Selanjutnya buat folder `config` dan file `database.go` untuk menyimpan konfigurasi koneksi ke database.

```bash
mkdir config \
    cd config \
    touch database.go
```

Copy script konfig dibawah ini ke dalam `database.go`.

```go
package config

import (
	"database/sql"
	"flag"
	"log"

	_ "github.com/go-sql-driver/mysql"
)

var pool *sql.DB

type DBConfig struct {
	connection string
	hostname   string
	port       string
	user       string
	password   string
	database   string
}

func init() {
	var dbConfig DBConfig

	dbConfig.hostname = "127.0.0.1"
	dbConfig.port = "3307"
	dbConfig.user = "root"
	dbConfig.password = "password"
	dbConfig.database = "goose-migration"
	dbConfig.connection = "mysql"

	consStr := dbConfig.user + ":" + dbConfig.password + "@tcp(" + dbConfig.hostname + ":" + dbConfig.port + ")/" + dbConfig.database + "?parseTime=true"
	dsn := flag.String("dsn", consStr, "connection data source name")
	flag.Parse()

	if len(*dsn) == 0 {
		log.Fatal("missing dsn flag")
	}
	var err error
	pool, err = sql.Open(dbConfig.connection, *dsn)
	if err != nil {
		log.Fatal("unable to use data source name", err)
	}
}

func SQLDBConn() *sql.DB {
	return pool
}
```

Sesuiakan nilai-nilai dibawah ini, menyesuaikan dengan konfigurasi koneksi yang Anda miliki.

```go
dbConfig.hostname = "127.0.0.1"
dbConfig.port = "3307"
dbConfig.user = "root"
dbConfig.password = "password"
dbConfig.database = "goose-migration"
dbConfig.connection = "mysql"
```

Penjelasan singkat terkait script konfigurasi diatas:

1. `pool` adalah variabel global yang akan menyimpan koneksi database.
2. `DBConfig` adalah struktur data yang menyimpan konfigurasi database seperti `hostname`, `port`, `user`, `password`, dan `database` yang akan digunakan.
3. Fungsi `init()` dijalankan secara otomatis ketika package di-load.
4. Di dalamnya, kita membuat sebuah variabel `dbConfig` yang merupakan instans dari `DBConfig` dan menginisialisasikan nilainya.
5. Kemudian kita membentuk connection string menggunakan nilai-nilai dari `dbConfig`.
6. Flag `dsn` digunakan untuk menerima data source name dari command line.
7. Selanjutnya, kita membuka koneksi database menggunakan `sql.Open()`, dan menyimpan hasilnya ke dalam variabel `pool`. Jika terjadi error saat membuka koneksi, program akan keluar dengan pesan error.
8. Fungsi `SQLDBConn()` digunakan untuk mengembalikan pool koneksi database yang telah dibuat. Hal ini memungkinkan penggunaan koneksi database tersebut di berbagai tempat dalam aplikasi.

Secara keseluruhan, script diatas bertujuan untuk mengatur koneksi ke database MySQL menggunakan Go, dengan memanfaatkan package `database/sql` dan driver MySQL yang disediakan oleh `github.com/go-sql-driver/mysql`.

Selanjutnya buatlah folder `migrations` untuk menyimpan file-file migrations.

Jika masih dalam folder config maka kita kembali ke root folder `cd ../` kemudian buat folder `migrations`

```bash
mkdir migrations
```

Selanjutnya buatlah file `main.go` untuk menjalankan migration.

```bash
touch main.go
```

Copy script dibawah ini:

```go
package main

import (
	"goose-migration/config"

	"github.com/pressly/goose"
)

func main() {
	sqlDB := config.SQLDBConn()

	if err := goose.SetDialect("mysql"); err != nil {
		panic(err)
	}

	err := goose.Up(sqlDB, "migrations")
	// err := goose.Down(sqlDB, "migrations")
	if err != nil {
		panic(err)
	}
}
```

Penjelasan terkait script `main.go` diatas:

1. Pertama-tama, kita memperoleh koneksi database dengan memanggil fungsi `SQLDBConn()` dari package `config`.
2. Selanjutnya, kita memanggil fungsi `goose.SetDialect("mysql")` untuk menetapkan dialek SQL yang digunakan, dalam hal ini MySQL.
3. Kemudian, kita melakukan migrasi database ke versi terbaru menggunakan fungsi `goose.Up(sqlDB, "migrations")`. Fungsi ini akan mencari file migrasi di direktori "migrations" dan menjalankan migrasi tersebut untuk membawa database ke versi terbaru.
4. Pilihan lain adalah melakukan rollback migrasi menggunakan `goose.Down(sqlDB, "migrations")`.

Secara keseluruhan, script diatas bertujuan untuk menjalankan migrasi database ke versi terbaru menggunakan package `goose`, dengan memanfaatkan koneksi database yang telah dikonfigurasi sebelumnya.

Setelah kita melakukan inisialisasi awal untuk migration menggunakan goose, maka diharapkan struktur folder akan seperti ini.

```bash
goose-migration
├─── config
│    └─── databse.go
├─── migrations
└─── main.go
```

## **Langkah 3: Membuat file migration**

Gunakan perintah `goose` untuk membuat file migrasi baru:

```bash
goose -dir migrations create create_users_table sql
```

Copy query migrasi ini;

```sql
-- +goose Up
-- +goose StatementBegin
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    uuid VARCHAR(36) NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL
);
-- +goose StatementEnd

-- +goose StatementBegin
INSERT INTO users (`uuid`, `name`, `email`, `password`) VALUES ("3d6ebd8d-6a04-449d-b232-bd1ca3ead0bf", "ADMIN", "admin@admin.com", "$2a$12$eHHwhkiE6/IEE37g1SLHRe5qcj1bTd26JMqRTdMonrY0hpiYSNJEe");
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
DROP TABLE users;
-- +goose StatementEnd
```

Ketika menjalankan migrasi untuk menaikkan versi skema dan rollback kembali, dapat dipisahkan dengan pernyataan `+goose Up` dan `+goose Down`.

Kita juga dapat menjalankan lebih dari satu query SQL saat melakukan migrasi up, dengan memisahkannya menggunakan `StatementBegin` dan mengakhiri dengan `StatementEnd`, begitu juga sama pada migrasi rollback. Pada script di atas, kita melakukan dua eksekusi query. Yang pertama adalah untuk membuat tabel "users", sementara yang kedua adalah untuk menyisipkan data user ke dalam tabel "users".

## **Langkah 4: Menjalankan Migration**

Selanjutnya kita bisa menjalankan migrations dengan meng-eksekusi file `main.go`.

```bash
go run .
```

Maka jika berhasil outputnya akan seperti ini.

```
2024/01/31 06:55:12 OK    20240131052901_create_users_table.sql
2024/01/31 06:55:12 goose: no migrations to run. current version: 20240131052901
```

## **Kesimpulan**

Membuat skrip fungsi untuk menjalankan migrasi bertujuan untuk memudahkan konfigurasi koneksi yang kita miliki, dalam hal ini yang telah kita buat adalah dalam file `main.go`.

Goose adalah alat migrasi database yang bermanfaat dalam pengembangan aplikasi berbasis Golang. Dengan menggunakan Goose, developer dapat mengelola perubahan skema database dengan lebih terstruktur dan mudah diikuti. Ini membantu memastikan integritas data dan kompatibilitas aplikasi seiring waktu.

Dengan mengikuti langkah-langkah di atas, Anda dapat mulai menggunakan Goose dalam proyek Golang Anda untuk mengelola perubahan skema database dengan efisien. Selamat mencoba!