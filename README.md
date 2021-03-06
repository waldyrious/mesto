# mesto

MEmory STOrage

## About

Mesto is a in-memory only storage, intended for use in Rich Browser
Applications.


## Usage

### Basic

This is fairly simple `assoc-in` call, almost identical to one from
`clojure.core`, but operating on an atom (here the `world` is `(atom {})`).

    (me/assoc-in world [:countries :ukraine]
        {:capital "Kyiv" :location "Europe"})

It behaves like you expect it to behave, just sets a value by some path. But
here is a thing - you often have not a map, but a list of items, like that:

    (me/assoc-in world [:countries]
                [{:name "Ukraine" :capital "Kyiv"}
                 {:name "Croatia" :capital "Zagreb"}])

In this case, addressing them becomes harder. You're not going to have this
list, render it somehow, and let it flow, as it often happens in back-end
applications, you're going to live with it, and possibly with changed version of
it. What if it is going to be sorted in different way? What if you get more
items appended/inserted? In this case, you have improved way to ask for an item:

    (me/get-in @world [:countries {:name "Ukraine"}])

Having a map in your path has special meaning - it's going to be treated like a
filter. Such a call will get you an item from `[:countries]` list, which
conforms to given filter. "Conforms" means "all matching keys from item should
have same value as in filter".

And here is the thing:

    (me/assoc-in world [:countries {:name "Ukraine"} :location] "Europe")

This will result in adding `:location` key to "Ukraine" item.

### Notifications

When building an interface, it's good to have interface which will react to
change of data. For this, there is an `on` function, which notifies given
handler when some changes appear inside of path:

    (me/on world [:countries {:name "Ukraine"}]
          (fn [path value] (log "path:" path "value:" value)))

Then updating anything in "Ukraine" item will result in handler being called.


## API specification

- `(assoc-in world [:items {:id 1} :name] "something")`

  Puts `"something"` in property `:name` of item, found by filter `{:id 1}`. If
  there is no match, fails. Works with atoms.

  TODO: If item does not exist, creates it based off filter and appends it in a
  sequence. If sequence does not exist, creates a vector.

  Or! TODO: Puts "x" in item, found by filter `{:id 1}`. If filter find no
  match, nothing happens. Creates items only by using literal paths, any
  "filters" will prevent creation of new elements. Makes it possible to have
  arbitrary functions as filters

- `(update-in world [:items {:id 1}] fn)`

  Works like `assoc-in`. Works with atoms.

- `(all-in @world [:items {:id 1} :name])`

  Returns all the items matched by path. Works with plain data structures.

- `(get-in @world [:items {:id 1} :name])`

  Returns first item matched by path. Works with plain data structures.

- `(on world [items {:id 1}] (fn [data path] (log path value)))`

  Notifies when change occurs in path (i.e. if changes appear at or inside
  whatever happens to be matched by given path). `path` argument holds relative
  path to changed element, and `data` holds value of an item matched by path
  supplied to `on` (not actually changed value). Works with atoms.
