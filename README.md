# TypeSkin

A lightweight, drop-in layer on top of JavaScript which gives it an expressive type system with Haskell-inspired invariants.

## Static type checks

The basic idea is that, when declaring a JavaScript value or function, instead of doing it as usual:

```javascript
const inc = function(a) {
  return "1" + x;
}
```

You can, optionally, wrap it with a type annotation:

```javascript
// A function from numbers to numbers
const inc = T(T.Fn(T.Number, T.Number), // inc :: Number -> Number
  function(x) {
    return "1" + x;
  });
```

And then run (i.e., "compile") your module with `node myLib.js`. TypeSkin will statically check your declarations:

![type mismatch](images/error0.png?raw=true)

That error is shown because `"1" + x` is a `String`, not a `Number`, as annotated. Once everything checks, your program behaves exactly the same as if TypeSkin wasn't there at all. And that's it!

## Rich type language

TypeSkin allows you to define rich types, using an expressive dialect. Here is, for example, a type for Pokémon:

```javascript
const T = require("TypeSkin");

// A Pokémon Type
const Type = T.Enum(
  "normal" , "fight"   , "flying" , "poison"   ,
  "ground" , "rock"    , "bug"    , "ghost"    ,
  "steel"  , "fire"    , "water"  , "grass"    ,
  "dragon" , "psychic" , "ice"    , "electric" ,
  "dark"   , "fairy"
).__name("Type");

// A Pokémon Stat
const Stat = T.Enum(
  "hp"  , "atk" , "def",
  "spe" , "spa" , "spd"
).__name("Stat");

// A Pokémon
const Pokemon = T.Struct({
  name: T.String,
  number: T.Uint16,
  types: T.Pair(Type, T.Maybe(Type)),
  attacks: T.Vector(4, T.String),
  stats: T.Struct({
    hp: T.Uint8,
    atk: T.Uint8,
    def: T.Uint8,
    spe: T.Uint8,
    spa: T.Uint8,
    spd: T.Uint8
  })
}).__name("Pokemon");

// A function that receives a Pokémon and returns an Uint8
const highestStat = T(
  T.Fn(Pokemon, T.Uint8),
  function(poke) { 
    return Math.max(
      poke.stats.hp,
      poke.stats.atk,
      poke.stats.spe,
      poke.stats.spa,
      poke.stats.spd); 
  })
```

Notice the presence of enums, several integer types, pairs, structs, a fixed-size vector and even Haskell's `Maybe`. Types are plentiful and everything is checked statically. Try "compiling" the file above and changing some bits!

## Invariants

The code above, while well-typed, is wrong. Let's add another line to it:

```javascript
T.forall([Pokemon, Stat], (poke, stat) => highestStat(poke) >= poke.stats[stat]);
```

This declaration says that, for every Pokémon, we expect that its `highestStat` is at least as large as any of its stats - which is obvious, isn't it? Let's "compile" that file again:

![invariant violation](images/error1.png?raw=true)

Woops! It is telling us that our invariant doesn't hold for the `def` stat of `ketecape`, a Pokémon randomly generated from its type, which happens to have its `def` as the highest stat. **Seems like we forgot to add `poke.stats.def` to our `highestStat` function!** Invariants are very similar to tests, as they spot mistakes even on well-typed code. They're easy to write and can be very specific, providing a powerful sense of safety.

## Learning

The best way to learn TypeSkin is by following the [TypeSkin game](game.js).

## Alternatives

TypeSkin shares similar use-cases with TypeScript, PureScript and such. It is very different, though. It is much simpler and requires less investment. Moreover, it has much more expressive types in general. It, though, **is not a real type system**. All its checks are based on repeatedly calling functions with random samples, so, **passing a TypeSkin check isn't a guarantee of correctness at all**. For most real-world scenarios, though, it does a very good job at spotting the same mistakes a type system would, with minimal programming effort. For a powerful, real type-system, I recommend [Idris](https://www.idris-lang.org).

