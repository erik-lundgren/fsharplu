## FSharp.Json

Newtonsoft's Json.Net is the library of choice for Json serialization. Unfortunately
the built-in converters generate rather verbose and ugly Json for certain F# data types like
discriminated unions and option types. The module `Microsoft.FSharpLu.Json` provides more succinct 
serialization for those types.

To use our serializer open the module with

    open Microsoft.FSharpLu.Json

To serialize an object using the default Json.Net format you can use
`Default.serialize`.

### Option type

The following examples shows how JSon.net serializes 
a simple value of type `(int option) list`:

```Json
Default.serialize [Some 5; None; Some 6]

val it : string = "
[
    {
        "Case": "Some",
        "Fields": [ 5 ]
    },
    null,
    {
        "Case": "Some",
        "Fields": [ 6 ]
    }
]"
```

Using the compact serializer provided by FSharpLu.Json the same objects
gets serialized instead as a one-liner heterogeneous array:

```Fsharp 
Compact.serialize [Some 5; None; Some 6]

val it : string = "[ 5, null, 6 ]"
```

### Discriminated unions
    
Now let's take a look at simple field-less discriminated unions. 
Take for instance the type `type SimpleDu = Foo | Bar`. 
The value `Foo` gets serialized by Json.Net as follows:

| Value | Default (Json.net) | Compact
| ----- | ------------------ | ----------
| `Foo` |  `{  "Case": "Foo" }` |  `"Foo"`| 


### Discriminated unions with fields

Our serializer also supports generic discrimnated unions with fields for instance take the following
binary Tree example:

```FSharp
type 'a Tree = Leaf of 'a | Node of 'a Tree * 'a Tree
let x = Node (Node((Leaf 1), (Leaf 4)), Leaf 6)
```

Default Json.net serialization:

```FSharp
Default.serialize x

val it : string = "
{
    "Case": "Node",
    "Fields": [
        {
            "Case": "Node",
            "Fields": [
                    {
                        "Case": "Leaf",
                        "Fields": [ 1 ]
                    },
                    {
                        "Case": "Leaf",
                        "Fields": [ 4 ]
                    }
                ]
        },
        {
            "Case": "Leaf",
            "Fields": [ 6 ]
        } ]
}"
```

where FSharpLu.Json produces the more succinct and easier to read:
 
```Json
Compact.serialize x

val it : string = "
{
    "Node": [
        {
            "Node": [
                { "Leaf": 1 },
                { "Leaf": 4 }
            ]
        },
        { "Leaf": 6 }
    ]
}"
```

### Backward compatibility with Json.Net

FSharpLu.Json incldues a third serializer called `BackwardCompatible`. While it produces the same
Json as the compact serializer it can deserialize Json in both compact format as well as the
default format produced by the stock JSon.net serializer.

This is helpful when migrating projects from Json.Net to FSharpLu.Json as it lets you deserialize Json that 
was produced by earlier versions of your code.

Expressed more formally this serializer verifies the following properties:

- `BackwardCompatible.serialize` = `Compact.serialize`
- `Default.serialize >> BackwardCompatible.deserialize` = `id`
- `Compact.serialize >> BackwardCompatible.deserialize` = `id`
