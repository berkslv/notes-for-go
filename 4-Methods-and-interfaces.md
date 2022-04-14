# Methods and interfaces

- Go class içermez fakat type'lara methodlar tanımlayabilirsin. `method`, özel bir alıcı bağımsız değişkenine sahip bir fonksiyondur. Alıcı, func anahtar sözcüğü ile method adı arasında kendi bağımsız değişken listesinde görünür. Bu örnekte, Abs yönteminin v adlı Vertex türünde bir alıcısı (receiver) vardır.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	type Vertex struct {
		X, Y float64
	}

	func (v Vertex) Abs() float64 {
		return math.Sqrt(v.X*v.X + v.Y*v.Y)
	}

	func main() {
		v := Vertex{3, 4}
		fmt.Println(v.Abs())
	}
	```

- Methodlar sadece receiver içeren fonksiyonlardır. Bu fonksiyonlar `regular function` olarakta yazılabilir.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	type Vertex struct {
		X, Y float64
	}

	func Abs(v Vertex) float64 {
		return math.Sqrt(v.X*v.X + v.Y*v.Y)
	}

	func main() {
		v := Vertex{3, 4}
		fmt.Println(Abs(v))
	}
	```

- non-struct türlerde de bir yöntem bildirebilirsiniz. Bu örnekte, Abs yöntemiyle sayısal bir MyFloat türü görüyoruz. Yalnızca türü, yöntemle aynı pakette tanımlanan bir alıcıyla bir yöntem bildirebilirsiniz. Türü başka bir pakette tanımlanmış (int gibi yerleşik türleri içeren) bir alıcıyla bir yöntem bildiremezsiniz.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	type MyFloat float64

	func (f MyFloat) Abs() float64 {
		if f < 0 {
			return float64(-f)
		}
		return float64(f)
	}

	func main() {
		f := MyFloat(-math.Sqrt2)
		fmt.Println(f.Abs())
	}
	```

- Pointer reciever ile method bildirebilirsiniz. Bu, receiver türünün bazı T türleri için *T değişmez sözdizimine sahip olduğu anlamına gelir. (Ayrıca, T'nin kendisi *int gibi bir işaretçi olamaz.) Örneğin, buradaki Scale yöntemi *Vertex üzerinde tanımlanmıştır. Pointer receiverları olan yöntemler, receiver'ın işaret ettiği değeri değiştirebilir (Scale'in burada yaptığı gibi). Yöntemlerin genellikle receiverlarını değiştirmesi gerektiğinden, işaretçi receiverları değer receiverlarından daha yaygındır. Scale işlevinin bildiriminden * işaretini kaldırmayı deneyin ve programın davranışının nasıl değiştiğini gözlemleyin. Bir değer receiver'ı ile Scale yöntemi, orijinal Vertex değerinin bir kopyası üzerinde çalışır. (Bu, diğer herhangi bir işlev argümanıyla aynı davranıştır.) Scale yönteminin, ana işlevde bildirilen Vertex değerini değiştirmek için bir işaretçi receiver'ı olmalıdır.

- Kısaca pointer receiver ile tanımlanan methodlar, tipin referansı üzerinde çalışır, bu şekilde tekrar `v = v.Scale(10)` işlemi yapmadan sadece `v.Scale(10)` işlemi v değerini günceller.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	type Vertex struct {
		X, Y float64
	}

	func (v Vertex) Abs() float64 {
		return math.Sqrt(v.X*v.X + v.Y*v.Y)
	}

	func (v *Vertex) Scale(f float64) {
		v.X = v.X * f
		v.Y = v.Y * f
	}

	func main() {
		v := Vertex{3, 4}
		v.Scale(10)
		fmt.Println(v.Abs())
	}
	```

- Bu methodları reguler function olarak yazmak istersen pointer receiver alan fonksiyonu `Scale(&v, 10)` gibi pointer değerini aktararak vermelisin. Buna karşın type üzerinde çalışan methodlar için `v.Scale(10)` kullanımı yeterlidir. Go compiler bunu `(&v).Scale(10)` olarak derler.

	```go
	package main

	import "fmt"

	type Vertex struct {
		X, Y float64
	}

	func (v *Vertex) Scale(f float64) {
		v.X = v.X * f
		v.Y = v.Y * f
	}

	func ScaleFunc(v *Vertex, f float64) {
		v.X = v.X * f
		v.Y = v.Y * f
	}

	func main() {
		v := Vertex{3, 4}
		v.Scale(2)
		ScaleFunc(&v, 10)

		p := &Vertex{4, 3}
		p.Scale(3)
		ScaleFunc(p, 8)

		fmt.Println(v, p)
	}
	```

- Bir pointer reciever kullanmanın iki nedeni vardır. Birincisi, methodun recevier'ı işaret ettiği değeri değiştirebilmesidir. İkincisi, her yöntem çağrısında değeri kopyalamaktan kaçınmaktır. Örneğin, recevier büyük bir struct ise bu daha verimli olabilir. Bu örnekte, hem Scale hem de Abs, *Vertex recevier tipindedir, ancak Abs yönteminin recevier'ını değiştirmesine gerek yoktur. Genel olarak, belirli bir türdeki tüm yöntemler, ya değer ya da işaretçi recevierlarına sahip olmalıdır, ancak her ikisinin bir karışımı olmamalıdır. (Nedenini ilerleyen sayfalarda göreceğiz.)

	```go
	package main

	import (
		"fmt"
		"math"
	)

	type Vertex struct {
		X, Y float64
	}

	func (v *Vertex) Scale(f float64) {
		v.X = v.X * f
		v.Y = v.Y * f
	}

	func (v *Vertex) Abs() float64 {
		return math.Sqrt(v.X*v.X + v.Y*v.Y)
	}

	func main() {
		v := &Vertex{3, 4}
		fmt.Printf("Before scaling: %+v, Abs: %v\n", v, v.Abs())
		v.Scale(5)
		fmt.Printf("After scaling: %+v, Abs: %v\n", v, v.Abs())
	}
	```

- Bir interface type, bir dizi yöntem imzası olarak tanımlanır. Bir interface type değeri, bu yöntemleri uygulayan herhangi bir değeri tutabilir. Not: 22. satırdaki örnek kodda bir hata var. Vertex (değer type) Abser'i uygulamaz çünkü Abs yöntemi yalnızca *Vertex'te (işaretçi type) tanımlanır.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	type Abser interface {
		Abs() float64
	}

	func main() {
		var a Abser
		f := MyFloat(-math.Sqrt2)
		v := Vertex{3, 4}

		a = f  // a MyFloat implements Abser
		a = &v // a *Vertex implements Abser

		// In the following line, v is a Vertex (not *Vertex)
		// and does NOT implement Abser.
		a = v

		fmt.Println(a.Abs())
	}

	type MyFloat float64

	func (f MyFloat) Abs() float64 {
		if f < 0 {
			return float64(-f)
		}
		return float64(f)
	}

	type Vertex struct {
		X, Y float64
	}

	func (v *Vertex) Abs() float64 {
		return math.Sqrt(v.X*v.X + v.Y*v.Y)
	}
	```

- Bir type, method'larını uygulayarak bir interface uygular. Açık bir niyet beyanı, "uygulama" anahtar kelimesi yoktur. Örtük interface'ler, bir interface'in tanımını, uygulanmasından ayırır ve bu, daha sonra herhangi bir pakette önceden düzenleme olmaksızın görünebilir.

	```go
	package main

	import "fmt"

	type I interface {
		M()
	}

	type T struct {
		S string
	}

	// This method means type T implements the interface I,
	// but we don't need to explicitly declare that it does so.
	func (t T) M() {
		fmt.Println(t.S)
	}

	func main() {
		var i I = T{"hello"}
		i.M()
	}
	```

- Başlık altında, interface değerleri bir tuple ve somut bir tip olarak düşünülebilir: `(value, type)` Bir interface değeri, belirli bir temel somut türün değerini tutar. Bir interface değerinde bir yöntemin çağrılması, aynı adı taşıyan yöntemi, temel alınan türünde çalıştırır.

    ```go
    package main

    import (
        "fmt"
        "math"
    )

    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        fmt.Println(t.S)
    }

    type F float64

    func (f F) M() {
        fmt.Println(f)
    }

    func main() {
        var i I

        i = &T{"Hello"}
        describe(i)
        i.M()

        i = F(math.Pi)
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

- Arayüzün içindeki somut değer `nil` ise, yöntem `nil` alıcı ile çağrılacaktır. Bazı dillerde bu bir null pointer expection tetikler, ancak Go'da `nil` alıcıyla çağrılmayı zarif bir şekilde işleyen yöntemler yazmak yaygındır (bu örnekte M yönteminde olduğu gibi). `nil` somut değere sahip bir arayüz değerinin kendisinin non-nil olduğuna dikkat edin.

    ```go
    package main

    import "fmt"

    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        if t == nil {
            fmt.Println("<nil>")
            return
        }
        fmt.Println(t.S)
    }

    func main() {
        var i I

        var t *T
        i = t
        describe(i)
        i.M()

        i = &T{"hello"}
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

- nil interface değeri, ne değeri ne de somut türü tutar. nil interface'inde bir yöntemin çağrılması, bir çalışma zamanı hatasıdır, çünkü arabirim demetinin içinde hangi somut yöntemin çağrılacağını gösteren bir tür yoktur.

    ```go
    package main

    import "fmt"

    type I interface {
        M()
    }

    func main() {
        var i I
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

- Sıfır yöntem belirten interface türü, boş interface olarak bilinir: `interface{}`. Boş interface her hangi bir type tutabilir. Boş arabirimler, bilinmeyen türdeki değerleri işleyen kod tarafından kullanılır. Örneğin, fmt.Print, arabirim{} türünde herhangi bir sayıda bağımsız değişken alır.

    ```go
    package main

    import "fmt"

    func main() {
        var i interface{}
        describe(i)

        i = 42
        describe(i)

        i = "hello"
        describe(i)
    }

    func describe(i interface{}) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```

- Bir type assertion, bir interface değerinin temel alınan somut değerine erişim sağlar: `t := i.(T)`. Bu ifade, i interface değerinin somut T tipini tuttuğunu ve temeldeki T değerini t değişkenine atadığını iddia eder. Eğer bir T tutmazsa, ifade bir panic tetikleyecektir. Bir interface değerinin belirli bir türü tutup tutmadığını test etmek için, bir type assertion iki değer döndürebilir: temel alınan değer ve onaylamanın başarılı olup olmadığını bildiren bir boolean değeri: `t, ok := i.(T)`. Eğer bir T'ye sahipsem, o zaman t temel değer olacak ve ok doğru olacaktır. Değilse, tamam yanlış olur ve t, T tipinin sıfır değeri olur ve panic olmaz. Bu sözdizimi ile bir haritadan okuma arasındaki benzerliğe dikkat edin.


    ```go
    package main

    import "fmt"

    func main() {
        var i interface{} = "hello"

        s := i.(string)
        fmt.Println(s)

        s, ok := i.(string)
        fmt.Println(s, ok)

        f, ok := i.(float64)
        fmt.Println(f, ok)

        f = i.(float64) // panic
        fmt.Println(f)
    }
    ```

- Bir type switch, seri halinde birkaç tür iddiasına izin veren bir yapıdır. Bir type switch, normal bir switch ifadesi gibidir, ancak bir type switch'daki durumlar türleri belirtir (değerleri değil) ve bu değerler, verilen arabirim değeri tarafından tutulan değerin türüyle karşılaştırılır.

    ```go
    switch v := i.(type) {
    case T:
        // here v has type T
    case S:
        // here v has type S
    default:
        // no match; here v has the same type as i
    }
    ```

- Bir type switchındaki bildirim, bir type onaylaması i.(T) ile aynı sözdizimine sahiptir, ancak belirli T type, switch sözcük type ile değiştirilir. Bu switch ifadesi, arayüz değerinin T veya S type'ında bir değere sahip olup olmadığını test eder. T ve S durumlarının her birinde, v değişkeni sırasıyla T veya S type'ında olacak ve i tarafından tutulan değeri tutacaktır. Varsayılan durumda (eşleşme olmadığında), v değişkeni, i ile aynı arabirim type'ında ve değerindedir.

    ```go
    package main

    import "fmt"

    func do(i interface{}) {
        switch v := i.(type) {
        case int:
            fmt.Printf("Twice %v is %v\n", v, v*2)
        case string:
            fmt.Printf("%q is %v bytes long\n", v, len(v))
        default:
            fmt.Printf("I don't know about type %T!\n", v)
        }
    }

    func main() {
        do(21)
        do("hello")
        do(true)
    }
    ```

- En yaygın interface'lerden biri, fmt paketi tarafından tanımlanan Stringer'dır. Bir Stringer, kendisini bir string olarak tanımlayabilen bir türdür. fmt paketi (ve diğerleri) değerleri yazdırmak için bu arabirimi arar.

    ```go
    type Stringer interface {
        String() string
    }
    ```

    ```go
    package main

    import "fmt"

    type Person struct {
        Name string
        Age  int
    }

    func (p Person) String() string {
        return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
    }

    func main() {
        a := Person{"Arthur Dent", 42}
        z := Person{"Zaphod Beeblebrox", 9001}
        fmt.Println(a, z)
    }
    ```

- Go programları, hata durumunu error type ile ifade eder. Hata türü, fmt.Stringer'a benzer yerleşik bir arabirimdir:

    ```go
    type error interface {
        Error() string
    }
    ```

- Fonksiyonlar genellikle bir hata değeri döndürür ve çağrı kodu, hatanın sıfıra eşit olup olmadığını test ederek hataları işlemelidir. nil error başarıyı gösterir; non-nil bir error, başarısızlığı gösterir.

    ```go
    package main

    import (
        "fmt"
        "time"
    )

    type MyError struct {
        When time.Time
        What string
    }

    func (e *MyError) Error() string {
        return fmt.Sprintf("at %v, %s",
            e.When, e.What)
    }

    func run() error {
        return &MyError{
            time.Now(),
            "it didn't work",
        }
    }

    func main() {
        if err := run(); err != nil {
            fmt.Println(err)
        }
    }
    ```

- io paketi, bir veri akışının okunan ucunu temsil eden io.Reader interface belirtir. Go standart kitaplığı, dosyalar, ağ bağlantıları, sıkıştırıcılar, şifreler ve diğerleri dahil olmak üzere bu arabirimin birçok uygulamasını içerir. io.Reader interface'in bir Read yöntemi vardır:

    ```go
    func (T) Read(b []byte) (n int, err error)
    ```

- Read, verilen bayt dilimini verilerle doldurur ve doldurulan bayt sayısını ve bir hata değeri döndürür. Akış sona erdiğinde bir io.EOF hatası döndürür. Örnek kod bir strings.Reader oluşturur ve çıktısını bir seferde 8 bayt tüketir.

    ```go
    package main

    import (
        "fmt"
        "io"
        "strings"
    )

    func main() {
        r := strings.NewReader("Hello, Reader!")

        b := make([]byte, 8)
        for {
            n, err := r.Read(b)
            fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
            fmt.Printf("b[:n] = %q\n", b[:n])
            if err == io.EOF {
                break
            }
        }
    }
    ```

- Paket image, image interface tanımlar:

    ```go
    package image

    type Image interface {
        ColorModel() color.Model
        Bounds() Rectangle
        At(x, y int) color.Color
    }
    ```

- Bounds yönteminin Rectangle dönüş değeri aslında bir image.Rectangle'dır, çünkü bildirim paket görüntüsünün içindedir. color.Color ve color.Model türleri de arabirimlerdir, ancak önceden tanımlanmış color.RGBA ve color.RGBAModel uygulamalarını kullanarak bunu göz ardı edeceğiz. Bu arayüzler ve türler, görüntü/renk paketi tarafından belirlenir. Ayrıntılı bilgi için [dökümantasyona göz at](https://pkg.go.dev/image#Image).

    ```go
    package main

    import (
        "fmt"
        "image"
    )

    func main() {
        m := image.NewRGBA(image.Rect(0, 0, 100, 100))
        fmt.Println(m.Bounds())
        fmt.Println(m.At(0, 0).RGBA())
    }
    ```