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
@0ap[0]: 1
@0ap[0]: 0
0
assertion "ba->ba_type == B_AT_VAR" failed: file "btrace.c", line 735, function "stmt_clear"
zsh: abort (core dumped)  btrace testcases/b_at_var_1.bt
```

Testcase: [b_at_var_2.bt](./testcases/b_at_var_2.bt)

```
=> Print with one element
@map[7]: 1
assertion "ba->ba_type == B_AT_VAR" failed: file "btrace.c", line 925, function "stmt_zero"
zsh: abort (core dumped)  btrace testcases/b_at_var_2.bt
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
@map[4]: 9999
=> t ant all map:
@map[4]: 9999
=> Print anter clear:
fter Velete:
zsh: segmentation fault (core dumped)  btrace testcases/map_rb_find.bt
```

### 0xdfdfdfdfdfdfdfdf

Testcase: [run_printmaps.bt](./testcases/run_printmaps.bt)

```
[#0] Id 1, stopped 0xc6b1f53f87b in rule_printmaps (), reason: SIGSEGV
[...]
gef➤  x $rax
0xdfdfdfdfdfdfdfdf:     Cannot access memory at address 0xdfdfdfdfdfdfdfdf
```

## Triaged

## Fixed
