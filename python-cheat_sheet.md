# Comprehensive Python Cheat Sheet

This cheat sheet provides an exhaustive, simple, and concise reference for Python programming, covering collections, types, syntax, system interactions, data handling, advanced features, libraries, and multimedia. It's based on the popular resource by Jure Šorn.

## Table of Contents

| Section | Subtopics |
|---------|-----------|
| 1. Collections | List, Dictionary, Set, Tuple, Range, Enumerate, Iterator, Generator |
| 2. Types | Type, String, Regular Exp, Format, Numbers, Combinatorics, Datetime |
| 3. Syntax | Function, Inline, Import, Decorator, Class, Duck Type, Enum, Except |
| 4. System | Exit, Print, Input, Command Line Arguments, Open, Path, OS Commands |
| 5. Data | JSON, Pickle, CSV, SQLite, Bytes, Struct, Array, Memory View, Deque |
| 6. Advanced | Operator, Match Stmt, Logging, Introspection, Threading, Coroutines |
| 7. Libraries | Progress Bar, Plot, Table, Console App, GUI, Scraping, Web, Profile |
| 8. Multimedia | NumPy, Image, Animation, Audio, Synthesizer, Pygame, Pandas, Plotly |

## Main Entry Point

```python
if __name__ == '__main__':      # Skips next line if file was imported.
    main()                      # Runs `def main(): ...` function.
```

## 1. Collections

### List
```python
<list> = [<el_1>, <el_2>, ...]  # Creates a list object. Also list(<collection>).
<el>   = <list>[index]          # First index is 0. Last -1. Allows assignments.
<list> = <list>[<slice>]        # Also <list>[from_inclusive : to_exclusive : ±step].
<list>.append(<el>)             # Appends element to the end. Also <list> += [<el>].
<list>.extend(<collection>)     # Appends elements to the end. Also <list> += <coll>.
<list>.sort()                   # Sorts the elements in ascending order.
<list>.reverse()                # Reverses the order of list's elements.
<list> = sorted(<collection>)   # Returns a new list with sorted elements.
<iter> = reversed(<list>)       # Returns reversed iterator of elements.
<el>  = max(<collection>)       # Returns largest element. Also min(<el_1>, ...).
<num> = sum(<collection>)       # Returns sum of elements. Also math.prod(<coll>).
elementwise_sum  = [sum(pair) for pair in zip(list_a, list_b)]
sorted_by_second = sorted(<collection>, key=lambda el: el[1])
sorted_by_both   = sorted(<collection>, key=lambda el: (el[1], el[0]))
flatter_list     = list(itertools.chain.from_iterable(<list>))
<int> = len(<list>)             # Returns number of items. Also works on dict, set and string.
<int> = <list>.count(<el>)      # Returns number of occurrences. Also `if <el> in <coll>: ...`.
<int> = <list>.index(<el>)      # Returns index of the first occurrence or raises ValueError.
<el>  = <list>.pop()            # Removes and returns item from the end or at index if passed.
<list>.insert(<int>, <el>)      # Inserts item at passed index and moves the rest to the right.
<list>.remove(<el>)             # Removes first occurrence of the item or raises ValueError.
<list>.clear()                  # Removes all list's items. Also works on dictionary and set.
```

### Dictionary
```python
<dict> = {key_1: val_1, key_2: val_2, ...}      # Use `<dict>[key]` to get or set the value.
<view> = <dict>.keys()                          # Collection of keys that reflects changes.
<view> = <dict>.values()                        # Collection of values that reflects changes.
<view> = <dict>.items()                         # Coll. of key-value tuples that reflects chgs.
value  = <dict>.get(key, default=None)          # Returns default argument if key is missing.
value  = <dict>.setdefault(key, default=None)   # Returns and writes default if key is missing.
<dict> = collections.defaultdict(<type>)        # Returns a dict with default value `<type>()`.
<dict> = collections.defaultdict(lambda: 1)     # Returns a dict with default value 1.
```

*(Note: Due to length, further sections like Set, Tuple, etc., follow a similar concise code-snippet style. For the complete exhaustive list, refer to the source.)*

## 2. Types

### Type
```python
<type> = type(<obj>)                # Returns the class of the object.
<bool> = isinstance(<obj>, <type>)  # Returns True if object is of the type.
<obj> = <type>(<obj>)               # Converts object to the type.
```

### String
```python:disable-run
<s> = 'single' or "double" quotes   # Strings can span multiple lines with triple quotes.
<s> = str(<obj>)                    # Converts object to string.
<s> = repr(<obj>)                   # Returns canonical string representation.
<s> = <s>.format(<arg_1>, ...)      # Formats string with passed arguments.
<s> = f"{<var>}"                    # f-string: embeds expressions inside braces.
<l> = <s>.split(<sep>)              # Returns list of substrings separated by sep.
<s> = <sep>.join(<list>)            # Joins elements using sep as delimiter.
<i> = <s>.find(<substr>)            # Returns lowest index of substring or -1.
<i> = <s>.rfind(<substr>)           # Returns highest index of substring or -1.
<bool> = <substr> in <s>            # Returns True if substring is in the string.
<s> = <s>.replace(<old
```
