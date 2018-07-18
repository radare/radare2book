## Code Analysis

Code analysis is a common technique used to extract information from assembly code. Radare uses internal data structures to identify basic blocks, function trees, to extract opcode-level information etc. The most common radare2 analysis command sequence is:

```
[0x08048440]> aa
[0x08048440]> pdf @ main
		   ; DATA XREF from 0x08048457 (entry0)
/ (fcn) fcn.08048648 141
|          ;-- main:
|          0x08048648    8d4c2404     lea ecx, [esp+0x4]
|          0x0804864c    83e4f0       and esp, 0xfffffff0
|          0x0804864f    ff71fc       push dword [ecx-0x4]
|          0x08048652    55           push ebp
|          ; CODE (CALL) XREF from 0x08048734 (fcn.080486e5)
|          0x08048653    89e5         mov ebp, esp
|          0x08048655    83ec28       sub esp, 0x28
|          0x08048658    894df4       mov [ebp-0xc], ecx
|          0x0804865b    895df8       mov [ebp-0x8], ebx
|          0x0804865e    8975fc       mov [ebp-0x4], esi
|          0x08048661    8b19         mov ebx, [ecx]
|          0x08048663    8b7104       mov esi, [ecx+0x4]
|          0x08048666    c744240c000. mov dword [esp+0xc], 0x0
|          0x0804866e    c7442408010. mov dword [esp+0x8], 0x1 ;  0x00000001
|          0x08048676    c7442404000. mov dword [esp+0x4], 0x0
|          0x0804867e    c7042400000. mov dword [esp], 0x0
|          0x08048685    e852fdffff   call sym..imp.ptrace
|             sym..imp.ptrace(unk, unk)
|          0x0804868a    85c0         test eax, eax
|      ,=< 0x0804868c    7911         jns 0x804869f
|      |   0x0804868e    c70424cf870. mov dword [esp], str.Don_tuseadebuguer_ ;  0x080487cf
|      |   0x08048695    e882fdffff   call sym..imp.puts
|      |      sym..imp.puts()
|      |   0x0804869a    e80dfdffff   call sym..imp.abort
|      |      sym..imp.abort()
|      `-> 0x0804869f    83fb02       cmp ebx, 0x2
|     ,==< 0x080486a2    7411         je 0x80486b5
|     |    0x080486a4    c704240c880. mov dword [esp], str.Youmustgiveapasswordforusethisprogram_ ;  0x0804880c
|     |    0x080486ab    e86cfdffff   call sym..imp.puts
|     |       sym..imp.puts()
|     |    0x080486b0    e8f7fcffff   call sym..imp.abort
|     |       sym..imp.abort()
|     `--> 0x080486b5    8b4604       mov eax, [esi+0x4]
|          0x080486b8    890424       mov [esp], eax
|          0x080486bb    e8e5feffff   call fcn.080485a5
|             fcn.080485a5() ; fcn.080484c6+223
|          0x080486c0    b800000000   mov eax, 0x0
|          0x080486c5    8b4df4       mov ecx, [ebp-0xc]
|          0x080486c8    8b5df8       mov ebx, [ebp-0x8]
|          0x080486cb    8b75fc       mov esi, [ebp-0x4]
|          0x080486ce    89ec         mov esp, ebp
|          0x080486d0    5d           pop ebp
|          0x080486d1    8d61fc       lea esp, [ecx-0x4]
\          0x080486d4    c3           ret
```
In this example, we analyze the whole file (`aa`) and then print disassembly of the `main()` function (`pdf`).
The `aa` command belongs to the family of autoanalysis commands and performs only the most basic
autoanalysis steps. In radare2 there are many different types of the autoanalysis commands with a
different analysis depth, including partial emulation: `aa`, `aaa`, `aab`, `aaaa`, ...
There is also a mapping of those commands to the r2 CLI options: `r2 -A`, `r2 -AA`, etc

It is a common sense that completely automated analysis can produce non sequitur results, thus
radare2 provides separate commands for the particular stages of the analysis allowing fine-grained
control of the analysis process. Moreover, there is a treasure trove of configuration variables
for controlling the analysis outcomes. You can find them in `anal.*` and `emu.*` cfg variables
namespaces.

One of the most important "basic" analysis commands is a set of `af` subcommands. `af` means
"analyze function". Using this command you can either allow automatic analysis of the particular
function or perform completely manual one.

```
|Usage: af
| af ([name]) ([addr])                  analyze functions (start at addr or $$)
| afr ([name]) ([addr])                 analyze functions recursively
| af+ addr name [type] [diff]           hand craft a function (requires afb+)
| af- [addr]                            clean all function analysis data (or function at addr)
| afb+ fcnA bbA sz [j] [f] ([t]( [d]))  add bb to function @ fcnaddr
| afb[?] [addr]                         List basic blocks of given function
| afB 16                                set current function as thumb (change asm.bits)
| afC[lc] ([addr])@[addr]               calculate the Cycles (afC) or Cyclomatic Complexity (afCc)
| afc[?] type @[addr]                   set calling convention for function
| afd[addr]                             show function + delta for given offset
| aff                                   re-adjust function boundaries to fit
| afF[1|0|]                             fold/unfold/toggle
| afi [addr|fcn.name]                   show function(s) information (verbose afl)
| afl[?] [l*] [fcn name]                list functions (addr, size, bbs, name) (see afll)
| afm name                              merge two functions
| afM name                              print functions map
| afn[?] name [addr]                    rename name for function at address (change flag too)
| afna                                  suggest automatic name for current offset
| afo [fcn.name]                        show address for the function named like this
| afs [addr] [fcnsign]                  get/set function signature at current address
| afS[stack_size]                       set stack frame size for function at current address
| aft[?]                                type matching, type propagation
| afu [addr]                            resize and analyze function from current address until addr
| afv[bsra]?                            manipulate args, registers and variables in function
| afx                                   list function references
```

One of the most challenging tasks while performing a function analysis - merge, crop or resize.
As with other analysis commands you have two ways - semi-automatic and totally manual one.
For the semi-automatic, you can use `afm <function name>` to merge current function with
the function specified, `aff` to readjust function after analysis changes or function edits,
`afu <address>` to do the resize and analysis of the current function until specified address.

Apart from those semi-automatic ways to edit/analyze the function, you can create it in
complete manual mode with `af+` command and edit basic blocks of it using `afb` commands.
Before changing the basic blocks of the function it is recommended to check the already
presented ones:
```
[0x00003ac0]> afb
0x00003ac0 0x00003b7f 01:001A 191 f 0x00003b7f
0x00003b7f 0x00003b84 00:0000 5 j 0x00003b92 f 0x00003b84
0x00003b84 0x00003b8d 00:0000 9 f 0x00003b8d
0x00003b8d 0x00003b92 00:0000 5
0x00003b92 0x00003ba8 01:0030 22 j 0x00003ba8
0x00003ba8 0x00003bf9 00:0000 81
```

There are two very important commands: `afc` and `afB`. The latter is a must-know
command for some platforms like ARM, it provides a way to change the "bitness" of the
particular function, basically allowing to select between ARM and Thumb modes.
`afc` on the other side allows to manually specify function calling convention

```
|Usage: afc[agl?]
| afc convention  Manually set calling convention for current function
| afc             Show Calling convention for the Current function
| afcr[j]         Show register usage for the current function
| afca            Analyse function for finding the current calling convention
| afcf name       Prints return type function(arg1, arg2...)
| afcl            List all available calling conventions
| afco path       Open Calling Convention sdb profile from given path
[0x00003ac0]> afc
amd64
[0x00003ac0]> afcl
amd64
ms
```

## Recursive analysis

There are 4 important program wide half-automated analysis commands:
 - `aab` - perform basic-block analysis ("Nucleus" algorithm)
 - `aac` - analyze function calls from one (selected or current function)
 - `aaf` - analyze all function calls
 - `aar` - analyze data references
 - `aad` - analyze pointers to pointers references

Those are only generic semi-automated reference searching algorithms. Radare2 provides a
wide choice of manual references' creation of any kind. For this fine-grained control
you can use `ax` commands.
```
|Usage: ax[?d-l*] # see also 'afx?'
| ax              list refs
| ax*             output radare commands
| ax addr [at]    add code ref pointing to addr (from curseek)
| ax- [at]        clean all refs/refs from addr
| ax-*            clean all refs/refs
| axc addr [at]   add generic code ref
| axC addr [at]   add code call ref
| axg [addr]      show xrefs graph to reach current function
| axgj [addr]     show xrefs graph to reach current function in json format
| axd addr [at]   add data ref
| axq             list refs in quiet/human-readable format
| axj             list refs in json format
| axF [flg-glob]  find data/code references of flags
| axt [addr]      find data/code references to this address
| axf [addr]      find data/code references from this address
| axs addr [at]   add string ref
```
The most commonly used `ax` commands are `axt` and `axf`, especially as a part of various r2pipe
scripts. Lets say we see the string in the data or a code section and want to find all places
it was referenced from, we should use `axt`:
```
[0x0001783a]> pd 2
	;-- str.02x:
	; STRING XREF from 0x00005de0 (sub.strlen_d50)
	; CODE XREF from 0x00017838 (str.._s_s_s + 7)
	0x0001783a     .string "%%%02x" ; len=7
	;-- str.src_ls.c:
	; STRING XREF from 0x0000541b (sub.free_b04)
	; STRING XREF from 0x0000543a (sub.__assert_fail_41f + 27)
	; STRING XREF from 0x00005459 (sub.__assert_fail_41f + 58)
	; STRING XREF from 0x00005f9e (sub._setjmp_e30)
	; CODE XREF from 0x0001783f (str.02x + 5)
	0x00017841 .string "src/ls.c" ; len=9
[0x0001783a]> axt
sub.strlen_d50 0x5de0 [STRING] lea rcx, str.02x
(nofunc) 0x17838 [CODE] jae str.02x
```

Apart from predefined algorithms to identify functions there is a way to specify
a function prelude with a configuration option `anal.prelude`. For example, like
`e anal.prelude = 0x554889e5` which means
```
push rbp
mov rbp, rsp
```
on x86_64 platform. It should be specified _before_ any analysis commands.

## Configuration

Radare2 allows to changing the behavior of almost any analysis stages or commands.
There are different kinds of the configuration options:

 - Flow control
 - Basic blocks control
 - References control
 - IO/Ranges
 - Jump tables analysis control
 - Platform/target specific options

### Control flow configuration

Two most commonly used options for changing the behavior of control flow analysis in radare2 are
`anal.hasnext` and `anal.afterjump`. The first one allows forcing radare2 to continue the analysis
after the end of the function, even if the next chunk of the code wasn't called anywhere, thus
analyzing all of the available functions. The latter one allows forcing radare2 to continue
the analysis even after unconditional jumps.

In addition to those we can also set `anal.ijmp` to follow the indirect jumps, continuing analysis;
`anal.pushret` to analyze `push ...; ret` sequence as a jump; `anal.nopskip` to skip the NOP
sequences at a function beginning.

For now, radare2 also allows you to change the maximum basic block size with `anal.bb.maxsize` option
. The default value just works in most use cases, but it's useful to increase that for example when 
dealing with obfuscated code. Beware that some of basic blocks
control options may disappear in the future in favor of more automated ways to set those.

For some unusual binaries or targets, there is an option `anal.noncode`. Radare2 doesn't try
to analyze data sections as a code by default. But in some cases - malware, packed binaries,
binaries for embedded systems, it is often a case. Thus - this option.

### Reference control

The most crucial options that change the analysis results drastically. Sometimes some can be
disabled to save the time and memory when analysing big binaries.

- `anal.jmpref` - to allow references creation for unconditional jumps
- `anal.cjmpref` - same, but for conditional jumps
- `anal.datarefs` - to follow the data references in code
- `anal.refstr` - search for strings in data references
- `anal.strings` - search for strings and creating references

Note that strings references control disabled by default because it increases analysis times.

### Analysis ranges

There are only three options for this:

- `anal.limits` - enables the range limits for analysis operations
- `anal.from` - starting address of the limit range
- `anal.to` - the corresponding end of the limit range

### Jump tables

Jump tables are being one of the trickiest targets in binary reverse engineering. There are hundreds
of different types, the end result depending on the compiler/linker and LTO stages of optimization.
Thus radare2 allows enabling some experimental jump tables detection algorithms using `anal.jmptbl`
option. Eventually, algorithms moved into the default analysis loops once they start to work on
every supported platform/target/testcase.
Two more options can affect the jump tables analysis results too:

- `anal.ijmp` - follow the indirect jumps, some jump tables rely on them
- `anal.datarefs` - follow the data references, some jump tables use those

### Platform specific controls

There are two common problems when analysing embedded targets: ARM/Thumb detection and MIPS GP
value. In case of ARM binaries radare2 supports some autodetection of ARM/Thumb mode switches, but
beware that it uses partial ESIL emulation, thus slowing the analysis process. If you will not
like the results, particular functions' mode can be overridden with `afB` command.

The MIPS GP problem is even trickier. It is a basic knowledge that GP value can be different not only
for the whole program, but also for some functions. To partially solve that there are options
`anal.gp` and `anal.gp2`. The first one sets the GP value for the whole program or particular
function. The latter allows to "constantify" the GP value if some code is willing to change its
value, always resetting it if the case. Those are heavily experimental and might be changed in the
future in favor of more automated analysis.

## Visuals

One of the easiest way to see and check the changes of the analysis commands and variables
is to perform a scrolling in a `Vv` special visual mode, allowing functions preview:

```
-[ functions ]----------------                              ...
(a) add     (x)xrefs     (q)quit                            Visual code review (pdf)
(r) rename  (c)calls     (g)go                              ╭ (fcn) entry0 43
(d) delete  (v)variables (?)help                            │   entry0 ();
>* 0x00005480   43 entry0                                   │          0x00005480      xor ebp, ebp
   0x000150f0   46 sym._obstack_allocated_p                 │          0x00005482      mov r9, rdx
   0x00014fe0  179 sym._obstack_begin_1                     │          0x00005485      pop rsi
   0x00014fc0  158 sym._obstack_begin                       │          0x00005486      mov rdx, rsp
   0x00015130   97 sym._obstack_free                        │          0x00005489      and rsp, 0xfffffffffffffff0
   0x00015000  283 sym._obstack_newchunk                    │          0x0000548d	   push rax
   0x000151a0   36 sym._obstack_memory_used                 │          0x0000548e	   push rsp
   0x000033f0    6 sym.imp.__ctype_toupper_loc              │          0x0000548f      lea r8, [0x00015e40]
   0x00003400    6 sym.imp.__uflow                          │          0x00005496      lea rcx, [0x00015dd0]
   0x00003410    6 sym.imp.getenv                           │		   0x0000549d      lea rdi, [main]  ; section..text ; 0x3ac0 ; "AWAVAUATUS\x89\xfdH\x89\xf3H\x83
   0x00003420    6 sym.imp.sigprocmask                      │          0x000054a4      call qword [reloc.__libc_start_main] ; [0x21efc8:8]=0
   0x00003430    6 sym.imp.__snprintf_chk                   ╰          0x000054aa      hlt
   0x00003440    6 sym.imp.raise
   0x00000000   48 sym.imp.free
   0x00003450    6 sym.imp.abort
   0x00003460    6 sym.imp.__errno_location
   0x00003470    6 sym.imp.strncmp
   0x00003480    6 sym.imp.localtime_r
   0x00003490    6 sym.imp._exit
   0x000034a0    6 sym.imp.strcpy
   0x000034b0    6 sym.imp.__fpending
   0x000034c0    6 sym.imp.isatty
```

When we want to check how analysis changes affect the result in the case of big functions, we can
use minimap instead, allowing to see a bigger flow graph on the same screen size. To get into
the minimap mode type `VV` then press `p` twice:
```
[0x00003ac0]> VV @ main (nodes 6 edges 5 zoom 100%) BB-MINI mouse:canvas-y mov-speed:5

[ 0x3ac0 ]
; [13] -r-x section size 74709 named .text
;-- section..text:
(fcn) main 313                                                                       <@@@@@@>
    main ();                                                                            │
  ; var int local_8h @ rsp+0x8                                                          │
  ; var int local_10h @ rsp+0x10                                                        │
  ; var int local_28h @ rsp+0x28                                                  t f  __3b7f__
  ; var int local_30h @ rsp+0x30                                                  │ │
  ; var int local_32h @ rsp+0x32                                                  ╰─│─────────╮
  ; var int local_38h @ rsp+0x38                                                  ╭─╯         │
  ; var int local_45h @ rsp+0x45                                                  │           │
  ; var int local_46h @ rsp+0x46                                                  │           │
  ; var int local_47h @ rsp+0x47                                                 __3b84__  __3b92__
  ; var int local_48h @ rsp+0x48                                                  │           │
  ; STRING XREF from 0x0000549d (entry0)                                          │           │
  push r15                                                                        │           │
  push r14                                                                       __3b8d__  __3ba8__
  push r13
  push r12
  push rbp
  push rbx
  mov ebp, edi
  mov rbx, rsi
  ; 'X'
  sub rsp, 0x58
  ; const char *s
  mov rdi, qword [rsi]
  ; [0x28:8]=0x1f6d0
  ; '('
  ...
```
This mode allows you to see the disassembly of each node separately, just navigate between them using `Tab` key.

## Analysis hints

It is not an uncommon case that analysis results are not perfect even after you tried every single
configuration option. This is where the "analysis hints" radare2 mechanism comes in. It allows
to override some basic opcode or metainformation properties, or even to rewrite the whole opcode
string. These commands are located under `ah` namespace:

```
|Usage: ah[lba-]Analysis Hints
| ah?                show this help
| ah? offset         show hint of given offset
| ah                 list hints in human-readable format
| ah.                list hints in human-readable format from current offset
| ah-                remove all hints
| ah- offset [size]  remove hints at given offset
| ah* offset         list hints in radare commands format
| aha ppc 51         set arch for a range of N bytes
| ahb 16 @ $$        force 16bit for current instruction
| ahc 0x804804       override call/jump address
| ahe 3,eax,+=       set vm analysis string
| ahf 0x804840       override fallback address for call
| ahh 0x804840       highlight this adrress offset in disasm
| ahi[?] 10          define numeric base for immediates (1, 8, 10, 16, s)
| ahj                list hints in JSON
| aho foo a0,33      replace opcode string
| ahp addr           set pointer hint
| ahr val            set hint for return value of a function
| ahs 4              set opcode size=4
| ahS jz             set asm.syntax=jz for this opcode
```
One of the most common case is to set a particular numeric base for immediates:
```
[0x00003d54]> ahi?
|Usage ahi [sbodh] [@ offset] Define numeric base
| ahi [base]  set numeric base (1, 2, 8, 10, 16)
| ahi b       set base to binary (2)
| ahi d       set base to decimal (10)
| ahi h       set base to hexadecimal (16)
| ahi o       set base to octal (8)
| ahi p       set base to htons(port) (3)
| ahi i       set base to IP address (32)
| ahi S       set base to syscall (80)
| ahi s       set base to string (1)
[0x00003d54]> pd 2
│          0x00003d54      0583000000     add eax, 0x83
│          0x00003d59      3d13010000     cmp eax, 0x113
[0x00003d54]> ahi d
[0x00003d54]> pd 2
│          0x00003d54      0583000000     add eax, 131
│          0x00003d59      3d13010000     cmp eax, 0x113
[0x00003d54]> ahi b
[0x00003d54]> pd 2
│          0x00003d54      0583000000     add eax, 10000011b
│          0x00003d59      3d13010000     cmp eax, 0x113
```
It is notable that some analysis stages or commands add the internal analysis hints,
which can be checked with `ah` command:
```
[0x00003d54]> ah
 0x00003d54 - 0x00003d54 => immbase=2
[0x00003d54]> ah*
 ahi 2 @ 0x3d54
 ```

Sometimes we need to override jump or call address, for example in case of tricky
relocation, which is unknown for radare2, thus we can change the value manually.
The current analysis information about a particular opcode can be checked with `ao` command.
We can use `ahc` command for performing such a change:
```
[0x00003cee]> pd 2
│          0x00003cee      e83d080100     call sub.__errno_location_530
│          0x00003cf3      85c0           test eax, eax
[0x00003cee]> ao
address: 0x3cee
opcode: call 0x14530
mnemonic: call
prefix: 0
id: 56
bytes: e83d080100
refptr: 0
size: 5
sign: false
type: call
cycles: 3
esil: 83248,rip,8,rsp,-=,rsp,=[],rip,=
jump: 0x00014530
direction: exec
fail: 0x00003cf3
stack: null
family: cpu
stackop: null
[0x00003cee]> ahc 0x5382
[0x00003cee]> pd 2
│          0x00003cee      e83d080100     call sub.__errno_location_530
│          0x00003cf3      85c0           test eax, eax
[0x00003cee]> ao
address: 0x3cee
opcode: call 0x14530
mnemonic: call
prefix: 0
id: 56
bytes: e83d080100
refptr: 0
size: 5
sign: false
type: call
cycles: 3
esil: 83248,rip,8,rsp,-=,rsp,=[],rip,=
jump: 0x00005382
direction: exec
fail: 0x00003cf3
stack: null
family: cpu
stackop: null
[0x00003cee]> ah
 0x00003cee - 0x00003cee => jump: 0x5382
```
As you can see, despite the unchanged disassembly view the jump address in opcode was changed
(`jump` option).

If anything of the previously described didn't help, you can simply override shown disassembly with anything you
like:
```
[0x00003d54]> pd 2
│          0x00003d54      0583000000     add eax, 10000011b
│          0x00003d59      3d13010000     cmp eax, 0x113
[0x00003d54]> "aho myopcode bla, foo"
[0x00003d54]> pd 2
│          0x00003d54                     myopcode bla, foo
│          0x00003d55      830000         add dword [rax], 0
```

