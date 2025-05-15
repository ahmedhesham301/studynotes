# Interfaces 
An interface in go works by defining a function with return type or arguments or both or an interface and if a type has that function/interface then it belongs to that interface.
Example:
```go
type bot interface {
	getGreetings() string
}

type englishBot struct{}

type spanishBot struct{}
```

```go
func (eb englishBot) getGreetings() string {
	// custom logic
	return "Hi there"
}

func (sb spanishBot) getGreetings() string {
	// different custom logic
	return "Hola amigo"
}
```

```go
func printGreetings(b bot) {
	fmt.Println(b.getGreetings())
}
```
>Explanation: 
>both englishBot and spanishBot has the function `getGreetings() string`
>the bot interface has the function `getGreetings() string`
>so both of them belong to the bot type therefore if we declare a function a bot as a parameter it will work on englishBot and spanishBot

>To specify more on what belong to an interface you can add more functions to it.