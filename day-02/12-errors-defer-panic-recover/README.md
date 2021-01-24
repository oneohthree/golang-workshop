# Manejo de errores: error, defer, recover, panic


## Manejo de errores en Golang

En lugar de excepciones como Python, Java o C#, Golang sigue un
enfoque más cercano a los lenguajes funcionales donde el estado de
error es representado por un tipo de datos,

```
var DivisionByZero = errors.New("Division por cero")

func safedivide(a int, b int) (int, error) {
	if b == 0 {
		return 0, DivisionByZero
	}
	return a / b, nil
}
```

El tipo `error` es una interfaz *built-in* del lenguaje

```go
type error interface {
    Error() string
}
```

Esto nos permite crear nuestros propios tipos de error

```go
type MyError struct {
	message: string
	status: int
}

func (m *MyError) Error() string {
	return fmt.Sprintf("Error - %s. Status %d", m.message, m.status)
}

func returnMyError() error {
	return &MyError{
	 message: "None",
	 status : -1,
	}
}
```

El paquete `errors` contiene funciones dedicadas a manejar tipos de
error. Podemos usar la función `errors.Is` para verificar si un error
es de un tipo específico.

```go
if _, err := os.Open("non-existing"); err != nil {
    if errors.Is(err, os.ErrNotExist) {
        fmt.Println("file does not exist")
    } else {
        fmt.Println(err)
    }
}
```

## Defer

La sentencia `defer` se utiliza para indicar que la función a
continuación se debe ejecutar a la salida de la función actual

Es muy usual en Golang usar `defer` para destruir o liberar cualquier
tipo de recursos temporal que se obtenga en la función.

```go

func doSomething() {
 a := getExternalResource();
 defer a.Release()
 /// Resto de la función
}

```

`defer` se ejecuta sin importar las causas por las que la función
actual haya terminado, lo que garantiza en el ejemplo anterior que
`a.Release()` siempre se ejecute.

## Panic y recover

Golang incluye una función especial `panic` para indicar un error que
no puede ser manejado de forma correcta.

La contraparte de `panic` es la función `recover`, que verifica si
ocurrió una llamada a `panic` en el contexto de la función actual.

Si una llamada a `panic` no es seguida por un `recover` la función
termina y el contexto pasa al invocador. Esto continúa hasta que se
encuentre un recover o se llegue a la función `main` en cuyo caso el
programa se detendrá con un mensaje de error.

```go
func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

La función `f` usa `defer` y `recover` para garantizar que la llamada
a `panic` en `g` no interrumpa al programa.


## Panic y recover no son try y catch

Uno de los errores más comunes para los que se inician en Golang es
pensar en `panic` y `recover` como una alternativa a los bloques
`try-catch` que aparecen en otros lenguajes. Esto es considerado una
mala práctica.

La función `panic` se debe utilizar solo para indicar estados en el
flujo de una aplicación para los cuales no hay solución efectiva. Por
otro lado `recover` debería utilizarse para liberar o destruir
recursos adquiridos por la aplicación antes de hacer una salida forzosa.