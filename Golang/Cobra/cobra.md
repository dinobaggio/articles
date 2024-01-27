# **Cara menggunakan cobra pada golang & membangun aplikasi basis command line**

Cobra adalah sebuah library di Golang yang digunakan untuk membuat aplikasi baris perintah (command-line applications) dengan fitur-fitur yang kuat dan ekstensibel. Dengan Cobra, Anda dapat dengan mudah membuat perintah, sub-perintah, dan opsi pada aplikasi baris perintah Golang Anda. Dalam artikel ini, kita akan membahas langkah-langkah dasar dalam menggunakan Cobra untuk mengembangkan aplikasi baris perintah.

## **Langkah 1: Instalasi Cobra**

Langkah pertama adalah menginstal pustaka Cobra. Anda dapat melakukannya dengan menjalankan perintah:

```bash
go get -u github.com/spf13/cobra/cobra
```

## **Langkah 2: Membuat Folder cmd dan command**

Buatlah sebuah folder bernama `cmd` dan create file dengan nama `root.go` file tersebut akan menajadi file basis perintah yang akan di eksekusi di awal:

```bash
mkdir cmd 
cd cmd
touch root.go
```

maka struktur folder akan seperti ini:

```bash
cobra-app
├── cmd
│   └── root.go
└── main.go
```

## **Langkah 3: Membuat command pertama**

Buatlah sebuah perintah pertama dan import package Cobra:

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
	"os"
)
```

Gunakan fungsi `cobra.Command` untuk membuat perintah utama:

```go
var rootCmd = &cobra.Command{
  Use:   "cobra-app",
  Short: "Aplikasi baris perintah sederhana dengan Cobra",
  Long:  `Ini adalah contoh aplikasi baris perintah sederhana yang dikembangkan dengan menggunakan Cobra di Golang.`,
  Run: func(cmd *cobra.Command, args []string) {
    // Kode yang akan dijalankan ketika perintah utama dijalankan
    fmt.Println("Halo dari aplikasi baris perintah!")
  },
}
```

Buatlah fungsi `Execute` untuk menjalankan perintah pertama yang akan di letakan pada file utama `main.go`

```go
func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Fprintf(os.Stderr, "Whoops. There was an error while executing your CLI '%s'", err)
		os.Exit(1)
	}
}
```

maka full of code pada file `root.go` akan seperti ini:

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
	"os"
)

var rootCmd = &cobra.Command{
  Use:   "cobra-app",
  Short: "Aplikasi baris perintah sederhana dengan Cobra",
  Long:  `Ini adalah contoh aplikasi baris perintah sederhana yang dikembangkan dengan menggunakan Cobra di Golang.`,
  Run: func(cmd *cobra.Command, args []string) {
    // Kode yang akan dijalankan ketika perintah utama dijalankan
    fmt.Println("Halo dari aplikasi baris perintah!")
  },
}

func Execute() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Fprintf(os.Stderr, "Whoops. There was an error while executing your CLI '%s'", err)
		os.Exit(1)
	}
}

```

## **Langkah 4: Membuat sub command**

Buatlah sebuah file `greet.go` untuk menjalan sub command

```bash
touch greet.go
```

buatlah perintah 

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
)

var greetCmd = &cobra.Command{
  Use:   "greet",
  Short: "Sapa pengguna",
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Halo! Selamat datang di aplikasi baris perintah.")
  },
}
```

buatlah fungsi `init` dan tambahkan sub-command greet kedalam root command

```
NOTE: fungsi init digunakan untuk menjalankan kode inisialisasi sebelum program utama dimulai.
```

```go
func init() {
	rootCmd.AddCommand(greetCmd)
}
```

maka full of code pada file `greet.go` akan seperti ini:

```go
package cmd

import (
	"fmt"
	"github.com/spf13/cobra"
)

var greetCmd = &cobra.Command{
  Use:   "greet",
  Short: "Sapa pengguna",
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Halo! Selamat datang di aplikasi baris perintah.")
  },
}

func init() {
	rootCmd.AddCommand(greetCmd)
}
```

## **Langkah 5: Tambahkan root command kedalam main package**

Panggilang fungsi `Execute` pada root command kedalam `main.go` seperti ini:

```go
package main

import "cobra-app/cmd"

func main() {
	cmd.Execute()
}
```

## **Langkah 6: Menjalankan aplikasi**

Anda bisa menjalankan aplikasi menggunakan perintah:

```bash
go run .
```

```
Halo dari aplikasi baris perintah!
```

dan 

```bash
go run . greet
```

```
Halo! Selamat datang di aplikasi baris perintah.
```

## **Kesimpulan**

Dengan menggunakan Cobra, Anda dapat dengan mudah membuat aplikasi baris perintah yang terstruktur dan mudah di-maintain. Pustaka ini menyediakan banyak fitur tambahan seperti penanganan opsi baris perintah, validasi input, dan dokumentasi otomatis. Jadi, mulailah mengintegrasikan Cobra dalam proyek Golang Anda untuk membangun aplikasi baris perintah yang kuat dan bersih.