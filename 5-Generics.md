# Generics

- Generics 1.18 versiyonuyla gelmiştir.

- Go fonksiyonları, tür parametreleri kullanılarak birden çok tür üzerinde çalışacak şekilde yazılabilir. Bir işlevin tür parametreleri, işlevin bağımsız değişkenlerinden önce parantezler arasında görünür.

    ```go
    func Index[T comparable](s []T, x T) int
    ```

- Bu bildirim, s'nin, karşılaştırılabilir yerleşik kısıtlamayı karşılayan herhangi bir T türünden bir dilim olduğu anlamına gelir. x de aynı türden bir değerdir. karşılaştırılabilir, türün değerleri üzerinde == ve != operatörlerinin kullanılmasını mümkün kılan kullanışlı bir kısıtlamadır. Bu örnekte, bir eşleşme bulunana kadar bir değeri tüm dilim öğeleriyle karşılaştırmak için kullanıyoruz. Bu İndeks işlevi, karşılaştırmayı destekleyen herhangi bir tür için çalışır.

    ```go
    package main

    import "fmt"

    // Index returns the index of x in s, or -1 if not found.
    func Index[T comparable](s []T, x T) int {
        for i, v := range s {
            // v and x are type T, which has the comparable
            // constraint, so we can use == here.
            if v == x {
                return i
            }
        }
        return -1
    }

    func main() {
        // Index works on a slice of ints
        si := []int{10, 20, 15, -10}
        fmt.Println(Index(si, 15))

        // Index also works on a slice of strings
        ss := []string{"foo", "bar", "baz"}
        fmt.Println(Index(ss, "hello"))
    }
    ```

- Generic methodlara ek olarak Go, generic type'larıda de destekler. Bir tür, generic veri yapılarını uygulamak için yararlı olabilecek bir tür parametresiyle parametrelendirilebilir. Bu örnek, herhangi bir tür değeri tutan tek bağlantılı bir liste için basit bir tür bildirimini gösterir. Bir alıştırma olarak, bu liste uygulamasına bazı işlevler ekleyin.

    ```go
    package main

    // List represents a singly-linked list that holds
    // values of any type.
    type List[T any] struct {
        next *List[T]
        val  T
    }

    func main() {
    }
    ```