ezirmin — An easy interface on top of the Irmin library.
-------------------------------------------------------------------------------
%%VERSION%%

Ezirmin is an easy interface on top of the
[Irmin](https://github.com/mirage/irmin) library. It comes with set of mergeable
datatypes, instantiated to specific backends to quickly get on with it. For
example,

```ocaml
$ utop
utop # #require "ezirmin";;
utop # module M = Ezirmin.FS_queue(Tc.String);; (* Mergeable queue of strings *)
utop # open M;;
utop # open Lwt.Infix;;
utop # let m = Lwt_main.run (init ~root:"/tmp/ezirminq" ~bare:true () >>= master);;
val m : branch = <abstr>
utop # push m ["home"; "todo"] "buy milk";;
- : unit = ()
utop # push m ["work"; "todo"] "publish ezirmin";;
- : unit = ()
```

This persistent mergeable queue is saved in the git repository at
`/tmp/ezirminq`. In another terminal,

```ocaml
$ utop
utop # #require "ezirmin";;
utop # module M = Ezirmin.FS_queue(Tc.String);; (* Mergeable queue of strings *)
utop # open M;;
utop # open Lwt.Infix;;
utop # let m = Lwt_main.run (init ~root:"/tmp/ezirminq" ~bare:true () >>= master);;
val m : branch = <abstr>
utop # pop m ["work"; "todo"];;
- : string option = Some "buy milk"
```

For concurrency control, use branches. In the first terminal,

```ocaml
utop # let wip_t1 = Lwt_main.run @@ clone_force m "wip_t1";;
utop # push wip_t1 ["home"; "todo"] "walk dog";;
- : unit = ()
utop # push wip_t1 ["home"; "todo"] "take out trash";;
- : unit = ()
```

The changes are not visible until the branches are merged.

```ocaml
utop # to_list m ["home"; "todo"];;
- : string list = []
utop # merge wip_t1 ~into:m;;
- : unit = ()
utop # to_list m ["home"; "todo"];;
- : string list = ["walk dog"; "take out trash"]
```

### What's in the box

The mergeable datatypes currently available are:

* [Blog_log](http://kcsrk.info/ezirmin/Ezirmin.Blob_log.html): An append-only
  log stored as [blobs](). Append is an `O(n)` operation where `n` is the size
  of the log.
* [Log](http://kcsrk.info/ezirmin/Ezirmin.Log.html): An append-only
  write-optimized log with support for paginated reads. Append is an `O(1)`
  operation
* [Queue](http://kcsrk.info/ezirmin/Ezirmin.Queue.html): An efficient queue with
  `O(1)` push and pop operations.
* [Lww_register](http://kcsrk.info/ezirmin/Ezirmin.Lww_register.html):
  Last-writer-wins register.

Since these datatypes are defined such that a merge is always possible and that
merges in different orders converge, they are
[CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type).

The backends available are:

* Git file system backend
* Git in-memory backend

Ezirmin is distributed under the ISC license.

Homepage: https://github.com/kayceesrk/ezirmin

Contact: KC Sivaramakrishnan `<sk826@cl.cam.ac.uk>`

## Installation

Ezirmin can be installed with `opam`:

    opam install ezirmin

If you don't use `opam` consult the [`opam`](opam) file for build instructions.

## Documentation

The documentation and API reference is automatically generated by `ocamldoc`
from the interfaces. It can be consulted [online][doc] and there is a generated
version in the `doc` directory of the distribution.

[doc]: http:/kcsrk.info/ezirmin

## Sample programs

If you installed Ezirmin with `opam` sample programs are located in the
directory `opam config var ezirmin:doc`.

In the distribution sample programs and tests are located in the
[`examples`](examples) directory of the distribution. They can be built and run
with:

    topkg build --tests true && topkg test
