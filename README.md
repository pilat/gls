gls
===

Fast goroutine local storage

### WARNING ###

There is extensive documentation and discussion on why implementing and using
thread-local storage in Go - actually, goroutine-local storage - is a bad idea.

See for example the [Go FAQ on goroutine id](https://golang.org/doc/faq#no_goroutine_id)
and [context.Context](https://blog.golang.org/context), which is how you're encouraged
to solve problems that would require a goroutine-local storage.

The main obstacle in adopting `context.Context` is that *all* of your functions
must have a new first argument. So, if that horrifies you or is simply not feasible
for your use case, feel free to ignore this warning and read on.

Just remember that, if some Go programmers frowns at your use of goroutine-local
storage, there are good reasons.

### Why? ###

To retrieve per-goroutine data that some function did not - or could not -
pass through the call chain, down to where you need it.

Other goroutine-local libraries, as [jtolds/gls](https://github.com/jtolds/gls)
and [tylerb/gls](https://github.com/tylerb/gls) explain the reasons
and use cases for goroutine-local storage more in detail.

### How it works ###

Go runtime has an internal, i.e. unexported, goroutine-local `runtime.g` struct.
It is used for several purposes, including `defer()`, `recover()`,
by the goroutine scheduler, and it even has an unexported `goid` field,
i.e. a goroutine ID.

Several other goroutine-local libraries extract this goroutine ID
with various tricks, most notably from `runtime.Stack()` textual output.

Instead, we use a tiny bit of assembler code to retrieve the address
of the `runtime.g` struct and return it converted to an opaque `uintptr`.

We use it as the key in a global variable containing per-goroutine data.

This is also **fast**, probably orders of magnitude faster than most other solutions.

#### Why not the same goroutine ID? ####

To avoid fiddling with the internal layout of `runtime.g` struct,
only its address is taken.

Accessing the `goid` field would require knowing its offset within the struct,
which is both tedious and error-prone to retrieve, since it's an unexported
field of an unexported struct type.

### Documentation ###

See the autogenerated API docs at http://godoc.org/github.com/cosmos72/gls

### License ###

BSD 3-Clause License

