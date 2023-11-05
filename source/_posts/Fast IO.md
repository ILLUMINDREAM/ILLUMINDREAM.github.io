---
title: FastIO 模板
date: 2023-11-01
tag: 卡常
categories: 基础语法
top_img: https://pic.imgdb.cn/item/65475ddac458853aef0c1e3b.jpg
---

## Fast IO (fread, fwrite)

```cpp
#define endl '\n'
#define cin IO
#define cout IO

class FastIO
{
    char ibuf[1 << 16], obuf[1 << 16], *ipl = ibuf, *ipr = ibuf, *op = obuf;
    public:
    ~FastIO() { write(); }
    void read() { if(ipl == ipr) ipr = (ipl = ibuf) + fread(ibuf, 1, 1 << 15, stdin); }
    void write() { fwrite(obuf, 1, op - obuf, stdout), op = obuf; }
    char getchar() { return (read(), ipl != ipr) ? *ipl ++ : EOF; }
    void putchar(char c) { *op ++ = c; if(op - obuf > (1 << 15)) write(); }
    template <typename T> FastIO& operator >> (T &v)
    {
        int s = 1; char c = getchar(); v = 0;
        for( ; !isdigit(c); c = getchar()) if(c == '-') s = ~s + 1;
        for( ; isdigit(c); c = getchar()) v = (v << 1) + (v << 3) + (c ^ 48);
        return v *= s, *this;
    }
    FastIO& operator << (char c) { return putchar(c), *this; }
    template <typename T> FastIO& operator << (T v)
    {
        if(!v) putchar('0'); if(v < 0) putchar('-'), v = ~v + 1;
        char stk[100], *top = stk; for( ; v; v /= 10) *++top = v % 10 + '0';
        while(top != stk) putchar(*top -- ); return *this;
    }
} IO;
```

