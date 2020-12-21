# Level 01

## Login

```
ssh -p 2224 level1@io.netgarage.org
password: level1
```

## Task

### Solution

#### Brute-force

```shell
for i in {000..999}; do if [[ $(echo "$i" | ./level01 | wc -c) -gt 37 ]]; then (echo "$i" && cat) | ./level01; fi; done
```

Standard character count of ./leve01 is 37. If there are more characters than 37 execute the command again and go to the interactive shell.

#### Decompile

```asm
level1@io:/levels$ r2 -A level01 -c pdd -q
Warning: Cannot initialize dynamic strings
            ;-- main:
            ;-- section..text:
            ;-- main:
            ;-- _start:
/ (fcn) entry0 31
|   entry0 ();
|           0x08048080      6828910408     push str.Enter_the_3_digit_passcode_to_enter:_Congrats_you_found_it__now_read_the_password_for_level2_from__home_level2_.pass_n_bin_sh ; loc.prompt1 ; "Enter the 3 digit passcode to enter: Congrats you found it, now read the password for level2 from /home/level2/.pass./bin/sh" @ 0x8049128 ; [1] va=0x08048080 pa=0x00000080 sz=31 vsz=31 rwx=--r-x .text
|           0x08048085      e885000000     call loc.puts
|           0x0804808a      e810000000     call loc.fscanf
|           0x0804808f      3d0f010000     cmp eax, 0x10f              ; 271
|       ,=< 0x08048094      0f8442000000   je loc.YouWin
\       |   0x0804809a      e864000000     call loc.exit
        |   ;-- section_end..text:
        |   ;-- section..lib:
        |   ;-- fscanf:
        |   ; CALL XREF from 0x0804808a (entry0)
        |   0x0804809f      81ec00100000   sub esp, 0x1000             ; [2] va=0x0804809f pa=0x0000009f sz=134 vsz=134 rwx=--r-- .lib
        |   0x080480a5      b803000000     mov eax, 3
        |   0x080480aa      bb00000000     mov ebx, 0
        |   0x080480af      89e1           mov ecx, esp
        |   0x080480b1      ba00100000     mov edx, 0x1000
        |   0x080480b6      cd80           int 0x80
        |   0x080480b8      89e6           mov esi, esp
        |   0x080480ba      31db           xor ebx, ebx
       .--> ;-- skipwhite:
       .--> 0x080480bc      31c0           xor eax, eax
       ||   0x080480be      ac             lodsb al, byte [esi]
       ||   0x080480bf      3c20           cmp al, 0x20                ; 32
       `==< 0x080480c1      74f9           je loc.skipwhite
       .--> ;-- doit:
       .--> 0x080480c3      2c30           sub al, 0x30                ; '0'
       ||   0x080480c5      3c09           cmp al, 9                   ; 9
      ,===< 0x080480c7      770a           ja loc.exitscanf
      |||   0x080480c9      6bdb0a         imul ebx, ebx, 0xa
      |||   0x080480cc      01c3           add ebx, eax
      |||   0x080480ce      31c0           xor eax, eax
      |||   0x080480d0      ac             lodsb al, byte [esi]
      |`==< 0x080480d1      ebf0           jmp loc.doit
      `---> ;-- exitscanf:
      `---> 0x080480d3      89d8           mov eax, ebx
        |   0x080480d5      81c400100000   add esp, 0x1000
        |   0x080480db      c3             ret
        `-> ;-- YouWin:
        |   ; JMP XREF from 0x08048094 (entry0)
        `-> 0x080480dc      b804000000     mov eax, 4
            0x080480e1      bb01000000     mov ebx, 1
            0x080480e6      b94d910408     mov ecx, loc.prompt2        ; "Congrats you found it, now read the password for level2 from /home/level2/.pass./bin/sh" @ 0x804914d
            0x080480eb      ba50000000     mov edx, 0x50               ; 'P' ; 80
            0x080480f0      cd80           int 0x80
            0x080480f2      31c0           xor eax, eax
            0x080480f4      bb9d910408     mov ebx, loc.shell          ; "/bin/sh" @ 0x804919d
            0x080480f9      50             push eax
            0x080480fa      53             push ebx
            0x080480fb      89e1           mov ecx, esp
            0x080480fd      99             cdq
            0x080480fe      b00b           mov al, 0xb                 ; 11
            0x08048100      cd80           int 0x80
            0x08048102      c3             ret
            ;-- exit:
            ; CALL XREF from 0x0804809a (entry0)
            0x08048103      b801000000     mov eax, 1
            0x08048108      bb00000000     mov ebx, 0
            0x0804810d      cd80           int 0x80
            ;-- puts:
            ; CALL XREF from 0x08048085 (entry0)
            0x0804810f      8b4c2404       mov ecx, dword [esp + 4]    ; [0x4:4]=0x10101 ; 4
            0x08048113      b804000000     mov eax, 4
            0x08048118      bb01000000     mov ebx, 1
            0x0804811d      ba25000000     mov edx, 0x25               ; '%' ; 37
            0x08048122      cd80           int 0x80
            0x08048124      c3             ret
            ;-- section_end..lib:
            ;-- section_end.LOAD0:
            0x08048125      0000           add byte [eax], al
            0x08048127      00456e         add byte [ebp + 0x6e], al
        ,=< 0x0804812a      7465           je 0x8048191
       ,==< 0x0804812c      7220           jb 0x804814e
      ,===< 0x0804812e      7468           je 0x8048198
      |||   0x08048130      652033         and byte gs:[ebx], dh
      |||   0x08048133      20646967       and byte [ecx + ebp*2 + 0x67], ah
      |||   0x08048137      697420706173.  imul esi, dword [eax + 0x70], 0x63737361
      |||   0x0804813f      6f             outsd dx, dword [esi]
      |||   0x08048140      646520746f20   and byte gs:[edi + ebp*2 + 0x20], dh
      |||   0x08048146      656e           outsb dx, byte gs:[esi]
     ,====< 0x08048148      7465           je 0x80481af
    ,=====< 0x0804814a      723a           jb 0x8048186
    |||||   0x0804814c      20436f         and byte [ebx + 0x6f], al
    ||| |   0x0804814f      6e             outsb dx, byte [esi]
    |||,==< 0x08048150      677261         jb 0x80481b4
   ,======< 0x08048153      7473           je 0x80481c8
   ||||||   0x08048155      20796f         and byte [ecx + 0x6f], bh
  ,=======< 0x08048158      7520           jne 0x804817a
  |||||||   0x0804815a      666f           outsw dx, word [esi]
  ========< 0x0804815c      756e           jne 0x80481cc
  |||||||   0x0804815e      64206974       and byte fs:[ecx + 0x74], ch
  |||||||   0x08048162      2c20           sub al, 0x20
  |||||||   0x08048164      6e             outsb dx, byte [esi]
  |||||||   0x08048165      6f             outsd dx, dword [esi]
  ========< 0x08048166      7720           ja 0x8048188
  ========< 0x08048168      7265           jb 0x80481cf
  |||||||   0x0804816a      61             popal
  |||||||   0x0804816b      6420746865     and byte fs:[eax + ebp*2 + 0x65], dh
  |||||||   0x08048170      207061         and byte [eax + 0x61], dh
  ========< 0x08048173      7373           jae 0x80481e8
  ========< 0x08048175      776f           ja 0x80481e6
  ========< 0x08048177      7264           jb 0x80481dd
  |||||||   0x08048179      20666f         and byte [esi + 0x6f], ah
  ,=======< 0x0804817c      7220           jb 0x804819e
  |||||||   0x0804817e      6c             insb byte es:[edi], dx
  |||||||   0x0804817f      ff             invalid
  |||||||   0x08048180      ff             invalid
  |||||||   0x08048181      ff             invalid
  |||||||   0x08048182      ff             invalid
  |||||||   0x08048183      ff             invalid
  |||||||   0x08048184      ff             invalid
  |||||||   0x08048185      ff             invalid
  ||`-----> 0x08048186      ff             invalid
  || ||||   0x08048187      ff             invalid
  --------> 0x08048188      ff             invalid
  || ||||   0x08048189      ff             invalid
  || ||||   0x0804818a      ff             invalid
  || ||||   0x0804818b      ff             invalid
  || ||||   0x0804818c      ff             invalid
  || ||||   0x0804818d      ff             invalid
  || ||||   0x0804818e      ff             invalid
  || ||||   0x0804818f      ff             invalid
  || ||||   0x08048190      ff             invalid
  || |||`-> 0x08048191      ff             invalid
  || |||    0x08048192      ff             invalid
  || |||    0x08048193      ff             invalid
  || |||    0x08048194      ff             invalid
  || |||    0x08048195      ff             invalid
  || |||    0x08048196      ff             invalid
  || |||    0x08048197      ff             invalid
  || |`---> 0x08048198      ff             invalid
  || | |    0x08048199      ff             invalid
  || | |    0x0804819a      ff             invalid
  || | |    0x0804819b      ff             invalid
  || | |    0x0804819c      ff             invalid
  || | |    0x0804819d      ff             invalid
  `-------> 0x0804819e      ff             invalid
   | | |    0x0804819f      ff             invalid
   | | |    0x080481a0      ff             invalid
   | | |    0x080481a1      ff             invalid
   | | |    0x080481a2      ff             invalid
   | | |    0x080481a3      ff             invalid
   | | |    0x080481a4      ff             invalid
   | | |    0x080481a5      ff             invalid
   | | |    0x080481a6      ff             invalid
   | | |    0x080481a7      ff             invalid
   | | |    0x080481a8      ff             invalid
   | | |    0x080481a9      ff             invalid
   | | |    0x080481aa      ff             invalid
   | | |    0x080481ab      ff             invalid
   | | |    0x080481ac      ff             invalid
   | | |    0x080481ad      ff             invalid
   | | |    0x080481ae      ff             invalid
   | `----> 0x080481af      ff             invalid
   |   |    0x080481b0      ff             invalid
   |   |    0x080481b1      ff             invalid
   |   |    0x080481b2      ff             invalid
   |   |    0x080481b3      ff             invalid
   |   `--> 0x080481b4      ff             invalid
   |        0x080481b5      ff             invalid
   |        0x080481b6      ff             invalid
   |        0x080481b7      ff             invalid
   |        0x080481b8      ff             invalid
   |        0x080481b9      ff             invalid
   |        0x080481ba      ff             invalid
   |        0x080481bb      ff             invalid
   |        0x080481bc      ff             invalid
   |        0x080481bd      ff             invalid
   |        0x080481be      ff             invalid
   |        0x080481bf      ff             invalid
   |        0x080481c0      ff             invalid
   |        0x080481c1      ff             invalid
   |        0x080481c2      ff             invalid
   |        0x080481c3      ff             invalid
   |        0x080481c4      ff             invalid
   |        0x080481c5      ff             invalid
   |        0x080481c6      ff             invalid
   |        0x080481c7      ff             invalid
   `------> 0x080481c8      ff             invalid
            0x080481c9      ff             invalid
            0x080481ca      ff             invalid
            0x080481cb      ff             invalid
  --------> 0x080481cc      ff             invalid
            0x080481cd      ff             invalid
            0x080481ce      ff             invalid
  --------> 0x080481cf      ff             invalid
            0x080481d0      ff             invalid
            0x080481d1      ff             invalid
            0x080481d2      ff             invalid
            0x080481d3      ff             invalid
            0x080481d4      ff             invalid
            0x080481d5      ff             invalid
            0x080481d6      ff             invalid
            0x080481d7      ff             invalid
            0x080481d8      ff             invalid
            0x080481d9      ff             invalid
            0x080481da      ff             invalid
            0x080481db      ff             invalid
            0x080481dc      ff             invalid
  --------> 0x080481dd      ff             invalid
            0x080481de      ff             invalid
            0x080481df      ff             invalid
            0x080481e0      ff             invalid
            0x080481e1      ff             invalid
            0x080481e2      ff             invalid
            0x080481e3      ff             invalid
            0x080481e4      ff             invalid
            0x080481e5      ff             invalid
  --------> 0x080481e6      ff             invalid
            0x080481e7      ff             invalid
  --------> 0x080481e8      ff             invalid
            0x080481e9      ff             invalid
            0x080481ea      ff             invalid
            0x080481eb      ff             invalid
            0x080481ec      ff             invalid
            0x080481ed      ff             invalid
            0x080481ee      ff             invalid
            0x080481ef      ff             invalid
            0x080481f0      ff             invalid
            0x080481f1      ff             invalid
            0x080481f2      ff             invalid
            0x080481f3      ff             invalid
            0x080481f4      ff             invalid
            0x080481f5      ff             invalid
            0x080481f6      ff             invalid
            0x080481f7      ff             invalid
            0x080481f8      ff             invalid
            0x080481f9      ff             invalid
            0x080481fa      ff             invalid
            0x080481fb      ff             invalid
            0x080481fc      ff             invalid
            0x080481fd      ff             invalid
            0x080481fe      ff             invalid
            0x080481ff      ff             invalid
            0x08048200      ff             invalid
            0x08048201      ff             invalid
            0x08048202      ff             invalid
            0x08048203      ff             invalid
            0x08048204      ff             invalid
            0x08048205      ff             invalid
            0x08048206      ff             invalid
            0x08048207      ff             invalid
            0x08048208      ff             invalid
            0x08048209      ff             invalid
            0x0804820a      ff             invalid
            0x0804820b      ff             invalid
            0x0804820c      ff             invalid
            0x0804820d      ff             invalid
            0x0804820e      ff             invalid
            0x0804820f      ff             invalid
            0x08048210      ff             invalid
            0x08048211      ff             invalid
            0x08048212      ff             invalid
            0x08048213      ff             invalid
            0x08048214      ff             invalid
            0x08048215      ff             invalid
            0x08048216      ff             invalid
            0x08048217      ff             invalid
            0x08048218      ff             invalid
            0x08048219      ff             invalid
            0x0804821a      ff             invalid
            0x0804821b      ff             invalid
            0x0804821c      ff             invalid
            0x0804821d      ff             invalid
            0x0804821e      ff             invalid
            0x0804821f      ff             invalid
            0x08048220      ff             invalid
            0x08048221      ff             invalid
            0x08048222      ff             invalid
            0x08048223      ff             invalid
            0x08048224      ff             invalid
            0x08048225      ff             invalid
            0x08048226      ff             invalid
```

### Secret

```
sh-4.3$ cat /home/level2/.pass
XNWFtWKWHhaaXoKI
```