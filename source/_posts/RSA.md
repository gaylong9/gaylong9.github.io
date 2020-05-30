---
title: RSA
date: 2020-05-30 20:09:02
tags: 信息安全
---



信息安全技术实验4

<!--more-->



<!-- toc -->



<br/>

[toc]

# RSA

## 单向陷门函数与公钥密码

* 单向陷门函数：

1. 给定$x$，计算$y=f(x)$是容易的；
2. 给定$y$, 计算$x$使$x=f-1(y)$是不可行的；
3. 存在陷门$t$，已知$t$时，对给定的任何$y$，若相应的原象$x$存在，则计算$x$是容易的。

* 加密：使用函数$f(x)$作为公钥，陷门$t$作为私钥。加密明文$m$得到密文$y=f(m)$。仅有知道私钥时可求出原文。

<br/>

## 欧拉函数

1. 在$[1,n]$中，不大于$n$，且与其互素的正整数的个数，记为$\phi(x)$
2. $p$为素数时， $\phi(p^k)=p^{k-1}(p-1)$
3. 若$(m_1, m_2)=1$，则$\phi(m_1 m_2)=\phi(m_1) \phi(m_2)$
4. 若自变量为普通数字，则可改写为素数幂的乘积的形式。之后由3.中公式分解。

<br/>

## RSA步骤

1. 随机生成两个不同大素数$p$, $q$;
2. 计算$n=pq$, $\phi(n)=(p-1)(q-1)$;
3. 随机选取整数$e$, $1<e<\phi(n)$,满足$(e,\phi(n))=1$;
4. 利用扩展欧基里德算法求出满足$ed=1 mod(\phi(n))$的整数$d$;
5. 公开$(n,e)$，保密$(p,q,\phi(n),d)$。其中$e$就是加密密钥，而$d$就是解密密钥，$n$称为模数；
6. 加密：$c=m^e \ mod \ n$ （若明文长度超过$n$，则分组）
7. 解密：$m=c^d \ mod \ n$

<br/>

## 使用mpir实现短数据加解密

```c++
//随机生成一个1024素数
void gen_prime(mpz_t prime)  
{                                         
    gmp_randstate_t grt;                  
    gmp_randinit_default(grt);    
    gmp_randseed_ui(grt, time(NULL));     
      
    mpz_t p;   
    mpz_init(p);
    mpz_urandomb(p, grt, 1024); //随机生成1024位的大整数                
    mpz_nextprime(prime, p);  //使用GMP自带的素数生成函数  
    mpz_clear(p);   
}

int _tmain(int argc, _TCHAR* argv[])
{
	mpz_t p, q, n, e, d, phi, sp, sq, m, c, m1;
	mpz_t one, a, b;
	mpz_init(n);
	mpz_init(e);
	mpz_init(d);
	mpz_init(phi);
	mpz_init(sp);
	mpz_init(sq);
	mpz_init(a);
	mpz_init(b);
	mpz_init(one);
        mpz_init(p);
	mpz_init(q);
	mpz_init(m);
	mpz_init(c);
	mpz_init(m1);

	// 生成p和q
	gen_prime(p);
	gmp_printf ("p:\n%Zd\n\n", p);
	Sleep(500);	// 以防因时间相同导致生成的pq相同
	gen_prime(q);
        gmp_printf ("q:\n%Zd\n\n", q);

	// 计算n和phi(n)
	mpz_sub_ui(sp, p, 1);
	mpz_sub_ui(sq, q, 1);
	mpz_mul(n, p, q);
	mpz_mul(phi, sp, sq);
	
	// 选取整数e（公钥）
	mpz_set_ui(e, 65537);

	// 求私钥d
	mpz_invert(d, e, phi);

	// 短数据加密
	char * message = "1234567890ABCDEF";
	mpz_set_str(m, message, 16);
	gmp_printf("原文:\n%Zd\n", m);
	mpz_powm(c, m, e, n);
	gmp_printf("\n密文:\n%Zd\n", c);

	// 短数据解密
	mpz_powm(m1, c, d, n);
	gmp_printf("\n解密:\n%Zd\n\n", m1);
 
	return 0;
}
```



![](RSA/c62d183717d174ae73fe66980c67833b_wallhaven-gjvg6l.png)