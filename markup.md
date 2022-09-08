# Markup

[![NuGet Version](https://img.shields.io/nuget/v/Falco.Markup.svg)](https://www.nuget.org/packages/Falco.Markup)
[![build](https://github.com/pimbrouwers/Falco/actions/workflows/build.yml/badge.svg)](https://github.com/pimbrouwers/Falco/actions/workflows/build.yml)

A core feature of Falco is the XML markup module. It can be used to produce __any__ form of angle-bracket markup (i.e. HTML, SVG, XML etc.) within a Falco project and is also available directly as a standalone [NuGet](https://www.nuget.org/packages/Falco.Markup) package

_All_ of the standard HTML tags & attributes have a functional representation that produces objects to represent the HTML node. Nodes are either:

- `Text` which represents `string` values. (Ex: `Text.raw "hello"`, `Text.rawf "hello %s" "world"`)
- `SelfClosingNode` which represent self-closing tags (Ex: `<br />`).
- `ParentNode` which represent typical tags with, optionally, other tags within it (Ex: `<div>...</div>`).

## HTML

The benefits of using the Falco markup module as an HTML engine include:

- Writing your views in plain F#, directly in your assembly.
- Markup is compiled alongside the rest of your code, leading to improved performance and ultimately simpler deployments.


### Strongly-typed views

```fsharp
open Falco.Markup

type Person =
    { FirstName : string
      LastName : string }

let doc (person : Person) =
    Elem.html [ Attr.lang "en" ] [
        Elem.head [] [
            Elem.title [] [ Text.raw "Sample App" ]
        ]
        Elem.body [] [
            Elem.main [] [
                Elem.h1 [] [ Text.raw "Sample App" ]
                Elem.p [] [ Text.rawf "%s %s" person.First person.Last ]
            ]
        ]
    ]
```

Rumor has it that the Falco [creator](https://twitter.com/pim_brouwers) makes a dog sound every time he uses `Text.rawf`.

> **Note**: I am the creator, and this is entirely true.

### Combining views to create complex output

```fsharp
open Falco.Markup

// Components
let divider =
    Elem.hr [ Attr.class' "divider" ]

// Template
let master (title : string) (content : XmlNode list) =
    Elem.html [ Attr.lang "en" ] [
        Elem.head [] [
            Elem.title [] [ Text.raw "Sample App" ]
        ]
        Elem.body [] content
    ]

// Views
let homeView =
    master "Homepage" [
        Elem.h1 [] [ Text.raw "Homepage" ]
        divider
        Elem.p [] [ Text.raw "Lorem ipsum dolor sit amet, consectetur adipiscing."]
    ]

let aboutView =
    master "About Us" [
        Elem.h1 [] [ Text.raw "About" ]
        divider
        Elem.p [] [ Text.raw "Lorem ipsum dolor sit amet, consectetur adipiscing."]
    ]
```

## SVG

The vast majority of the SVG spec has been mapped to element and attributes functions. There is also an SVG template to help initialize a new drawing with a valid viewbox.

```fsharp
open Falco.Markup
open Falco.Markup.Svg

// https://developer.mozilla.org/en-US/docs/Web/SVG/Element/text#example
let svgDrawing =
    Templates.svg (0, 0, 240, 80) [
        Elem.style [] [
            Text.raw ".small { font: italic 13px sans-serif; }"
            Text.raw ".heavy { font: bold 30px sans-serif; }"
            Text.raw ".Rrrrr { font: italic 40px serif; fill: red; }"
        ]
        Elem.text [ Attr.x "20"; Attr.y "35"; Attr.class' "small" ] [ Text.raw "My" ]
        Elem.text [ Attr.x "40"; Attr.y "35"; Attr.class' "heavy" ] [ Text.raw "cat" ]
        Elem.text [ Attr.x "55"; Attr.y "55"; Attr.class' "small" ] [ Text.raw "is" ]
        Elem.text [ Attr.x "65"; Attr.y "55"; Attr.class' "Rrrrr" ] [ Text.raw "Grumpy!" ]
    ]

let svg = renderNode svgDrawing
```

## Merging Attributes

The markup module allows you to easily create components, an excellent way to reduce code repetition in your UI. To support runtime customization, it is advisable to ensure components (or reusable markup blocks) retain a similar function "shape" to standard elements. That being, `XmlAttribte list -> XmlNode list -> XmlNode`.

This means that you will inevitably end up needing to combine your predefined `XmlAttribute list` with a list provided at runtime. To facilitate this, the `Attr.merge` function will group attributes by key, and concat the values in the case of `KeyValueAttribute`.

```fsharp
open Falco.Markup

// Components
let heading (attrs : XmlAttribute list) (content : XmlNode list) =
    // safely combine the default XmlAttribute list with those provided
    // at runtime
    let attrs' =
        Attr.merge [ Attr.class' "text-large" ] attrs

    Elem.div [] [
        Elem.h1 [ attrs' ] content
    ]

// Template
let master (title : string) (content : XmlNode list) =
    Elem.html [ Attr.lang "en" ] [
        Elem.head [] [
            Elem.title [] [ Text.raw "Sample App" ]
        ]
        Elem.body [] content
    ]

// Views
let homepage =
    master "Homepage" [
        heading [ Attr.class' "red" ] [ Text.raw "Welcome to the homepage" ]
        Elem.p [] [ Text.raw "Lorem ipsum dolor sit amet, consectetur adipiscing."]
    ]

let homepage =
    master "About Us" [
        heading [ Attr.class' "purple" ] [ Text.raw "This is what we're all about" ]
        Elem.p [] [ Text.raw "Lorem ipsum dolor sit amet, consectetur adipiscing."]
    ]
```


## Custom Elements & Attributes

Every effort has been taken to ensure the HTML and SVG specs are mapped to functions in the module. In the event an element or attribute you need is missing, you can either file an [issue](https://github.com/pimbrouwers/Falco/issues), or more simply extend the module in your project.


An example creating custom XML elements and using them to create a structured XML document:

```fsharp
open Falco.Makrup

module Elem =
    let books = Elem.create "books"
    let book = Elem.create "book"
    let name = Elem.create "name"

module Attr =
    let soldOut = Attr.createBool "soldout"

let xmlDoc =
    Elem.books [] [
        Elem.book [ Attr.soldOut ] [
            Elem.name [] [ Text.raw "To Kill A Mockingbird" ]
        ]
    ]

let xml = renderXml xmlDoc
```