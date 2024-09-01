+++
title = 'MapSets in Elixir'
author = "Eric Leohner"
date = 2024-09-01T12:39:57-04:00
draft = false
+++

MapSets are a fundamental Elixir type, but they are often overlooked. When I first encountered them, I thought it was a case of poor naming (it's not, and we'll get to that soon). They are a powerful and important tool in the programming toolbox.

## What is a MapSet?

If you're already familiar with the concept of maps in Elixir, the name `MapSet` might be a bit confusing. On the surface, the structure does not appear to be related to a map. And what is the `Set` part?

### Mathematical Set

**In Plain English**
> A set is a group of unique things.

The most important part of that sentence is arguably the word _unique_.

## More Common Elixir Types

When we look at other familiar types in Elixir, like `List`s and `Map`s, we see that there is no requirement that all of the items have to be different. 

Let's look at some examples:

```elixir
#########
# Lists #
#########

# With a list, we have elements and the order of the elements is important
iex> a = [1, 2]
[1, 2]

iex> b = [2, 1]
[2, 1]

iex> a == b
false

# With a list, we can also have multiple items with the same value
iex> [1, 2, 3, 2, 1]
[1, 2, 3, 2, 1]

```
With lists, we have no guarantee of uniqueness and the order of the items has meaning (that is, it's important).

```elixir
########
# Maps #
########

# With a map, we have elements that have key-value pairs
# Each key item corresponds to some other value item
iex> a = %{a: "A", b: "B"}
%{a: "A", b: "B"}

# The order of the items is also not important
# Note: Elixir tries to make some ordering, which is why the output is different from the input; but this proves that the order is not important
iex> b = %{b: "B", a: "A"}
%{a: "A", b: "B"}

iex> a == b
true

# Maps also make no guarantee of item uniqueness
# They will overwrite existing values without a problem
# Elixir gives us a warning and chucks out any any already-existing duplicate keys
# This might seem like a guarantee of uniqueness, but if we already have a map and we want to add something to it that already happens to exist, we can inadvertently destroy this existing data
iex> c = %{a: "A", b: "B", a: 1}
warning: key :a will be overriden in map
%{a: 1, b: "B"}
```

With maps, we have a taste of uniqueness. But it's a taste that could also prove dangerous if we try to update a map with some key that already exists.

Unlike lists, the collection of data is not distinct units. Instead, we have key-value pairs. And these key-value pairs are absolutely critical for maps to exist. We can't have a maps without that structure.

```elixir
# A map without key-value pairs cannot exist
iex> %{:a, :b}
** (SyntaxError) ... syntax error before: ','
|
| %{:a, :b}
|     ^
```
## `MapSets`

When we look at `MapSet`s, we can see some similarities with both `List`s and `Map`s. For example, the order does not matter, just like a map. But we can also store a group of things without a key-value pair structure.

```elixir
###########
# MapSets #
###########

# In fact, MapSets take a list as an argument
iex> MapSet.new([1, 2, 3])
MapSet.new([1, 2, 3])

# Attempting to create a duplicate element does not throw an error;
# rather, it will just be ignored
iex> a = MapSet.new([1, 2, 3])
MapSet.new([1, 2, 3])

iex> MapSet.put(a, 3)
MapSet.new([1, 2, 3])
```

Because the duplicate element is exactly that, a duplicate, there is no need for a warning or error message. The MapSet remains unchanged.

Now, let's prove that ordering does not matter.

```elixir
iex> a = MapSet.new([1, 2, 3])
MapSet.new([1, 2, 3])

# Just like the map, the mapset is internally ordered;
# this means that the order we gave it does not matter
iex> b = MapSet.new([3, 2, 1])
MapSet.new([1, 2, 3])

iex> a == b
true
```

Aha! We get some benefits of lists and some benefits of maps.

But this begs a question. Why not call it `MapList` or something similar?

This can be answered in two parts, one obvious and one not so obvious:

1. The obvious part: they are called sets in math, so it makes sense to use the word `Set` to describe them.

2. The not-so-obvious part: in the internal workings of Elixir, `MapSet`s are stored as maps. However, they are maps where the keys are the data we store and the values are empty lists.

As with all things, I want you to question what I am saying. And to that end, I want to give you proof that `MapSet`s are stored as map types.

```elixir
# Remember, the mapset gets reordered by Elixir internal ordering,
# which means that order does not matter
iex> a = MapSet.new([1, :a, "Some String", {:tuple}])
MapSet.new([1, :a, {:tuple}, "Some String"])

# The MapSet struct has the `map` field that reveals the underlying data structure
iex> a.map
%{1 => [], :a => [], {:tuple} => [], "Some String" => []}
```

## Why are MapSets Stored as Maps?

I don't want to get into the nitty-gritty of why MapSets use maps internally. However, the short and important answer is:

> `MapSet`s are stored as maps for efficiency reasons.

Maps make it easy to look up data. And the complexity of checking whether a value exists in a map is trivial, as is the complexity of adding a new element to a map.

This answers the question of `Why are MapSets called MapSets`.

## The Takeaway

In short, the are a few critical things to learn from this article:

- `MapSet`s are a data structure in Elixir
- `MapSet`s allow us to store a list of things where:
  - order does not matter
  - each thing in the list is unique
- Internally, Elixir stores `MapSet`s as maps
