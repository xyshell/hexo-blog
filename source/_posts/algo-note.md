---
title: algo-note
date: 2020-08-12 11:10:57
tags: algo
toc: true
---

# Merge Sort

## Explanation:

1. Wikipedia: https://en.wikipedia.org/wiki/Merge_sort
2. Youtube: https://www.youtube.com/watch?time_continue=17&v=KF2j-9iSf4Q&feature=emb_logo

Time complexity: O(nlogn)
Space complexity: O(n)

## Implementation:

### Python

```python

def merge(arr1: list, arr2: list) -> tuple:
    """Merge two arrays

    Args:
        arr1 (list): first array
        arr2 (list): second array

    Returns:
        tuple(int, list): number of swaps, sorted array
    """
    result = []
    count = 0
    i, j = 0, 0
    m, n = len(arr1), len(arr2)
    while i < m and j < n:
        if arr1[i] <= arr2[j]:
            result.append(arr1[i])
            i += 1
        else:
            result.append(arr2[j])
            count += m - i
            j += 1
    result += arr1[i:]
    result += arr2[j:]
    return count, result


def msort(arr: list) -> tuple:
    """Merge Sort Algorithm

    Args:
        arr (list): unsorted array

    Returns:
        tuple(int, list): number of swaps, sorted array
    """
    n = len(arr)
    if n > 1:
        mid = n // 2
        left_swaps, left_result = msort(arr[:mid])
        right_swaps, right_result = msort(arr[mid:])
        merge_swaps, result = merge(left_result, right_result)
        return left_swaps+right_swaps+merge_swaps, result
    return 0, arr

assert msort([1, 2, 5, 6, 3, 7, 4, 8]) == (5, [1, 2, 3, 4, 5, 6, 7, 8])
```

# LCS(longest common subsequence) problem

## Explanation:

1. Wikipedia: https://en.wikipedia.org/wiki/Longest_common_subsequence_problem

complexity: O(n × m)

## Implementation

### Python

```python
def commonChild(s1: str, s2: str) -> int:
    """Find length of common child string

    Args:
        s1 (str): first string
        s2 (str): second string

    Returns:
        int: length of common child string
    """
    m = len(s1)
    n = len(s2)
    mat = [[0 for _ in range(n+1)] for _ in range(m+1)]
    for r in range(1, m+1):
        for c in range(1, n+1):
            i, j = r - 1, c - 1
            if s1[i] == s2[j]:
                mat[r][c] = mat[r-1][c-1] + 1
            else:
                mat[r][c] = max(mat[r-1][c], mat[r][c-1])
    return mat[m][n]

assert commonChild("SHINCHAN", "NOHARAAA") == 3
```
