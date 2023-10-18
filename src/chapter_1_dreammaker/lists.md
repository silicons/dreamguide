# Lists

BYOND does not have a difference between an 'array' and a 'dictionary'; its only collection datatype at time of writing is a list.

- A list is created with `list(...)`, where everything in `...` is put into the list.
- A list has a specific length, accessible with `length(L)` or `L.len` where 'L' is the list.
- A list is an ordered collection of data with an arbitrary (any) type.
- Data in the list can be associated to a value - as long as it's not a number.
- **BYOND lists are 1-indexed.** This means that `L[1]` is the first element, **not** `L[0]`.
- Supplemental reading: [BYOND Reference on lists](http://www.byond.com/docs/ref/#/list). Make sure to read this if something is going wrong.

```dm
// create an empty list
var/list/L = list()

// reassign the variable L, which is of type /list, to a **new** list that has 'c' and 'd' in it.
L = list("c", "d")

// associate 'a' with 'b'
L["a"] = "b"
// associate a typepath with something else
L[/obj/gun] = "example"
// associate text with numbers
L["one"] = 1
// even 'null' can be associated, as it is not a number
L[null] = "Actually works"

// outputs a JSON encoded output of the list to world
world << json_encode(L)

// the output will be
//
// {"c", "d", "a": "b", "/obj/gun": "example", "one": 1, null: "Actually works"}
//
// notice how paths are turned into strings, but nulls are not, as null is a valid data-type
// in Javascript, and json stands for JavaScript Object Notation

// illegal: numbers are treated as list indices, and may not be associative keys
L[55] = "fifty-five" // Runtime Error: List index out of bounds

```

## Operators

- Add to list: `L.Add(...)`, `L += ...` where `...` is some data, or a list.
    - Adding a list to a list will automatically 'unwrap' the list, meaning the list itself isn't added, but rather the contents.
    - This unwrapping is not recursive, meaning a list of lists will have the lists inside it added to the first list.
- Remove from list: `L.Remove(...)`, `L -= ...` where `...` is some data, or a list.
    - Same rules as above.
    - **Matching**: To determine if something matches a list entry (and therefore should be removed), `==` operator is implicitly used, so if something is equal under that, it will be removed.
- Distinct add to list: `L |= ...`, where `...` is some data, or a list.
    - Same rules as both of the above.
    - Unlike add, this will not add something twice; `L |= "a"`, `L |= "a"` will only result in one copy of `"a"` in `L`.
- **Get** list length: `length(L)`, `L.len`
- **Modify** list length: `L.len = #` where `#` is a number.
    - If this shrinks the list, elements are dropped as needed.
    - If this grows the list, the new elements are initialized to `null`.
- Check if things are in a list: `thing in L`, `L.Find(thing)`
    - This is a bit different: `Find` gets the index if found, and `0` otherwise, while `in` is just a binary truth-y operator that returns `1` (`TRUE`) or `0` (`FALSE`).
    - **There is a faster way to do this, if you are using list associations** - see next operator.
- Indexing operator `[]`: `L[#]` where `#` is a number for index access, `L[thing]` where `thing` is non-numerical data for key-value access.
    - Index access is self-explanatory. There will be a runtime error if the number is out of bounds (so not `1` to `L.len`)
    - Keyed access gets the **associated value of the key, or `null` if it is not there.**
        - Make sure your code handles `null` properly; If you intentionally set things to `null`, it's the same as dropping the association, which can cause problems if you are also using it to track if something is in the list, but `null` both means 'was there, and set to null', and 'was never there'.
        - **This is a far faster way of checking existence if something is not mapped to null.** The 'why' will be explained in chapter 4, but `!isnull(L[key])` is miles faster than `L.Find(key)` or `key in L`.
- Checking if two lists are **key/index-only shallow-equivalent** (different from equal): `L1 ~= L2`
    - Checks if two lists have **indexes/keys** that are the same that are on another list.
    - **The values are not checked.** `list("a", "b" = "c")` is **shallow-equivalent** to `list("a", "b" = "d")`.
- **Copy** a list: `L.Copy()`
    - This is a **key-value shallow copy** of the list. `list("a" = "b")` will properly copy over.
    - This is a **shallow** copy because it does not **clone** reference-types; this is harmless for numbers, strings, paths, and other constant data-types, but **potentially problematic on reference types**, including `/list`, `/obj`, and anything you create with `new`.
    - Reference types just have their references copied - so both lists now point ot the same object. See 'Referential Equality' down below.
- **Insert** a series of elements in a specific position of a list: `L.Insert(#, ...)` where `...` is some data or a list and `#` is the index.
    - Anything at the index and after it will be pushed back to make room: `list("a", "b", "c").Insert(2, "d")` will become `["a", "d", "b", "c"]`.
- **Swap** the elements at two positions of a list: `L.Swap(i1, i2)`, where `i1`, `i2` are the two positions.
- **Cut** a section of a list: `L.Cut(Start=1, End=0)`.
    - The `=` notation in the args mean 'default' arguments. By default, Cut() cuts the first element, all the way to the last element (0).
    - The first index is the first position to start cutting at, the second index is **one after** the last element to cut. To cut one thing, you would do `.Cut(n, n + 1)`.
    - This is list cutting a ribbon and glueing the ends back together: `list("a", "b", "c", "d").Cut(2, 4)` becomes `["a", "d"]`.
    - Read the [BYOND Reference](http://www.byond.com/docs/ref/#/list/proc/Cut) on `Cut()` to learn more.
- **Splice**: Read the [BYOND Reference](http://www.byond.com/docs/ref/#/list/proc/Splice) on `Splice()`, as it's a slight bit more advanced.

## List ~~Comprehensions~~ Constructions

You can combine two lists using certain set-like operators (this guide will not explain set theory).

- `L1 + L2` gives you a new list that's just putting `L2` to the end of `L1`
- `L1 - L2` just subtracts `L2` from `L1` and returns a new list
- `L1 & L2` gives you a new list with elmeents that are in both lists.
- `L1 | L2` gives you a new list with elements that are in either list, **without duplicating the elements if they are in both.**
- `L1 ^ L2` gives you a new list with elements that are in one list or the other, **but not both**.

These work with `L1 &= L2`, where `&` can be `+`, `-`, `&`, `|`, `^`; this basically modifies `L1` to be the result of what you would get from the above operators, but without making a new list.

## Referential Equality

This page talks about **equivalence** vs **equality**, and **new list** vs **modify the existing list**.
We'll get into it more but, the simple way of explaining this, is that every list is a distinct object.
`==` cannot equate list values as `==` on any **reference** datatype checks for **referential equality**, not **content equivalence**.

As a new coder, one of the biggest footguns you will experience with lists is **unintentionally sharing lists.**
If you share a list across two objects without meaning to, any changes to the list will be seen by both objects accessing the list.
