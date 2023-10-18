# Lists

BYOND does not have a difference between an 'array' and a 'dictionary'; its only collection datatype at time of writing is a list.

- A list is created with `list(...)`, where everything in `...` is put into the list.
- A list has a specific length, accessible with `length(L)` or `L.len` where 'L' is the list.
- A list is an ordered collection of data with an arbitrary (any) type.
- Data in the list can be associated to a value - as long as it's not a number.
- **BYOND lists are 1-indexed.** This means that `L[1]` is the first element, **not** `L[0]`.
- Supplemental reading: [BYOND Reference on lists](http://www.byond.com/docs/ref/#/list/operators). Make sure to read this if something is going wrong.

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
    - If this grows the list, the new elements are initializde to `null`.
- Check if things are in a list: `thing in L`, `L.Find(thing)`
    - This is a bit different: `Find` gets the index if found, and `0` otherwise, while `in` is just a binary truth-y operator that returns `1` (`TRUE`) or `0` (`FALSE`).
    - **There is a faster way to do this, if you are using list associations** - see next operator.
- Indexing operator `[]`: `L[#]` where `#` is a number for index access, `L[thing]` where `thing` is non-numerical data for key-value access.
    - Index access is self-explanatory. There will be a runtime error if the number is out of bounds (so not `1` to `L.len`)
    - Keyed access gets the **associated value of the key, or `null` if it is not there.**
        - Make sure your code handles `null` properly; If you intentionally set things to `null`, it's the same as dropping the association, which can cause problems if you are also using it to track if something is in the list, but `null` both means 'was there, and set to null', and 'was never there'.
        - **This is a far faster way of checking existence if something is not mapped to null.** The 'why' will be explained in chapter 4, but `!isnull(L[key])` is miles faster than `L.Find(key)` or `key in L`.