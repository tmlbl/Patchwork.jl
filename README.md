# Patchwork

[![Build Status](https://travis-ci.org/shashi/Patchwork.jl.svg?branch=master)](https://travis-ci.org/shashi/Patchwork.jl)

A library for representing the [DOM](http://www.w3.org/TR/WD-DOM/introduction.html) in Julia. It supports [element creation](#creating-elements), [diff computation](#diff-computation) and [browser-side patching](#javascript-setup-and-patching) for efficient re-rendering.

## Setup

From the REPL, run
```julia
Pkg.add("Patchwork")
```

## Creating Elements

The `Elem` constructor can be used to create an element.

```julia
# E.g.
using Patchwork

Elem(:h1, "Hello, World!")
```
creates an `h1` heading element which reads "Hello, World!"

You can attach any property (e.g. `className`, `style`, `height`, `width`) that you would like the DOM node to have by passing it as a keyword argument to `Elem`

```julia
# E.g.
Elem(:h1, "Hello, World!", className="welcome", style=[:color => :white, :backgroundColor => :black])
```
This creates a `h1` with white text on a black background.

You can of course nest elements inside another
```julia
Elem(:div, [
    Elem(:h1, "Hello, World!"),
    Elem(:p, "How are you doing today?")])
```

The `Patchwork.HTML5` module gives you helper functions which are named after the HTML elements, allowing you to create nodes in a concise DSL-esque way.

```julia
div(
    h1("Hello, World!", style=[:color=>:green]),
    p("How are you doing today?"))
```

`Elem` objects are immutable, they have a `children` field which is an immutable vector and an `attributes` field which is an immutable hash map. There are some infix operators defined for `Elem`.

The `&` operator can set attributes
```julia
# E.g.
div_with_class = div("This div's class can change") & [:className => "shiny"]
```
The `<<` operator can append an element to the end of another.

```julia
h1_and_p = div(h1("Hello, World!")) << p("How are you doing today?")
```

Similar to the `Patchwork.HTML` there is also a `Patchwork.SVG` module which provides helper functions to construct and embed SVG documents.

Here is a `@manipulate` statement waiting to be played with!
```julia
using Interact, Patchwork.SVG
@manipulate for r=1:100, cx = 1:500, cy=1:400, color=["orange", "green", "blue"]
    svg(circle(cx=cx, cy=cy, r=r, fill=color), width=500, height=500)
end
```

## Diff computation

Patchwork provides a `diff` method to compute the difference between two elements.

```julia
# E.g.
patch = diff(left::Elem, right::Elem)
```
The above call to diff returns a "patch". A patch is a `Dict` which maps node indices to a list of patches on that node. The node index is a number representing the position of the node in a depth-first ordering starting at the root node (here `left`), whose index is 0.

`Elem`s are based on immutable datastructures. `&` and `<<` operations return new `Elem`s, which may share structure with the operands. The more structure two nodes share, the faster the diffing.

For example, if you have a big `Elem`, say `averybigelem`, the running time of the following diff call

```julia
diff(averybigelem, averybigelem & [:className => "shiny"])
```

will not depend on the size and complexity of `averybigelem` because diffing gets *short-circuited* since `left.children === right.children`. It will probably be helpful to keep this in mind while building something with Patchwork.

## JavaScript setup and patching

Patchwork has a javascript "runtime" in `runtime/build.js` that needs to be included into a page where you would like to display Patchwork nodes.

```html
<script src="/path/to/build.js"></script>
```

This is automatically done for you if you are using Patchwork from IJulia.

Patchwork defines the `writemime(io::IO, ::MIME"text/html", ::Elem)` method which can use this runtime to display nodes and/or apply patches to nodes that are already displayed.

At a lower level, the runtime exposes the `window.Patchwork` object, which can be used to render nodes from their JSON representations and also apply patches.

```js
// E.g.
node = new Patchwork.Node(mountId, elemJSON)
```
this renders the node represented by `elemJSON` and appends it to a DOM element with id `mountId`.

`Patchwork.Node` instances have an `applyPatch` method which can be used to patch the node.

```js
// E.g.
node.applyPatch(patchJSON)
```

## With Compose and Gadfly

If Patchwork is installed, interactive plots or Compose graphics automatically use Patchwork to efficiently render them into SVG Virtual DOM. Any updates to the plot get turned into patches, sent over to the browser and applied to the plot.

## Usage in IJulia

When you load Patchwork in IJulia, the runtime is setup automatically for you. If the result of executing a cell is an `Elem` object, it gets rendered in the cell's output area. `display(::Elem)` will work too.

When used with [Reactive](http://julialang.org/Reactive.jl) (or Interact), any `Signal{Elem}` values (see [Reactive.Signal](http://julialang.org/Reactive.jl/#signals)) get displayed with its initial value first. Subsequent updates are sent as patches and applied at the front-end.

## Development

You will need a recent `nodejs` and `npm` installed to hack on the JavaScript part of this package.

To build the JS files run the following from `runtime/` directory:

```sh
npm install .
npm install -g browserify
npm install -g uglifyjs
make
```

## Thanks

* This package is largely based on [Matt-Esch](https://github.com/Matt-Esch)'s excellent [virtual-dom](https://github.com/Matt-Esch/virtual-dom) and [vtree](https://github.com/Matt-Esch/vtree) JavaScript modules. Patchwork's JS runtime makes use of virtual-dom and virtual-hyperscript by [Raynos](https://github.com/Raynos).
* Thanks to [Evan Czaplicki](https://github.com/evancz) for creating [Elm](http://elm-lang.org/) which inspired me to take the FRP road.

