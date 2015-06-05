---
layout: post
title:  彻底理解KMP算法
categories: [Algorithms]
---

###暴力匹配算法

假设现在我们面临这样一个问题：有一个文本串S，和一个模式串P，现在要查找P在S中的位置，怎么查找呢？

如果用暴力匹配的思路，并假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置，则有：

- 如果当前字符匹配成功（即S[i] == P[j]），则i++，j++，继续匹配下一个字符；
- 如果失配（即S[i]! = P[j]），令i = i - (j - 1)，j = 0。相当于每次匹配失败时，i 回溯，j 被置为0。

理清楚了暴力匹配算法的流程及内在的逻辑，咱们可以写出暴力匹配的代码，如下：

```cpp
vector<int> ViolentMatch(char* s, char* p) {  
    vector<int> result;
    
    int sLen = strlen(s);  
    int pLen = strlen(p);  
  
    int i = 0;  
    int j = 0;  
    while (i < sLen) {  
        if (s[i] == p[j]) {
            //①如果当前字符匹配成功（即S[i] == P[j]），则i++，j++      
            i++;  
            j++;  
        }  
        else {  
            //②如果失配（即S[i]! = P[j]），令i = i - (j - 1)，j = 0      
            i = i - j + 1;  
            j = 0;  
        }
        if(j == pLen) {
            result.push_back(i - j);
            i = i - j + 1;  
            j = 0;
        }
    }
	return result;
}
```

###KMP算法

假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置

-   如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++，继续匹配下一个字符；
-   如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]。此举意味着失配时，模式串P相对于文本串S向右移动了j - next [j] 位。
    -   换言之，当匹配失败时，模式串向右移动的位数为：失配字符所在位置 - 失配字符对应的next 值（next 数组的求解会在下文中详细阐述），即移动的实际位数为：j - next[j]，且此值大于等于1。


