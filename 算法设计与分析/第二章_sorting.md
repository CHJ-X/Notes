# 第二章 Sorting

## Insertion sort
  
  最好线性、最差平方

## 算时间复杂度

```C
Max(A):
  Max <- A[0];
  for(i <- 1 to 9>):
    if(A[i] > Max):
      Max <- A[i]
  return Max
```

1. 升序
2. 第一次：1, 7, 6, 9, 3, 10
  第二次：1, 6, 7, 3, 9, 10
  第三次：1, 6, 3, 7, 9, 10
3. 15次
