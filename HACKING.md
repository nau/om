Om hacking
====================


Developers
----------

```
$ git clone git://github.com/groupoid/om && cd om
$ make
$ ./mad sh
```

Session example with Om:

```erlang
> om:all().
...
> om:scan().
...
> om:modes().
["erased","girard","hurkens","normal","setoids"]
> om:mode("normal").
> om:extract().
>
Active: module loaded: {reloaded,'List'}
Active: module loaded: {reloaded,'Maybe'}
Active: module loaded: {reloaded,'Nat'}
Active: module loaded: {reloaded,'Vector'}
> ch:main().
Zero: 0
Cons/Nil: [2,1]
Test Big List: [2,3,5,8,11,19]
Two: 2
ok
Pack/Unpack 1 000 000 Inductive List: {714483,'_'}
Pack/Unpack 1 000 000 Inductive Nat: {743755,1000000}
```

## Parser Term Specification

This information is subject to change.

### Result AST

```erlang
     {  star,          Universe  }  -- universe
     { {"λ",{Name,I}}, {In,Out}  }  -- lambda
     { {"∀",{Name,I}}, {In,Out}  }  -- pi
     {  "→",           {In,Out}  }  -- anonymous pi
     {  var,           {Name,I}  }  -- var
     {  app,           {Fun,Arg} }  -- app
```

### Intermediate AST

```erlang
     {  open     }
     {  close    }
     {  arrow    }
     {  colon    }
     {  lambda   }
     {  pi       }
     {  var,      {Name,Depth} }
     {  app,      {Fun,Arg}    }
     {  typevar,  {Name,Depth} }    -- argument name,
                                       transforms to lambda or pi
                                       during rewind
```

## Commands

### mode

Set the environment folder:

```erlang
> om:mode("normal").
ok
```

### a

Inline some terms:

```erlang
> om:a("\\(x:*)->\\(y:#List/id)->y").
{{"λ",{x,0}},
 {{star,1},
  {{"λ",{y,0}},
   {{{"λ",{'X',0}},
     {{star,1},
      {{"λ",{'Cons',0}},
       {{star,1},{{"λ",{'Nil',0}},{{star,1},{var,{'X',0}}}}}}}},
    {var,{y,0}}}}}}
```

### term

Check how term inlining and loading works:

```erlang
 > om:a("#List/map") == om:term("List/map").
 true
```

### show

Use internal functions:

```erlang
> om:show("priv/normal/List/@").

===[ File: priv/normal/List/@ ]==========
Cat: λ(a : *) → ∀(List : *) → ∀(Cons : ∀(head : a) → ∀(tail : List) → List) → ∀(Nil : List) → List
Term: 279
{{"λ",{a,0}},
 {{star,1},
  {{"∀",{'List',0}},
   {{star,1},
    {{"∀",{'Cons',0}},
     {{{"∀",{head,0}},
       {{var,{a,0}},
        {{"∀",{tail,0}},{{var,{'List',0}},{var,{'List',0}}}}}},
      {{"∀",{'Nil',0}},{{var,{'List',0}},{var,{'List',0}}}}}}}}}}
```

### parse

Parse raw expressions:

```erlang
> om:parse("∀ (a: *) → λ (b: * → * → *) → λ (c: * → a) → (((b (c a)) a) a))").
{[],
 [{{"∀",{a,0}},
   {{star,1},
    {{"λ",{b,0}},
     {{"→",{{star,1},{"→",{{star,1},{star,1}}}}},
      {{"λ",{c,0}},
       {{"→",{{star,1},{var,{a,0}}}},
        {app,{{app,{{app,{{var,{b,0}},{app,{{var,...},{...}}}}},
                    {var,{a,0}}}},
              {var,{a,0}}}}}}}}}}]}
```

### extract

Extract Erlang Modules:

```erlang
> om:extract("priv/normal/List").
ok
Active: module loaded: {reloaded,'List'}
> om:mode("normal").
> om:extract().
ok
Active: module loaded: {reloaded,'Bool'}
Active: module loaded: {reloaded,'List'}
```

### main

Sandbox for testing from bare Erlang.
Example of usage of compiled modules `List` and `Nat`:

```erlang
> ch:main().
Zero: 0
Cons/Nil: [2,1]
Test Big List: [2,3,5,8,11,19]
Two: 2
ok
Pack/Unpack 1 000 000 Inductive List: {733256,'_'}
Pack/Unpack 1 000 000 Inductive Nat: {748433,1000000}
```

`foldl` version of `stdlib` (for comparison):

```erlang
> timer:tc(lists,foldl,[fun(X,A) -> A end,0,lists:seq(1,1000000)]).
{735410,0}
```

### type

Typechecking:

```
> om:type("DEP.AND.PR-L-test").
** exception error: ["∀",
                     {app,{{{"λ",{'B',0}},
                            {{{"∀",{"_",0}},{{var,{'A',0}},{star,1}}},
                             {{"∀",{'AND',0}},
                              {{star,1},
                               {{"∀",{pair,0}},
                                {{{"∀",{a,0}},
                                  {{var,{'A',0}},
                                   {{"∀",{b,0}},
                                    {{app,{{var,{'B',...}},{var,{...}}}},{var,{'AND',0}}}}}},
                                 {var,{'AND',0}}}}}}}},
                           {var,{'B',0}}}}]
```

### scan

Scan modules in current mode:

```erlang
> om:mode("erased").
ok
> om:scan().
PASSED
[{true,"priv/erased/Bool/False"},
 {true,"priv/erased/Bool/True"},
 {true,"priv/erased/Bool/id"},
 {true,"priv/erased/List/Cons"},
 {true,"priv/erased/List/Nil"},
 {true,"priv/erased/List/id"},
 {true,"priv/erased/Nat/Succ"},
 {true,"priv/erased/Nat/Zero"},
 {true,"priv/erased/Nat/id"},
 {true,"priv/erased/Prod/Mk"},
 {true,"priv/erased/Prod/id"},
 {true,"priv/erased/Prod/pr1"},
 {true,"priv/erased/Prod/pr2"},
 {true,"priv/erased/Ret/Error"},
 {true,"priv/erased/Ret/Io"},
 {true,"priv/erased/Ret/Ok"},
 {true,"priv/erased/Ret/id"}]
```
