---
title: 字符串匹配技术之 Horspool算法和Boyer-Moore算法
tags: ["string-matching"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-10-15 18:12:24
thumbnail:
---



字符串匹配问题要求在一个较长的字符串中，匹配一个较短的字符串，找到短字符串在长字符串中第一次出现的位置。本文将介绍 Boyer-Moore 算法及其简化版本 Horspool 算法。

<!--more-->

# 问题

在一个被称为**文本(text)**的n个字符的串中，寻找一个被称为**模式(pattern)**的给定m个字符的串。



# Horspool 算法



## 代码实现

### C++ 实现

```c++
#include <iostream>
#include <map>
#include <string.h>

class ShiftTable {
public:
    ShiftTable(const char* pattern) {
        len = strlen(pattern);
        for (int i = 0; i < len - 1; i++) {
            table[pattern[i]] =  len - 1 - i;
        }
    }

    int Get(char c) {
        return (table.find(c) == table.end() ? len : table[c]);
    }

    int operator[](char c) {
        return Get(c);    
    }

private:
    std::map<char, int> table;
    int len;
};

int HorspoolMatching(const char* pattern,const char* text) {
    int lenPattern = strlen(pattern);
    int lenText = strlen(text);

    // text长度比pattern小时text必定不包含pattern
    if (lenText < lenPattern) { 
        return -1;
    }

    // 构建移动表
    ShiftTable table(pattern);

    int i = lenPattern - 1;
    while (i < lenText) {
        int k = 0;
        while (k <= lenPattern - 1 && pattern[lenPattern - 1 - k] == text[i - k]) {
            k++;
        }

        if (k == lenPattern) {
            return i - lenPattern + 1;
        }
        else {
            i += table[text[i]];
        }
    }
    return -1;
}

void Test() {
    std::cout << HorspoolMatching("cde", "abcdefg") << std::endl;
}

int main() {
    Test();
    return 0;
}
```



# Boyer-Moore 算法



## 代码实现

### C++ 实现

```c++
#include <iostream>
#include <map>
#include <string.h>

int Max(int a, int b) {
    return a > b ? a : b;
}

int Min(int a, int b) {
    return a > b ? b : a;
}

class ShiftTable {
public:
    ShiftTable(const char* pattern) {
        len = strlen(pattern);
        for (int i = 0; i < len - 1; i++) {
            table[pattern[i]] =  len - 1 - i;
        }
    }

    int Get(char c) {
        return (table.find(c) == table.end() ? len : table[c]);
    }

    int operator[](char c) {
        return Get(c);    
    }

private:
    std::map<char, int> table;
    int len;
};

class BadSymbolShiftTable: public ShiftTable {
public:
    BadSymbolShiftTable(const char* pattern): ShiftTable(pattern) {}

    int Get(char c, int k) {
        return Max(ShiftTable::Get(c) - k, 1);
    }
};

class GoodSuffixShiftTable {
public:
    GoodSuffixShiftTable(const char* pattern) {
        len = strlen(pattern);
        table = new int[len];
        
        table[0] = len;

        char* filpPattern = FlipString(pattern);
        for (int k = 1; k < len; k++) { 
            char* filpPostPattern = FlipString(&pattern[len - k]);
            table[k] = HorspoolMatchingWithSuffix(filpPostPattern, &filpPattern[1]) + 1;
            if (table[k] == 0) {
                table[k] = len;
            }
            delete[] filpPostPattern;
        }
        delete[] filpPattern;
    }

    ~GoodSuffixShiftTable() {
        delete[] table;
    }

    int Get(int k) {
        if (0 <= k < len) {
            return table[k];
        }
        else {
            throw "GoodShuffixShiftTable index should be in [0, len).";
        }
    }

    int operator[](int k) {
        return Get(k);
    }

private:
    char* FlipString(const char* str) {
        int len = strlen(str);
        char* flipString = new char[len + 1];
        for (int i = 0; i < len; i++) {
            flipString[len - 1 - i] = str[i];
        }
        flipString[len] = '\0';
        return flipString;
    }

    int HorspoolMatchingWithSuffix(const char* pattern,const char* text) {
        int lenPattern = strlen(pattern);
        int lenText = strlen(text);

        // text长度比pattern小时text必定不包含pattern
        if (lenText < lenPattern) { 
            return -1;
        }

        // 构建移动表
        ShiftTable table(pattern);

        // text边界处只匹配pattern的前缀也判定为匹配
        int i = lenPattern - 1;
        while (i < lenText + lenPattern - 1) {
            int k = Max(0, i - lenText + 1); 
            while (k <= lenPattern - 1 && pattern[lenPattern - 1 - k] == text[i - k]) {
                k++;
            }

            if (k == Min(lenPattern, i + 1)) {
                return i - lenPattern + 1;
            }
            else {
                i += table[text[i]];
            }
        }
        return -1;
    }

private:
    int* table;
    int len;
};

int BoyerMooreMatching(const char* pattern,const char* text) {
    int lenPattern = strlen(pattern);
    int lenText = strlen(text);

    // text长度比pattern小时text必定不包含pattern
    if (lenText < lenPattern) { 
        return -1;
    }

    // 构建移动表
    BadSymbolShiftTable d1(pattern);
    GoodSuffixShiftTable d2(pattern);

    int i = lenPattern - 1;
    while (i < lenText) {
        int k = 0;
        while (k <= lenPattern - 1 && pattern[lenPattern - 1 - k] == text[i - k]) {
            k++;
        }

        if (k == lenPattern) {
            return i - lenPattern + 1;
        }
        else {
            if (k == 0) {
                i += d1.Get(text[i], k);
            }
            else {
                i += Max(d1.Get(text[i], k), d2.Get(k));
            }
        }
    }
    return -1;
}

void Test() {
    GoodSuffixShiftTable table("ABCBAB");
    for (int i=1; i<6; i++) {
        std::cout << table.Get(i) << " ";
    }
    std::cout << "\n-------------------\n";
    std::cout << BoyerMooreMatching("efg", "abcdefg") << std::endl;
}

int main() {
    Test();
    return 0;
}
```

