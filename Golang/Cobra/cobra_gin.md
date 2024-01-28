# **Cara menggunakan Cobra untuk menjalankan server Golang Gin**

Gin dan Cobra adalah dua alat yang populer dalam ekosistem Go (atau Golang) yang dapat digunakan untuk membuat aplikasi CLI (Command Line Interface) dan aplikasi web secara efisien. Dalam artikel ini, kita akan membahas cara menggunakan keduanya untuk membuat aplikasi CLI sederhana dan aplikasi web dengan Go.

## **Langkah 1: Instalasi Gin dan Cobra**

Langkah pertama adalah instalasi Gin dan Cobra, sebelum instalasi Gin dan Cobra buatlah sebuah root folder project kita yang diberi nama gin-cobra

```bash
mkdir gin-cobra
cd gin-cobra
```

kemudian inisiasi mod module

```go
go mod init gin-cobra
```

selanjutnya instalasi Gin dan Cobra, jalankan perintah dibawah ini untuk intalasi Gin dan Cobra

```bash
go get \
    github.com/gin-gonic/gin \
    github.com/spf13/cobra/cobra
```

Setelah proses instalasi selesai, Gin dan Cobra akan tersedia untuk aplikasi anda

## **Langkah 2: Membuat Directory dan File-File yang dibutuhkan**

Sebelumnya anda telah membuat root folder untuk project yang akan digunakan untuk membuat aplikasi Gin dengan Cobra, tahapan ini adalah untuk membuat directory dan file-file yang akan dibutuhkan untuk menjalankan aplikasi Gin dan Cobra.

Selanjutnya buatlah file `main.go` file ini adalah entry point untuk menjalankan aplikasi Gin dan Cobra.

```bash
touch main.go
```

isian awal untuk file `main.go` adalah sebagai berikut

```go
package main

func main() {}
```

selanjutnya buatlah folder `bin` dan `cmd`

folder `bin` berguna untuk minyimpan syntax aplikasi Gin dimana isinya adalah sebuah code untuk menjalankan sebuah server http dengan host `localhost` dan default port `3000`

folder `cmd` berguna untuk menyimpan syntax command line Cobra untuk menjalankan code yang telah dibuat pada folder `bin`

jalankanlah perintah dibawah ini:

```bash
mkdir bin \
    cmd
```

selanjutnya buatlah file `app.go` pada folder `bin` dan file `root.go` pada folder `cmd`

dimana telah disebutkan sebelumnya file `app.go` pada folder `bin` digunakan untuk menyimpan sebuah code untuk menjalankan aplikasi server Gin.

dan file `root.go` pada folder `cmd` untuk menyimpan sebuah code inisiasi awal untuk kebutuhan command line Cobra.

maka struktur folder akan seperti ini:

```bash
gin-cobra
├── bin
│   └── app.go
├── cmd
│   └── root.go
└── main.go
```

## **Langkah 3: Membuat Aplikasi Web Gin**

selanjutnya pada file `bin/app.go` buatlah sebuah code untuk menjalankan aplikasi web Gin

```go
package bin

import (
	"fmt"
	"log"
	"net/http"

	"github.com/gin-gonic/gin"
)

type App struct {
	Router *gin.Engine
}

func (a *App) Initialize() {
	a.Router = gin.Default()
	a.setRoutes()
}

// setRoutes sets up the routes for the application.
func (a *App) setRoutes() {
	router := a.Router
	router.GET("/healthcheck", func(c *gin.Context) {
        c.JSON(http.StatusOK, "It's work")
    })
}

// Run starts the application.
func (a *App) Run(addr string) {
	fmt.Println("running", addr)
	err := http.ListenAndServe(addr, a.Router)
	if err != nil {
		log.Panic("Failed to start the server:", err)
	}
}

func NewApp() *App {
	app := App{}
	app.Initialize()

	return &app
}
```

Berikut adalah penjelasan singkat dari setiap bagian code diatas:

1. `type App struct`: Ini adalah sebuah struktur yang mendefinisikan aplikasi. Struktur ini memiliki satu properti yaitu Router yang bertipe `*gin.Engine`.

2. `func (a *App) Initialize()`: Ini adalah method untuk menginisialisasi aplikasi. Method ini membuat router menggunakan `gin.Default()` dan memanggil method `setRoutes()` untuk menetapkan rute-rute aplikasi.

3. `func (a *App) setRoutes()`: Ini adalah method untuk menetapkan rute-rute aplikasi. Dalam contoh ini, hanya ada satu rute yaitu `/healthcheck` yang akan merespons dengan pesan **"It's work"** jika diakses.

4. `func (a *App) Run(addr string)`: Ini adalah method untuk menjalankan server. Method ini mencetak pesan "running" diikuti dengan alamat server yang sedang berjalan, kemudian memulai server HTTP menggunakan `http.ListenAndServe()`.

5. `func NewApp() *App`: Ini adalah fungsi pembuat (constructor) untuk membuat instance baru dari aplikasi. Fungsi ini membuat instance dari `App`, memanggil method `Initialize()` untuk menginisialisasinya, dan mengembalikan pointer ke instance tersebut.

Dengan demikian, code diatas membuat sebuah aplikasi web sederhana yang dapat menjalankan server HTTP dengan satu rute `/healthcheck` yang akan merespons dengan pesan **"It's work"** jika diakses.

## **Lankah 4: Membuat Command Line untuk menjalankan server menggunakan Cobra**

kemudian pada file `cmd/root.go` buatlah sebuah inisiasi awal untuk menjalankan aplikasi command line Cobra pada file ini kita tidak akan menggunakannya untuk menjalankan server, file ini hanya bertujuan untuk root awal command line.

```go
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
	Use:   "gin-cobra",
	Short: "gin-cobra menjalankan sebuah aplikasi server dengan command line",
	Long:  "",
	Run: func(cmd *cobra.Command, args []string) {
        // some code
	},
}

func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Fprintf(os.Stderr, "Whoops. There was an error while executing your CLI '%s'", err)
		os.Exit(1)
	}
}
```

## **Kesimpulan**

Dengan menggunakan Cobra dan Gin, Anda dapat dengan mudah membuat aplikasi CLI dan aplikasi web menggunakan Go. Kedua alat ini menyediakan abstraksi yang kuat untuk mempercepat pengembangan dan memungkinkan Anda fokus pada logika bisnis aplikasi Anda.