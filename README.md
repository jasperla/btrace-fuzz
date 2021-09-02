# btrace fuzzing

[btrace(8)](http://man.openbsd.org/btrace) fuzzing.

## Untriaged

### `assertion "SLIST_EMPTY(&l_variables)" failed`

Testcase: [l_variables.bt](testcases/l_variables.bt)

```
testcases/l_variables.bt:3:11: syntax error:
        foo;
          ^
assertion "SLIST_EMPTY(&l_variables)" failed: file "bt_parse.y", line 954, function "btparse"
zsh: abort (core dumped)  btrace testcases/1.bt
```

### `assertion "ba->ba_type == B_AT_VAR" failed`

Testcase: [b_at_var_1.bt](./testcases/b_at_var_1.bt)

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

Testcase: [b_at_var_2.bt](./testcases/b_at_var_2.bt)

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

### Hang

Testcase: [hangt.bt](./testcases/hang.bt).
Fails due to missing newline, also reproduceable with `btrace -e "0"`.

```
testcases/hang.bt:1:0: syntax error:
0
^
^C
```

### `assertion "index == 1" failed`

Testcase: [index.bt](./testcases/index.bt).

```
assertion "index == 1" failed: file "bt_parse.y", line 256, function "get_varg"
zsh: abort (core dumped)  btrace testcases/index.bt
```

### Div by zero

Testcase: [divzero.bt](./testcases/divzero.bt)

```
zsh: floating point exception (core dumped)  btrace testcases/divzero.bt
```

### `0x000005c00da07df1 in map_RB_FIND (head=0x0, elm=0x7f7ffffc9c50) at map.c:55`

Testcase: [map_rb_find.bt](./testcases/map_rb_find.bt)

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

### 0xdfdfdfdfdfdfdfdf

Testcase: [run_printmaps.bt](./testcases/run_printmaps.bt)

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

## Triaged

## Fixed
