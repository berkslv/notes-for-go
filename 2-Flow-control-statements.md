# Flow control statements

- Go sadece bir adet döngü yapısı içerir. `for` statement için tanımlanan döngü değişkeni sadece for içerisinde geçerlidir. () parantezi kullanılmaz fakat {} parantezleri kullanılır.

	```go
	package main

	import "fmt"

	func main() {
		sum := 0
		for i := 0; i < 10; i++ {
			sum += i
		}
		fmt.Println(sum)
	}
	```

- Loop değişkeninin tanımı ve arttırımı zorunlu değildir. Bu şekilde C-like dillerdeki `while` yapısı uygulanmış olur.

	```go
	package main

	import "fmt"

	func main() {
		sum := 1
		for sum < 1000 {
			sum += sum
		}
		fmt.Println(sum)
	}
	```

- Eğer döngü koşulu boş bırakılırsa sonsuz döngü oluşur.

	```go
	package main

	func main() {
		for {
		}
	}
	```

- If yapısı içinde () kullanılmazken {} kullanılır.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	func sqrt(x float64) string {
		if x < 0 {
			return sqrt(-x) + "i"
		}
		return fmt.Sprint(math.Sqrt(x))
	}

	func main() {
		fmt.Println(sqrt(2), sqrt(-4))
	}
	```

- For yapısında oluduğu gibi if bloğunda değişken tanımlama yapılabilir. Bu şekilde tanımlanan değişkenlere sadece if içerisinden erişilebilir. Else yapısıda bilindiği gibidir. 

	```go
	func pow(x, n, lim float64) float64 {
		if v := math.Pow(x, n); v < lim {
			return v
		} else {
			fmt.Printf("%g >= %g\n", v, lim)
		}
		// can't use v here, though
		return lim
	}
	```

- Switch yapısında break kullanılmaz çünkü bir case tutarsa diğer case'ler kontrol edilmez. Switch top-to-bottom olarak işlenir.

	```go
	package main

	import (
		"fmt"
		"runtime"
	)

	func main() {
		fmt.Print("Go runs on ")
		switch os := runtime.GOOS; os {
		case "darwin":
			fmt.Println("OS X.")
		case "linux":
			fmt.Println("Linux.")
		default:
			// freebsd, openbsd,
			// plan9, windows...
			fmt.Printf("%s.\n", os)
		}
	}
	```

- Boş switch, `switch true` ile aynıdır. Bu yapı, uzun if-then-else zincirleri yazmanın temiz bir yolu olabilir.

	```go
	package main

	import (
		"fmt"
		"time"
	)

	func main() {
		t := time.Now()
		switch {
		case t.Hour() < 12:
			fmt.Println("Good morning!")
		case t.Hour() < 17:
			fmt.Println("Good afternoon.")
		default:
			fmt.Println("Good evening.")
		}
	}
	```

- Bir `defer` ifadesi, çevreleyen işlev dönene kadar bir işlevin yürütülmesini erteler. Ertelenen çağrının bağımsız değişkenleri hemen değerlendirilir, ancak çevreleyen işlev dönene kadar işlev çağrısı yürütülmez.

	```go
	package main

	import "fmt"

	func main() {
		defer fmt.Println("world")

		fmt.Println("hello")
		// hello
		// world
	}
	```

- Ertelenmiş işlev çağrıları bir yığına gönderilir. Bir işlev döndüğünde, ertelenen çağrıları son giren ilk çıkar sırasına göre yürütülür. [Daha fazla kaynak](https://go.dev/blog/defer-panic-and-recover)

	```go
	package main

	import "fmt"

	func main() {
		fmt.Println("counting")

		for i := 0; i < 10; i++ {
			defer fmt.Println(i)
		}

		fmt.Println("done")
	}
	```