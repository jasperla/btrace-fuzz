# btrace fuzzing

[btrace(8)](http://man.openbsd.org/btrace) fuzzing.

## Untriaged

### Hang

Testcase: [hangt.bt](./testcases/hang.bt).
Fails due to missing newline, also reproduceable with `btrace -e "0"`.

```
testcases/hang.bt:1:0: syntax error:
0
^
^C
```

### Div by zero

Testcase: [divzero.bt](./testcases/divzero.bt)

```
zsh: floating point exception (core dumped)  btrace testcases/divzero.bt
```

### no string conversion for type 0

Testcase: [no_string_type_0.bt](./testcases/no_string_type_0.bt)

```
no string conversion for type 0
zsh: abort (core dumped)  btrace -v testcases/no_string_type_0.bt
```

### no long conversion for type 1

Testcase: [no_long_type_1.bt](./testcases/no_long_type_1.bt)

```
no long conversion for type 1
zsh: abort (core dumped)  btrace -v no_long_type_1.bt
```

### store not implemented for type 3

Testcase: [store_type_3.bt](./testcases/store_type_3.bt)

```
store not implemented for type 3
zsh: abort (core dumped)  btrace -v store_type_3.bt
```
### store not implemented for type 6

Testcase: [store_type_6.bt](./testcases/store_type_6.bt)

```
store not implemented for type 6
zsh: abort (core dumped)  btrace -v store_type_6.bt
```

### 0xdfdfdfdfdfdfdfdf

Doesn't crash 100% of the time, may need a few tries.

Testcase: [run_printmaps.bt](./testcases/resolved/run_printmaps.bt)

```
debug: parsed probe 'BEGIN'
debug: eval rule 'BEGIN'
debug: map=0x0 'map' insert key=0x5fa3e29d4c0 '0' bval=0x5fa3e2a0ac0
debug: bv=0x5fa3e29d140 var 'map' store (0x5fa3e2ad5e0)
=> Using p, @map[4F);
}

END
{
        @map[7] = counp8
debug: eval default 'end' rule
zsh: segmentation fault (core dumped)  btrace -vv testcases/run_printmaps.bt
```

```
[#0] Id 1, stopped 0xc6b1f53f87b in rule_printmaps (), reason: SIGSEGV
[...]
gef➤  x $rax
0xdfdfdfdfdfdfdfdf:     Cannot access memory at address 0xdfdfdfdfdfdfdfdf
```

Testcase: [run_printmaps2.bt](./testcases/run_printmaps2.bt)

```
gef➤  bt
#0  0x000005374ad5adcb in rule_printmaps (r=0x539fc4a3200) at btrace.c:581
#1  0x000005374ad5a80b in rules_teardown (fd=0xffffffff) at btrace.c:543
#2  0x000005374ad59bcb in rules_do (fd=0xffffffff) at btrace.c:377
#3  0x000005374ad59671 in main (argc=0x0, argv=0x7f7ffffbea60) at btrace.c:203
gef➤  bt full
#0  0x000005374ad5adcb in rule_printmaps (r=0x539fc4a3200) at btrace.c:581
        bv = 0xdfdfdfdfdfdfdfdf
        map = 0x0
        ba = 0xdfdfdfdfdfdfdfdf
        bs = 0x539fc4a1200
```

## Fixed

### `assertion "ba->ba_type == B_AT_VAR" failed`

Fixed with: [0e0170ac867c821d90c352491e5bee4003e1fddd](https://github.com/openbsd/src/commit/0e0170ac867c821d90c352491e5bee4003e1fddd)

Testcase: [b_at_var_1.bt](./testcases/resolved/b_at_var_1.bt)

```
debug: parsed probe 'BEGIN'
debug: eval rule 'BEGIN'
debug: map=0x0 '0ap' insert key=0x9660ac53aa0 '0' bval=0x9660ac2aea0
debug: map=0x9660ac483b0 '0ap' print (top=-1)
@0ap[0]: 1
debug: map=0x9660ac483b0 '0ap' zero
debug: map=0x9660ac483b0 '0ap' print (top=-1)
@0ap[0]: 0
debug: map=0x0 '' insert key=0x9660ac533a0 '0' bval=0x9660ac538c0
debug: map=0x9660ac40a70 '' insert key=0x9660ac5b4e0 '0' bval=0x9660ac2a8c0
debug: map=0x9660ac40a70 '' insert key=0x9660ac2aae0 '0' bval=0x9660ac5b280
0
assertion "ba->ba_type == B_AT_VAR" failed: file "btrace.c", line 735, function "stmt_clear"
zsh: abort (core dumped)  btrace -vv testcases/b_at_var_1.bt
```

Testcase: [b_at_var_2.bt](./testcases/resolved/b_at_var_2.bt)

```
debug: parsed probe 'END'
debug: parsed probe 'BEGIN'
debug: eval rule 'BEGIN'
debug: ba=0xc0ce05a1600 eval '8 - 1 = 7'
debug: map=0x0 'map' insert key=0xc0ce05a1600 '7' bval=0xc0ce05a1c40
=> Print with one element
debug: map=0xc0ce05814a0 'map' print (top=-1)
@map[7]: 1
assertion "ba->ba_type == B_AT_VAR" failed: file "btrace.c", line 925, function "stmt_zero"
zsh: abort (core dumped)  btrace -vv testcases/b_at_var_2.bt
```

### `assertion "SLIST_EMPTY(&l_variables)" failed`

Fixed with: [6e5d64e676581b33a2b06cfa1896b2796501b136](https://github.com/openbsd/src/commit/6e5d64e676581b33a2b06cfa1896b2796501b136)
Testcase: [l_variables.bt](testcases/resolved/l_variables.bt)

```
testcases/l_variables.bt:3:11: syntax error:
        foo;
          ^
assertion "SLIST_EMPTY(&l_variables)" failed: file "bt_parse.y", line 954, function "btparse"
zsh: abort (core dumped)  btrace testcases/1.bt
```

### `assertion "index == 1" failed`

Fixed as part of [58afdee70d18227ff57089cd50d71573337e811d](https://github.com/openbsd/src/commit/58afdee70d18227ff57089cd50d71573337e811d)
Testcase: [index.bt](./testcases/resolved/index.bt).

```
assertion "index == 1" failed: file "bt_parse.y", line 256, function "get_varg"
zsh: abort (core dumped)  btrace testcases/index.bt
```

### `0x000005c00da07df1 in map_RB_FIND (head=0x0, elm=0x7f7ffffc9c50) at map.c:55`

Testcase: [map_rb_find.bt](./testcases/resolved/map_rb_find.bt)

```
debug: parsed probe 'END'
debug: parsed probe 'BEGIN'
debug: eval rule 'BEGIN'
debug: ba=0x6c965b38280 eval '3 + 1 = 4'
debug: map=0x0 'map' insert key=0x6c965b38280 '4' bval=0x6c965b38a20
debug: map=0x6c965b2fa50 'map' print (top=-1)
@map[4]: 9999
debug: eval rule 'END'
=> t ant all map:
debug: map=0x6c965b2fa50 'map' print (top=-1)
@map[4]: 9999
debug: map=0x0 'map' clear
=> Print anter clear:
fter Velete:
debug: map=0x0 'map' delete key=0x6c965b38d80 '4'
zsh: segmentation fault (core dumped)  btrace -vv testcases/map_rb_find.bt
```


### invalid argument type 6

Testcase: [invalid_arg_6.bt](./testcases/resolved/invalid_arg_6.bt)

```
invalid argument type 6
zsh: abort (core dumped)  btrace -v id:000000,sig:08,src:000001,time:81562,op:havoc,rep:2
```
