---
layout: post
title:  Algorithms——字符串相关库函数
categories: [APUE]
---


###strstr库函数

```cpp
//在字符串中查找指定字符串的第一次出现，不能找到则返回-1        
int strstr(char *string, char *substring) {       
    if (string == NULL || substring == NULL)          
        return -1;          
  
    int lenstr = strlen(string);       
    int lensub = strlen(substring);       
  
    if (lenstr < lensub)          
        return -1;
  
    int len = lenstr - lensub;
    int i,j;
    for (i = 0; i <= len; i++) {       
        for (j = 0; j < lensub; j++) {       
            if (string[i+j] != substring[j])       
                break;       
        }       
        if (j == lensub)       
            return i + 1; 
    }       
    return -1; 
}
```

###strcpy库函数

```cpp
//得2分       
void strcpy( char *strDest, char *strSrc ) {       
    while( (*strDest++ = * strSrc++) != '\0' ); 
}        
  
//得4分       
void strcpy( char *strDest, const char *strSrc ) {       
    //将源字符串加const，表明其为输入参数，加2分       
    while( (*strDest++ = * strSrc++) != '\0' );       
}        
  
//得7分       
void strcpy(char *strDest, const char *strSrc) {       
    //对源地址和目的地址加非0断言，加3分       
    assert( (strDest != NULL) && (strSrc != NULL) );       
    while( (*strDest++ = * strSrc++) != '\0' );       
}        
  
//得9分       
//为了实现链式操作，将目的地址返回，加2分！       
char * strcpy( char *strDest, const char *strSrc ) {       
    assert( (strDest != NULL) && (strSrc != NULL) );       
    char *address = strDest;        
    while( (*strDest++ = * strSrc++) != '\0' );        
    return address;       
}      
  
//得10分，基本上所有的情况，都考虑到了    
//如果有考虑到源目所指区域有重叠的情况，加1分！       
char * strcpy( char *strDest, const char *strSrc ) {       
    if(strDest == strSrc) { return strDest; }    
    assert( (strDest != NULL) && (strSrc != NULL) );       
    char *address = strDest;        
    while( (*strDest++ = * strSrc++) != '\0' );        
    return address;       
} 
```
###其他

```cpp
//@yansha:字串末尾要加结束符'\0'，不然输出错位结果    
char *strncpy(char *strDes, const char *strSrc, unsigned int count) {        
    assert(strDes != NULL && strSrc != NULL);        
    char *address = strDes;        
    while (count-- && *strSrc != '\0')        
        *strDes++ = *strSrc++;     
    *strDes = '\0';    
    return address;        
}     
  
//查找字符串s中首次出现字符c的位置     
char *strchr(const char *str, int c) {     
    assert(str != NULL);     
    for (; *str != (char)c; ++ str)     
        if (*str == '\0')     
            return NULL;     
        return str;     
}     
  
int strcmp(const char *s, const char *t) {     
    assert(s != NULL && t != NULL);     
    while (*s && *t && *s == *t) {     
        ++ s;     
        ++ t;     
    }     
    return (*s - *t);     
}     
  
char *strcat(char *strDes, const char *strSrc) {     
    assert((strDes != NULL) && (strSrc != NULL));     
    char *address = strDes;     
    while (*strDes != '\0')     
        ++ strDes;     
    while ((*strDes ++ = *strSrc ++) != '\0')     
        NULL;     
    return address;     
}     
  
int strlen(const char *str) {     
    assert(str != NULL);     
    int len = 0;     
    while (*str ++ != '\0')     
        ++ len;     
    return len;     
}     
  
//此函数，修改如下       
char *strdup_(char *strSrc)       
//将字符串拷贝到新的位置       
{       
    if(strSrc!=NULL) {       
        char *start=strSrc;       
        int len=0;       
        while(*strSrc++!='\0')       
            len++;       
  
        char *address=(char *)malloc(len+1);       
        assert(address != NULL);    
  
        while((*address++=*start++)!='\0');        
        return address-(len+1);        
    }       
    return NULL;       
}       
  
char *strstr(const char *strSrc, const char *str) {     
    assert(strSrc != NULL && str != NULL);     
    const char *s = strSrc;     
    const char *t = str;     
    for (; *strSrc != '\0'; ++ strSrc) {     
        for (s = strSrc, t = str; *t != '\0' && *s == *t; ++s, ++t)     
            NULL;     
        if (*t == '\0')     
            return (char *) strSrc;     
    }     
    return NULL;     
}     
  
char *strncat(char *strDes, const char *strSrc, unsigned int count) {     
    assert((strDes != NULL) && (strSrc != NULL));     
    char *address = strDes;     
    while (*strDes != '\0')     
        ++ strDes;     
    while (count -- && *strSrc != '\0' )     
        *strDes ++ = *strSrc ++;     
    *strDes = '\0';     
    return address;     
}     
  
int strncmp(const char *s, const char *t, unsigned int count) {     
    assert((s != NULL) && (t != NULL));     
    while (*s && *t && *s == *t && count --) {     
        ++ s;     
        ++ t;     
    }     
    return (*s - *t);     
}     
  
char *strpbrk(const char *strSrc, const char *str) {     
    assert((strSrc != NULL) && (str != NULL));     
    const char *s;     
    while (*strSrc != '\0') {     
        s = str;     
        while (*s != '\0') {     
            if (*strSrc == *s)     
                return (char *) strSrc;     
            ++ s;     
        }     
        ++ strSrc;     
    }     
    return NULL;     
}     
  
int strcspn(const char *strSrc, const char *str) {     
    assert((strSrc != NULL) && (str != NULL));     
    const char *s;     
    const char *t = strSrc;     
    while (*t != '\0') {     
        s = str;     
        while (*s != '\0') {     
            if (*t == *s)     
                return t - strSrc;     
            ++ s;     
        }     
        ++ t;     
    }     
    return 0;     
}     
  
int strspn(const char *strSrc, const char *str) {     
    assert((strSrc != NULL) && (str != NULL));     
    const char *s;     
    const char *t = strSrc;     
    while (*t != '\0') {     
        s = str;     
        while (*s != '\0') {     
            if (*t == *s)     
                break;     
            ++ s;     
        }     
        if (*s == '\0')     
            return t - strSrc;     
        ++ t;     
    }     
    return 0;     
}     
  
char *strrchr(const char *str, int c) {     
    assert(str != NULL);     
    const char *s = str;     
    while (*s != '\0')     
        ++ s;     
    for (-- s; *s != (char) c; -- s)     
        if (s == str)     
            return NULL;     
        return (char *) s;     
}     
  
char* strrev(char *str) {     
    assert(str != NULL);     
    char *s = str, *t = str, c;     
    while (*t != '\0')     
        ++ t;     
    for (-- t; s < t; ++ s, -- t) {     
        c = *s;     
        *s = *t;     
        *t = c;     
    }     
    return str;     
}     
  
char *strnset(char *str, int c, unsigned int count) {     
    assert(str != NULL);     
    char *s = str;     
    for (; *s != '\0' && s - str < count; ++ s)     
        *s = (char) c;     
    return str;     
}     
  
char *strset(char *str, int c) {     
    assert(str != NULL);     
    char *s = str;     
    for (; *s != '\0'; ++ s)     
        *s = (char) c;     
    return str;     
}     

//对原 strtok 的修改，根据MSDN,strToken可以为NULL.实际上第一次call strtok给定一字串，    
//再call strtok时可以输入NULL代表要接着处理给定字串。    
//所以需要用一 static 保存没有处理完的字串。同时也需要处理多个分隔符在一起的情况。    
char *strtok(char *strToken, const char *str) {    
    assert(str != NULL);    
    static char *last;    
  
    if (strToken == NULL && (strToken = last) == NULL)    
        return (NULL);    
  
    char *s = strToken;    
    const char *t = str;    
    while (*s != '\0') {    
        t = str;    
        while (*t != '\0') {    
            if (*s == *t) {    
                last = s + 1;    
                if (s - strToken == 0) {    
                    strToken = last;    
                    break;    
                }    
                *(strToken + (s - strToken)) = '\0';    
                return strToken;    
            }    
            ++ t;    
        }    
        ++ s;    
    }    
    return NULL;    
}    
  
char *strupr(char *str) {     
    assert(str != NULL);     
    char *s = str;     
    while (*s != '\0') {     
        if (*s >= 'a' && *s <= 'z')     
            *s -= 0x20;     
        s ++;     
    }     
    return str;     
}     
  
char *strlwr(char *str) {     
    assert(str != NULL);     
    char *s = str;     
    while (*s != '\0') {     
        if (*s >= 'A' && *s <= 'Z')     
            *s += 0x20;     
        s ++;     
    }     
    return str;     
}     
  
void *memcpy(void *dest, const void *src, unsigned int count) {     
    assert((dest != NULL) && (src != NULL));     
    void *address = dest;     
    while (count --) {     
        *(char *) dest = *(char *) src;     
        dest = (char *) dest + 1;     
        src = (char *) src + 1;     
    }     
    return address;     
}     
  
void *memccpy(void *dest, const void *src, int c, unsigned int count) {     
    assert((dest != NULL) && (src != NULL));     
    while (count --) {     
        *(char *) dest = *(char *) src;     
        if (* (char *) src == (char) c)     
            return ((char *)dest + 1);     
        dest = (char *) dest + 1;     
        src = (char *) src + 1;     
    }     
    return NULL;     
}     
  
void *memchr(const void *buf, int c, unsigned int count) {     
    assert(buf != NULL);     
    while (count --) {     
        if (*(char *) buf == c)     
            return (void *) buf;     
        buf = (char *) buf + 1;     
    }     
    return NULL;     
}     
  
int memcmp(const void *s, const void *t, unsigned int count) {     
    assert((s != NULL) && (t != NULL));     
    while (*(char *) s && *(char *) t && *(char *) s == *(char *) t && count --) {     
        s = (char *) s + 1;     
        t = (char *) t + 1;     
    }     
    return (*(char *) s - *(char *) t);     
}     

  
//要处理src和dest有重叠的情况，不是从尾巴开始移动就没问题了。    
//一种情况是dest小于src有重叠，这个时候要从头开始移动，    
//另一种是dest大于src有重叠，这个时候要从尾开始移动。    
void *memmove(void *dest, const void *src, unsigned int count) {    
    assert(dest != NULL && src != NULL);    
    char* pdest = (char*) dest;    
    char* psrc = (char*) src;    
  
    //pdest在psrc后面，且两者距离小于count时，从尾部开始移动. 其他情况从头部开始移动    
    if (pdest > psrc && pdest - psrc < count) {    
        while (count--) {    
            *(pdest + count) = *(psrc + count);    
        }    
    } 
    else {    
        while (count--) {    
            *pdest++ = *psrc++;    
        }    
    }    
    return dest;    
}    
  
void *memset(void *str, int c, unsigned int count) {     
    assert(str != NULL);     
    void *s = str;     
    while (count --) {     
        *(char *) s = (char) c;     
        s = (char *) s + 1;     
    }     
    return str;     
}
```

###string类赋值构造函数

```cpp
class myString {
pbulic:
    myString(char *pData = NULL);
    myString(const myString &str);
    ~myString();

private:
    char* m_pData;
}

myString& myString::operator =(const myString &str) {
    if(this == &str)
        return *this;
    delete []m_pData;
    m_pData = NULL;
    m_pData = new char[strlen(str.m_pData) + 1];
    strcpy(m_pData, str.m_pData);
    
    return *this;
}
```
