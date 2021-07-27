---
title: KMP
date: 2021-07-27 16:46:01
tags: 算法
---



<!-- more -->

<!-- t o c -->

&nbsp;

# 1. 原理

## 直观理解

如一个很好的例子，在aabaabaafa中，寻找aabaaf：

aabaabaafa

aabaaf

则两个指针同步出发，至idx=5时，指向了b和f。

如果是暴力，此时主串指针退回idx=1，子串指针退回idx=0，复杂度O(n\*m)。

观察可以发现，对于子串，若能一路匹配到f，则说明**在f前的两位，已经匹配成功了aa，恰好子串的开头也是aa，这说明主串指针不用动，子串指针只需回退到idx=2，指向b即可。**因为此时，主串指针前两位，和子串指针前面所有（也是两位），都是aa，可以看作是被自动匹配过了，只需继续比较主串所指的b和子串所指的b是否相等即可。

对子串指针的移动，可以看作是对子串的移动，上述移动后，两串对应如下：可以看出，只需从b开始比较即可，二者的aa必然相同，不需再次比较

aabaabaafa

___aabaaf

&nbsp;

## 前缀表与next数组

以上的分析是对两串同时分析，实际上关于如何回退（回退几步）的分析只针对子串即可进行。

如：对于aabaaf，当指到f时，就已说明主串此时所指的前两位是aa，与子串最前面的aa匹配，可以看作子串的开头aa，与主串f前的aa自动匹配过了，子串只需回退到b继续检测匹配。

又如：对于aabaaf，若指到b时发现与主串不匹配（如主串是aacaaf），同样不需要回退到最开始；因为此时主串所指的前一位一定是a，与子串开头一样是a，子串指针只需移动到idx=1，指向第二个a，继续比较即可。

可见，如何回退只对子串即可分析。那么，如何通过对子串的分析得出 当某一位匹配失败时，子串指针如何回退呢？答案是使用前缀表。

继续使用上例，主串aabaabaafa与子串aabaaf匹配，当idx=5，b与f不匹配时，我们发现当前指针的前面，aabaa，其后缀aa与前缀aa一致，长度为2，这也就是我们得出子串指针回退到idx=2，指向b的原因。

再看一个例子：

主串 aaaabcd

子串 aaaaxyz

子串指到idx=4，x时不匹配，此时应该回退到哪里呢？考虑指针所指的前半部分aaaa，其后缀aaa与前缀aaa一致，故可看作子串的前缀aaa已和主串b前的aaa自动完成匹配了，只需将子串指针回退到idx=3，用该处的a与主串的b继续匹配。

综上，可以看出，如何回退只需分析子串，并且第i位如何回退，是根据[0, i-1]得出的。那么回退到哪里呢？**回退到[0, i-1]中，最大长度的相等前缀后缀长度。**

如aabaaf，指到f时不匹配，则分析aabaa，其前缀有a、aa、aab、aaba（前缀不包含最后一位），后缀有a、aa、baa、abaa（后缀不包含首位），可见，两集合中最长相等是aa与aa，长度为2，所以子串指针回退时回退到前缀aa的后一位，即idx=2。

所以对于子串每一位，都可计算出若该位不匹配，回退到何处。回退位置由[0, i-1]中的最大相等前后缀长度确定。将这些数字记录下来，存入数组，这个数组习惯上命名为next。next[i]代表[0: i]中最大相等前后缀长度。

有了next数组后，若第i位匹配失败，则`j = next[j - 1]`，j回退，继续比较。

每次取i前一位的next值较为奇怪，于是有了将next数组整体右移一位的变体next，回退时`j = next[j]`即可。不过它的操作是对每个next的值-1，并且后续前缀表计算时也有部分修改，并不好理解，所以按下不提。原生next方便理解记忆，我觉得原生就挺好。

&nbsp;

## 前缀表如何计算

难道计算next时，要算出所有前缀和后缀，求交集再寻找最大值吗？那自然太麻烦了。对于KMP算法中的next计算，有两种理解方式，不过代码相同，此处细致说一种：通过一趟遍历，计算出子串中所有子子串的最大相等前后缀长度。（子子串，如aabaaf是子串，那么a、aa、aab、aaba、aabaa、aabaaf是其子子串，这是一个为了理解瞎叫的名字；一趟遍历也正对应这所有的子子串）

首先初始化next数组：`next[0] = 0`，因为只有一个字符时，其最大相等前后缀长度是0。

随后定义两个指针i和j，`i = 1, j = 0`，它们都指向子串。i代表[0: i]中的某一后缀，j代表[0: i]中的确切前缀[0: j]。这样1和0的初始化，就代表了长度为2的子子串（如上例的aa）中，前缀是idx=0处的a，后缀是idx=1处的a。

一趟遍历：`for(; i < strlen(string); i++)`

遍历内：若两字符匹配成功，`string[j] == string[i]`，代表两指针所代表的前后缀匹配成功，且此前后缀是最长相等前后缀，将`next[i] = ++j`（+1是因为j是下标，索引，而next内存长度）；若两字符匹配失败，j以next回退`j = next[j - 1]`，随后继续比较。

有两个问题：

为什么说两字符匹配成功时，所代表的的前后缀就是最长相等前后缀呢？考虑aaaa，指针i=1，j=0，一路成功下去，i=3，j=2，此时两字符相等，所代表的是前缀aaa和后缀aaa，是最长相等前后缀。由此可以看出，**当匹配成功时，j也在向后移动，会将匹配成功的长度累计下来，故而成功时是最大长度**。

为什么两字符匹配失败时，j要以next回退？如前一个问题，j肩负着记录最长匹配前缀的责任，如果j永远直接返回首位，则会丢失记录的前缀信息；而next中正包含着[0: i-1]中最长相等前后缀长度，故用next回退，可以保留[0: i-1]的最长相等前后缀，随后与string[i]的继续比较时，仍能代表最大长度。（这与KMP比较主串与子串的思路类似，所以也有直接将求next数组的过程看作 以子串为主串，以子串的前缀为子串，进行字符串匹配，这就是求next数组的另一种理解方式）

另：代码中，不管是求next，还是正式的利用next进行两串匹配，都是先判断处理匹配失败的情况，在while循环中将j不断回退，随后才处理匹配成功的情况。因为不成功的情况下，回退可能会使结果变为成功。

&nbsp;

# 2. 代码

## C

```c
// 说明
void getRawNext (char * string, int * next) {
    // 初始化
    next[0] = 0;
    int i = 1, j = 0;

    // 用前缀匹配模式串
    for (; i < strlen(string); i++) {
        while (j > 0 && string[i] != string[j]) // 匹配失败；next[0]=0，j一路左移至最前时j=next[1-1]=0,继续j=next[0-1]越界，故要求j>0，即匹配串最多右移至头部
            j = next[j-1];                      // 利用前面已构建好next数组，匹配串右移/游标左移
//        if (string[i] == string[j])           // 匹配成功
//            next[i] = ++j;                    // next记录的最长相等值为j+1（j从0开始故+1）
//        else                                  // j一路回退至0，仍不等，则next记为0
//            next[i] = j;
        // 将上面的if else优化一下：
        if (string[i] == string[j]) j++;	    // 匹配成功，当前最长前后缀相等长度+1
        next[i] = j;	                        // 更新next数组
    }
}

void getRightNext (char * str, int * next) {
    next[0] = -1;
    next[1] = 0;
    int i = 1, j = 0;
    for (; i < strlen(str)-1; i++) {    // 计算结果存入后一位，故len-1
        while (j>-1 && str[i]!=str[j])
            j = next[j];
//        if (str[i] == str[j]){    // 匹配成功，next后一位记录最大匹配长度，因j从0开始故+1
//            next[i+1] = ++j;
//        } else {                  // 匹配失败，由while知此时必是匹配串移动到最前，j=-1
//            j = 0;                // 故next后一位存入0
//            next[i+1] = j;
//        }
        // 将上面if else优化
        next[i+1] = ++j;
    }
} 
// search str2 in str1
int KMP (char * str1, char * str2) {
    if (strlen(str2) == 0) return 0;
    int * next = (int*) malloc(sizeof(int) * strlen(str2));
    getRawNext(str2, next);  // raw next
    // getRightNext(str2, next);   // right next
    int i = 0, j = 0;
    while (i < strlen(str1) && j < strlen(str2)) {
        while (j>0 && str1[i] != str2[j])
            j = next[j-1];    // raw next
            // j = next[j];        // right next
        if (str1[i] == str2[j])
            j++;
        i++;
    }
    free(next);
    if (j == strlen(str2))
        return i-(int)strlen(str2);
    else
        return -1;
}
```

&nbsp;

## Java

```java
public class Solution {
    public int strStr(String haystack, String needle) {
        char[] str2 = needle.toCharArray();
        if (str2.length == 0) {
            return 0;
        }
        char[] str1 = haystack.toCharArray();
        int[] next = getRawNext(needle.toCharArray());
        int i = 0, j = 0;
        while (i < str1.length && j < str2.length) {
            while (j > 0 && str1[i] != str2[j]) {
                j = next[j - 1];
            }
            if (str1[i] == str2[j]) {
                j++;
            }
            i++;
        }

        if (j == str2.length) {
            return i - str2.length;
        } else {
            return -1;
        }

    }

    private int[] getRawNext(char[] string) {
        int[] next = new int[string.length];
        next[0] = 0;
        for (int i = 1, j = 0; i < string.length; i++) {
            while (j > 0 && string[i] != string[j]) {
                j = next[j - 1];
            }

            if (string[i] == string[j]) {
                next[i] = ++j;
            } else {
                next[i] = j;
            }
        }

        return next;
    }
}
```

