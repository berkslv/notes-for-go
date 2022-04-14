 # More types

- Go'da işaretçiler vardır. İşaretçi, bir değerin bellek adresini tutar. `*T` türü, bir `T` değerine yönelik bir işaretçidir. Boş değeri `nil` olur. `&` operatörü, işlenenine bir işaretçi oluşturur. C dilinin aksine Go pointer aritmetiği içermez.

	```go
	package main

	import "fmt"

	func main() {
		i, j := 42, 2701

		p := &i         // point to i
		fmt.Println(*p) // read i through the pointer
		fmt.Println(&p) // read ram address
		*p = 21         // set i through the pointer
		fmt.Println(i)  // see the new value of i

		p = &j         // point to j
		*p = *p / 37   // divide j through the pointer
		fmt.Println(j) // see the new value of j
	}
	```

- Struct birden fazla alan (field) tutar. Nokta ile alanlarına erişilebilir.
  
	```go
	package main

	import "fmt"

	type Vertex struct {
		X int
		Y int
	}

	func main() {
		v := Vertex{1, 2}
		v.X = 4
		fmt.Println(v.X)
	}
	```

- Struct alanlarına bir struct pointer aracılığıyla erişilebilir. Bir struct'ın X alanına erişmek için p struct pointer'ına sahipken (*p).X yazabiliriz. Bununla birlikte, bu gösterim hantaldır, bu nedenle dil, bunun yerine, açık bir referans göndermeden sadece p.X yazmamıza izin verir.

	```go
	package main

	import "fmt"

	type Vertex struct {
		X int
		Y int
	}

	func main() {
		v := Vertex{1, 2}
		p := &v
		p.X = 1e9
		fmt.Println(v)
	}
	```

- Bir yapı değişmezi, alanlarının değerlerini listeleyerek yeni tahsis edilmiş bir yapı değerini belirtir. Ad: sözdizimini kullanarak alanların yalnızca bir alt kümesini listeleyebilirsiniz. (Ve adlandırılmış alanların sırası önemsizdir.) Özel önek &, yapı değerine bir işaretçi döndürür.

	```go
	package main

	import "fmt"

	type Vertex struct {
		X, Y int
	}

	var (
		v1 = Vertex{1, 2}  // has type Vertex
		v2 = Vertex{X: 1}  // Y:0 is implicit
		v3 = Vertex{}      // X:0 and Y:0
		p  = &Vertex{1, 2} // has type *Vertex
	)

	func main() {
		fmt.Println(v1, p, v2, v3)
	}
	```

- `[n]T` türü, `T` türünün `n` değerinden oluşan bir dizidir. Bir dizinin uzunluğu, türünün bir parçasıdır, bu nedenle diziler yeniden boyutlandırılamaz. Bu sınırlayıcı görünüyor, ancak endişelenmeyin; Go, dizilerle çalışmanın uygun bir yolunu sağlar.

	```go
	package main

	import "fmt"

	func main() {
		var a [2]string
		a[0] = "Hello"
		a[1] = "World"
		fmt.Println(a[0], a[1])
		fmt.Println(a)

		primes := [6]int{2, 3, 5, 7, 11, 13}
		fmt.Println(primes)
	}
	```

- Bir dizinin sabit bir boyutu vardır. Öte yandan slice (dilim), bir dizinin öğelerine dinamik olarak boyutlandırılmış, esnek bir görünümdür. Pratikte, slice'lar dizilerden çok daha yaygındır. `[]T` türü, `T` türünden öğeler içeren bir slice'dır. İki nokta üst üste ile ayrılmış, düşük ve yüksek sınır olmak üzere iki dizin belirtilerek bir slice oluşturulur: `a[low : high]` Bu, ilk öğeyi içeren ancak son öğeyi hariç tutan yarı açık bir aralık seçer. Takip eden ifade, a'nın 1'den 3'e kadar olan öğelerini içeren bir slice oluşturur: `a[1:4]`


	```go
	package main

	import "fmt"

	func main() {
		primes := [6]int{2, 3, 5, 7, 11, 13}

		var s []int = primes[1:4]
		fmt.Println(s)
	}
	```

- Bir slice herhangi bir veri saklamaz, sadece temel bir dizinin bir bölümünü tanımlar. Bir slice'ın öğelerini değiştirmek, onun temel dizisinin karşılık gelen öğelerini değiştirir. Aynı temel diziyi paylaşan diğer slice'lar bu değişiklikleri görecektir.

	```go
	package main

	import "fmt"

	func main() {
		names := [4]string{
			"John",
			"Paul",
			"George",
			"Ringo",
		}
		fmt.Println(names)

		a := names[0:2]
		b := names[1:3]
		fmt.Println(a, b)

		b[0] = "XXX"
		fmt.Println(a, b)
		fmt.Println(names)
	}
	```

- Slice literal, uzunluğu olmayan bir array literal gibidir. Bu bir array literaldir: `[3]bool{true, true, false}` Bu, yukarıdakiyle aynı diziyi oluşturur, ardından ona başvuran bir dilim oluşturur: `[]bool{true, true, false}`

	```go
	package main

	import "fmt"

	func main() {
		q := []int{2, 3, 5, 7, 11, 13}
		fmt.Println(q)

		r := []bool{true, false, true, true, false, true}
		fmt.Println(r)

		s := []struct {
			i int
			b bool
		}{
			{2, true},
			{3, false},
			{5, true},
			{7, true},
			{11, false},
			{13, true},
		}
		fmt.Println(s)
	}
	```

- Slice için sınır verilmezse alt sınıfr 0, üst sınır o dizinin length'i kadar olacaktır. 

- Bir slice'ın hem uzunluğu hem de kapasitesi vardır. Bir slice'ın uzunluğu, içerdiği öğelerin sayısıdır. Bir slice'ın kapasitesi, dilimdeki ilk öğeden itibaren temel alınan dizideki öğelerin sayısıdır. Bir slice'ın uzunluğu ve kapasitesi, len(s) ve cap(s) ifadeleri kullanılarak elde edilebilir. Yeterli kapasiteye sahip olması koşuluyla, bir slice'ın uzunluğunu yeniden dilimleyerek uzatabilirsiniz. Örnek programdaki dilim işlemlerinden birini kapasitesini aşmak için değiştirmeyi deneyin ve ne olduğunu görün.

	```go
	package main

	import "fmt"

	func main() {
		s := []int{2, 3, 5, 7, 11, 13}
		printSlice(s)

		// Slice the slice to give it zero length.
		s = s[:0]
		printSlice(s)

		// Extend its length.
		s = s[:4]
		printSlice(s)

		// Drop its first two values.
		s = s[2:]
		printSlice(s)
	}

	func printSlice(s []int) {
		fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
	}
	```

- Bir dilimin boş değeri `nil` olur. Boş dilimin uzunluğu ve kapasitesi 0'dır ve altta yatan bir diziye sahip değildir.

	```go
	package main

	import "fmt"

	func main() {
		var s []int
		fmt.Println(s, len(s), cap(s)) // [] 0 0
		if s == nil {
			fmt.Println("nil!") // nil!
		}
	}
	```

- Slice'lar, yerleşik `make` işleviyle oluşturulabilir; dinamik olarak boyutlandırılmış dizileri bu şekilde yaratırsınız. `make` işlevi sıfırlanmış bir dizi ayırır ve bu diziye başvuran bir dilim döndürür: `a := make([]int, 5)  // len(a)=5` Bir kapasite belirtmek için, aşağıdakileri yapmak üzere üçüncü bir argüman iletin: `b := make([]int, 0, 5) // len(b)=0, cap(b)=5`

	```go
	package main

	import "fmt"

	func main() {
		a := make([]int, 5)
		printSlice("a", a)

		b := make([]int, 0, 5)
		printSlice("b", b)

		c := b[:2]
		printSlice("c", c)

		d := c[2:5]
		printSlice("d", d)
	}

	func printSlice(s string, x []int) {
		fmt.Printf("%s len=%d cap=%d %v\n",
			s, len(x), cap(x), x)
	}
	```

- Dilimler, diğer dilimler de dahil olmak üzere herhangi bir türü içerebilir.

- Bir dilime yeni öğeler eklemek yaygındır ve bu nedenle Go yerleşik bir `append` işlevi sağlar. Paketin dökümantasyonuna göre append: `func append(s []T, vs ...T) []T`. Eklemenin ilk parametresi s, T türünde bir dilimdir ve geri kalanı dilime eklenecek T değerleridir. Sonuç ekleme değeri, orijinal dilimin tüm öğelerini ve sağlanan değerleri içeren bir dilimdir. s'nin destek dizisi verilen tüm değerlere sığmayacak kadar küçükse, daha büyük bir dizi tahsis edilecektir. Döndürülen dilim, yeni tahsis edilen diziye işaret edecektir.

	```go
	package main

	import "fmt"

	func main() {
		var s []int
		printSlice(s)

		// append works on nil slices.
		s = append(s, 0)
		printSlice(s)

		// The slice grows as needed.
		s = append(s, 1)
		printSlice(s)

		// We can add more than one element at a time.
		s = append(s, 2, 3, 4)
		printSlice(s)
	}

	func printSlice(s []int) {
		fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
	}
	```

- Slices hakkında daha fazla bilgi için [ziyaret et](https://go.dev/blog/slices-intro).

- for döngüsünün `range` formu, bir slice veya map üzerinde yinelenir. Bir slice üzerinde değişiklik yapıldığında, her yineleme için iki değer döndürülür. Birincisi dizin, ikincisi ise o dizindeki öğenin bir kopyasıdır. 

	```go
	package main

	import "fmt"

	var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

	func main() {
		for i, v := range pow {
			fmt.Printf("2**%d = %d\n", i, v)
		}
	}
	```
- _ öğesine atayarak dizini veya değeri atlayabilirsiniz. Yalnızca dizini istiyorsanız, ikinci değişkeni atlayabilirsiniz.

	```go
	for i, _ := range pow
	for _, value := range pow
	for i := range pow
	```

- Bir map, anahtarları değerlere eşler. Bir map'ın boş değeri nil olur. Boş bir map'ın anahtarı yoktur ve anahtarlar eklenemez. make işlevi, verilen türde, başlatılmış ve kullanıma hazır bir map döndürür.

	```go
	package main

	import "fmt"

	type Vertex struct {
		Lat, Long float64
	}

	var m map[string]Vertex

	func main() {
		m = make(map[string]Vertex)
		m["Bell Labs"] = Vertex{
			40.68433, -74.39967,
		}
		fmt.Println(m["Bell Labs"])
	}
	```

- Map literal, struct literal gibidir fakat key gereklidir.

	```go
	package main

	import "fmt"

	type Vertex struct {
		Lat, Long float64
	}

	var m = map[string]Vertex{
		"Bell Labs": Vertex{
			40.68433, -74.39967,
		},
		"Google": Vertex{
			37.42202, -122.08408,
		},
	}

	func main() {
		fmt.Println(m)
	}
	```

- Üst düzey type yalnızca bir type adıysa, onu değişmezin öğelerinden atlayabilirsiniz.

	```go
	package main

	import "fmt"

	type Vertex struct {
		Lat, Long float64
	}

	var m = map[string]Vertex{
		"Bell Labs": {40.68433, -74.39967},
		"Google":    {37.42202, -122.08408},
	}

	func main() {
		fmt.Println(m)
	}
	```

- Map öğresi mutable bir tiptir. Tanımlanmış bir map üzerine ekleme, silme, güncelleme ve kontrol yapılabilir. `v, ok := m["Answer"]` ifadesinde eğer bir veri varsa ok true döner.

	```go
	package main

	import "fmt"

	func main() {
		m := make(map[string]int)

		m["Answer"] = 42 // set
		fmt.Println("The value:", m["Answer"])

		m["Answer"] = 48 // re-set
		fmt.Println("The value:", m["Answer"])

		delete(m, "Answer") // delete
		fmt.Println("The value:", m["Answer"])

		v, ok := m["Answer"] // has?
		fmt.Println("The value:", v, "Present?", ok)
	}
	```

- Fonksiyonlar birer değişkendir, ve fonksiyonlar fonksiyon girişi alabilir.

	```go
	package main

	import (
		"fmt"
		"math"
	)

	func compute(fn func(float64, float64) float64) float64 {
		return fn(3, 4)
	}

	func main() {
		hypot := func(x, y float64) float64 {
			return math.Sqrt(x*x + y*y)
		}
		fmt.Println(hypot(5, 12))

		fmt.Println(compute(hypot))
		fmt.Println(compute(math.Pow))
	}
	```

- Go işlevleri closures olabilir. Closure, değişkenleri gövdesinin dışından referans alan bir işlev değeridir. İşlev, başvurulan değişkenlere erişebilir ve bunları atayabilir; bu anlamda fonksiyon değişkenlere "bağlıdır". Örneğin, toplayıcı işlevi bir kapatma döndürür. Her kapanış kendi toplam değişkenine bağlıdır.

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```