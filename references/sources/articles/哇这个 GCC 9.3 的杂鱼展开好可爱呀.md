来源：[cyffff 的神秘挂分](https://www.luogu.com.cn/article/q2usylhj)。

那我可就来兴趣了啊。直接拿来他的代码丢进 [Compiler Explorer](https://godbolt.org/) 里面编译，使用 GCC 9.3，开 O2 优化，启动！

*若干秒之后*

温馨提醒：上面的所有东西都是从同一个函数里炸出来的：

```cpp
inline int query(int rt,int l,int r,int p){
    if(!rt) return 0;
    if(l==r) return a[rt].tag;
    int mid=l+r>>1;
    if(p<=mid) return query(ls,l,mid,p)+a[rt].tag;
    else return query(rs,mid+1,r,p)+a[rt].tag;
}
```

不行赶紧关掉 O2，它在干啥啊？？？

*若干秒之后*

```wasm
Segment_Tree::query(int, int, int, int):
        push    rbp
        mov     rbp, rsp
        sub     rsp, 48
        mov     QWORD PTR [rbp-24], rdi
        mov     DWORD PTR [rbp-28], esi
        mov     DWORD PTR [rbp-32], edx
        mov     DWORD PTR [rbp-36], ecx
        mov     DWORD PTR [rbp-40], r8d
        cmp     DWORD PTR [rbp-28], 0
        jne     .L42
        mov     eax, 0
        jmp     .L43
.L42:
        mov     eax, DWORD PTR [rbp-32]
        cmp     eax, DWORD PTR [rbp-36]
        jne     .L44
        mov     rcx, QWORD PTR [rbp-24]
        mov     eax, DWORD PTR [rbp-28]
        movsx   rdx, eax
        mov     rax, rdx
        add     rax, rax
        add     rax, rdx
        sal     rax, 2
        add     rax, rcx
        add     rax, 8
        mov     eax, DWORD PTR [rax]
        jmp     .L43
.L44:
        mov     edx, DWORD PTR [rbp-32]
        mov     eax, DWORD PTR [rbp-36]
        add     eax, edx
        sar     eax
        mov     DWORD PTR [rbp-4], eax
        mov     eax, DWORD PTR [rbp-40]
        cmp     eax, DWORD PTR [rbp-4]
        jg      .L45
        mov     rcx, QWORD PTR [rbp-24]
        mov     eax, DWORD PTR [rbp-28]
        movsx   rdx, eax
        mov     rax, rdx
        add     rax, rax
        add     rax, rdx
        sal     rax, 2
        add     rax, rcx
        mov     esi, DWORD PTR [rax]
        mov     edi, DWORD PTR [rbp-40]
        mov     ecx, DWORD PTR [rbp-4]
        mov     edx, DWORD PTR [rbp-32]
        mov     rax, QWORD PTR [rbp-24]
        mov     r8d, edi
        mov     rdi, rax
        call    Segment_Tree::query(int, int, int, int)
        mov     ecx, eax
        mov     rsi, QWORD PTR [rbp-24]
        mov     eax, DWORD PTR [rbp-28]
        movsx   rdx, eax
        mov     rax, rdx
        add     rax, rax
        add     rax, rdx
        sal     rax, 2
        add     rax, rsi
        add     rax, 8
        mov     eax, DWORD PTR [rax]
        add     eax, ecx
        jmp     .L43
.L45:
        mov     eax, DWORD PTR [rbp-4]
        lea     edi, [rax+1]
        mov     rcx, QWORD PTR [rbp-24]
        mov     eax, DWORD PTR [rbp-28]
        movsx   rdx, eax
        mov     rax, rdx
        add     rax, rax
        add     rax, rdx
        sal     rax, 2
        add     rax, rcx
        add     rax, 4
        mov     esi, DWORD PTR [rax]
        mov     ecx, DWORD PTR [rbp-40]
        mov     edx, DWORD PTR [rbp-36]
        mov     rax, QWORD PTR [rbp-24]
        mov     r8d, ecx
        mov     ecx, edx
        mov     edx, edi
        mov     rdi, rax
        call    Segment_Tree::query(int, int, int, int)
        mov     ecx, eax
        mov     rsi, QWORD PTR [rbp-24]
        mov     eax, DWORD PTR [rbp-28]
        movsx   rdx, eax
        mov     rax, rdx
        add     rax, rax
        add     rax, rdx
        sal     rax, 2
        add     rax, rsi
        add     rax, 8
        mov     eax, DWORD PTR [rax]
        add     eax, ecx
.L43:
        leave
        ret
```

那是不是旧的编译器太笨了啊？换个编译器试试（选中 GCC 15.2）。

*若干秒之后*

```wasm
Segment_Tree::query(int, int, int, int):
        xor     eax, eax
        test    esi, esi
        je      .L366
        cmp     edx, ecx
        je      .L370
        mov     r10d, ecx
        movsx   rsi, esi
        lea     ecx, [rdx+rcx]
        sub     rsp, 24
        lea     rax, [rsi+rsi*2]
        sar     ecx
        lea     r9, [rdi+rax*4]
        cmp     ecx, r8d
        jl      .L350
        movsx   rax, DWORD PTR [r9]
        test    eax, eax
        je      .L354
        cmp     edx, ecx
        je      .L368
        lea     r10d, [rdx+rcx]
        lea     rax, [rax+rax*2]
        mov     QWORD PTR [rsp+8], r9
        sar     r10d
        lea     r11, [rdi+rax*4]
        cmp     r8d, r10d
        jg      .L353
        mov     esi, DWORD PTR [r11]
.L369:
        mov     ecx, r10d
        mov     QWORD PTR [rsp], r11
        call    Segment_Tree::query(int, int, int, int)
        mov     r11, QWORD PTR [rsp]
        mov     r9, QWORD PTR [rsp+8]
        add     eax, DWORD PTR [r11+8]
.L354:
        add     eax, DWORD PTR [r9+8]
        add     rsp, 24
        ret
.L350:
        movsx   rax, DWORD PTR [r9+4]
        test    eax, eax
        je      .L354
        lea     edx, [rcx+1]
        cmp     edx, r10d
        je      .L368
        lea     ecx, [rdx+r10]
        mov     QWORD PTR [rsp+8], r9
        lea     rax, [rax+rax*2]
        sar     ecx
        cmp     r8d, ecx
        jg      .L356
        lea     r10, [rdi+rax*4]
        mov     esi, DWORD PTR [r10]
        mov     QWORD PTR [rsp], r10
        call    Segment_Tree::query(int, int, int, int)
        mov     r10, QWORD PTR [rsp]
        mov     r9, QWORD PTR [rsp+8]
        add     eax, DWORD PTR [r10+8]
        jmp     .L354
.L366:
        ret
.L370:
        movsx   rsi, esi
        lea     rax, [rsi+rsi*2]
        mov     eax, DWORD PTR [rdi+8+rax*4]
        ret
.L368:
        lea     rax, [rax+rax*2]
        mov     eax, DWORD PTR [rdi+8+rax*4]
        jmp     .L354
.L356:
        lea     r11, [rdi+rax*4]
        lea     edx, [rcx+1]
        mov     esi, DWORD PTR [r11+4]
        jmp     .L369
.L353:
        mov     esi, DWORD PTR [r11+4]
        lea     edx, [r10+1]
        mov     QWORD PTR [rsp], r11
        call    Segment_Tree::query(int, int, int, int)
        mov     r11, QWORD PTR [rsp]
        mov     r9, QWORD PTR [rsp+8]
        add     eax, DWORD PTR [r11+8]
        jmp     .L354
```

欸你说它展开是不是因为它看到了那个 `inline` 啊？把配置调回去删掉 `inline` 试试。

*若干秒之后*

你完全不听是吧？还在展开？

---

评价是某协会都 2025 年了有 GCC 15.2 了还在用 GCC 9.3 完全就是不知所云。不过对于普通选手的我们还有救吗？完全不知道，还是听天由命吧。
