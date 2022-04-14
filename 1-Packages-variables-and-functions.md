# Packages, variables and functions

## Packages

- Aşağıdaki iki import ifadeside aynı anlama gelir.

	```go
	import (
		"fmt"
		"math"
	)
	```

	```go

	import "fmt"
	import "math"
	```

- Büyük harfle başlayan isimler otomatik olarak export edilir. 


## Functions


- Fonksiyonlar aşağıdaki gibi oluşturulur. Parametre ve dönüş tipleri belirtilir.

	```go
	func add(x int, y int) int {
		return x + y
	}
	```

- Parametre tipi aynı ise tek seferde yazılabılır.

	```go
	func add(x, y int) int {
		return x + y
	}
	```

- Birden çok sonuç dönülebilir.

	```go
	func swap(x, y string) (string, string) {
		return y, x
	}
	```

- Dönüş değerleri adlandırılabilir. Eğer öyleyse, fonksiyonun en üstünde tanımlanan değişkenler olarak kabul edilirler. Bunlar dönüş değerlerini açıklamak için kullanulabilir. "naked return" ile aynı isimdeki değişken değerleri döndürülür. Bu yapının sadece kısa fonksiyonlarda kullanılması tavsiye edilir.

	```go
	func split(sum int) (x, y int) {
		x = sum * 4 / 9
		y = sum - x
		return
	}
	```

## Variables

- `var` deyimi bir değişkenler listesi bildirir. Package veya function seviyesinde tanımlanabilir.

	```go
	package main

	import "fmt"

	var i, j int = 1, 2

	func main() {
		var c, python, java = true, false, "no!"
		fmt.Println(i, j, c, python, java)
	}
	```
- Değişkenler tanımlanırken tipleride verilebilir. 

- `:=` ifadesi ile implicit değişken tanımlama yapılabilir. Fonksiyon dışarısında çalışmaz.

	```go
	package main

	import "fmt"

	func main() {
		var i, j int = 1, 2
		k := 3
		c, python, java := true, false, "no!"

		fmt.Println(i, j, k, c, python, java)
	}
	```

- Go için `basic types` aşağıdaki gibidir. `int`, `uint`, `uintptr` tipleri 32 bit sistemlerde 32 bit, 64 bit sistemlerde 64 bit yer kaplar. Özel olarak gerekmedikçe unsigned yerine normal int kullanmak yeterlidir. 

	```txt
	bool

	string

	int  int8  int16  int32  int64
	uint uint8 uint16 uint32 uint64 uintptr

	byte // alias for uint8

	rune // alias for int32
		// represents a Unicode code point

	float32 float64

	complex64 complex128
	```

	```go
	package main

	import (
		"fmt"
		"math/cmplx"
	)

	var (
		ToBe   bool       = false
		MaxInt uint64     = 1<<64 - 1
		z      complex128 = cmplx.Sqrt(-5 + 12i)
	)

	func main() {
		fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
		// Type: bool Value: false
		fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
		// Type: uint64 Value: 18446744073709551615
		fmt.Printf("Type: %T Value: %v\n", z, z)
		// Type: complex128 Value: (2+3i)
	}
	```

- Açık bir başlangıç değeri olmadan bildirilen değişkenlere `zero` değerleri verilir.

	```
	zero value:

	numbers -> 0
	boolean -> false 
	strings -> ""
	```

- T(v) ifadesi, v değerini T tipine dönüştürür.

	```go
	var i int = 42
	var f float64 = float64(i)
	var u uint = uint(f)
	```

- Implicit olarak tip ataması yapılabilir. Fakat sayısal değerleri implicit olarak atamak `int`, `float64` veya `complex128` sayıları oluşturabilir

	```go
	var i int
	j := i // j is an int

	i := 42           // int
	f := 3.142        // float64
	g := 0.867 + 0.5i // complex128
	```

- Constants, değişkenler gibi ancak `const` anahtar sözcüğüyle bildirilir. Constants character, string, boolean veya numeric değerler olabilir. Constants := sözdizimi kullanılarak bildirilemez.

	```go
	package main

	import "fmt"

	const Pi = 3.14

	func main() {
		const World = "世界"
		fmt.Println("Hello", World)
		fmt.Println("Happy", Pi, "Day")

		const Truth = true
		fmt.Println("Go rules?", Truth)
	}
	```

- Sayısal sabitler yüksek kesinlik taşırlar.

	```go
	package main

	import "fmt"

	const (
		// Create a huge number by shifting a 1 bit left 100 places.
		// In other words, the binary number that is 1 followed by 100 zeroes.
		Big = 1 << 100
		// Shift it right again 99 places, so we end up with 1<<1, or 2.
		Small = Big >> 99
	)

	func needInt(x int) int {
		return x*10 + 1
	}
	func needFloat(x float64) float64 {
		return x * 0.1
	}

	func main() {
		fmt.Println(needInt(Small))
		// 21
		fmt.Println(needFloat(Small))
		// 0.2
		fmt.Println(needFloat(Big))
		// 1.2676506002282295e+29
	}
	```