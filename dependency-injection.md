# Dependency Injection

Inject custom services registered with ASP.NET Core's dependency injection container into any `HttpHandler`.

## Register Services

First, define and register your custom service using [add_service](/docs/host.html#add_service).

```fsharp
module Program = 
    open Falco
    open Falco.Markup.Elem
    open Falco.Markup.Text

    type IAdder =
        abstract member Add : int -> int -> int

    type AddSomeMore(number) =
        interface IAdder with
            member this.Add first second = 
                first + second + number

    let handler : HttpHandler = // ... See below

    [<EntryPoint>]
    let main args =

        webHost args {
            add_service (fun services ->
                .AddTransient<IAdder>(fun _ -> AddSomeMore(DateTime.Now.Minute)))

            endpoints
            [ 
                get "/" handler
            ]
        }
    0
```

## Inject Dependencies in `HttpHandler`

Now use `Falco.Services.inject` to inject one or more services into your `HttpHandler`.

```fsharp
module Program = 
    open Falco
    open Falco.Markup.Elem
    open Falco.Markup.Text

    type IAdder = // ... See above

    // ...

    let handler: HttpHandler =
        Services.inject<IAdder> (fun adder -> fun ctx ->
            let sum = adder.Add 1 2

            let template = div [] [ 
                h1 [] [ raw $"The sum is {sum}" ] ]

            Response.ofHtml template ctx
        )

    [<EntryPoint>]
        let main args = // ...
```

You can inject multiple services by specifying additional generic arguments on `Services.inject`:

```fsharp
let handler: HttpHandler =
    Services.inject<IAdder, IMultiplier, IDivider> (fun adder -> fun multiplier -> fun divider -> fun ctx ->
        let sum = adder.Add 1 2
        let product = multiplier.Multiply 1 3
        let quotient = divider.Divide 100 7

        // ...
    )
```