---
title: Quick Sort 快速排序
tags: ["sort"]
categories: ["algorithm"]
reward: true
copyright: true
date: 2019-10-11 01:40:08
thumbnail:
---


基于比较的排序问题的最快时间复杂度为 $O(n \log n)$, 而快速排序的平均复杂度为 $O(n \log n)$。



<!--more-->

# 时间复杂度

+ 最佳情况： $O(n)$
+ 最坏情况： $O(n^2)$
+ 平均情况： $O(n \log n)$



# 代码实现

## C++ 实现

```c++
#include <iostream>

void Swap(int* a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp; 
}

void QuickSort(int arr[], int len) {
    if (len < 2) {
        return;
    }
    
    int front = 0;
    int back = len - 1;

    for(; front < back; back--){
        if (arr[front] > arr[back]) {
            Swap(&arr[front], &arr[back]);
            front++;
            for(; front < back; front++) {
                if (arr[front] > arr[back]) {
                    Swap(&arr[front], &arr[back]);
                    break;
                }
            }
        }
    }
    
    QuickSort(arr, front);
    QuickSort(&arr[front + 1], len-front - 1);
}

void Test(){

    int arr[] = {4, 2, 3, 1, 2, 3, 2};
    
    std::cout << "---before sort---\n";

    for (int i = 0; i < 7; i++) {
        std::cout << arr[i] << std::endl;
    }

    QuickSort(arr, 7);

    std::cout << "---after sort---\n";

    for (int i = 0; i < 7; i++) {
        std::cout << arr[i] << std::endl;
    }
}

int main() {
    Test();
    return 0;
}
```

