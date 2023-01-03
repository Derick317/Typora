# Bugs in Merge Sort

My code for merge sort:

```c
void merge_sort(int* nums, int* tmp, int size)
{
    if(1 == size)
        return;
    
    int mid = size / 2;
    merge_sort(nums, tmp, mid);
    merge_sort(nums + mid, tmp + mid, size - mid);
    
    // Merge two parts
    for(int i = 0; i < mid; i++)
        tmp[i] = nums[i];
    for(int i = mid; i < size; i++)
        tmp[i] = nums[mid + size - i - 1];
    int left = 0, right = size - 1, idx = 0;
    while (left <= right) // Note: <= is correct
    {
        if(tmp[left] < tmp[right]) // Note: not stable
            nums[idx++] = tmp[left++];
        else
            nums[idx++] = tmp[right--];
    }
}
```

## Bug:

In line 16 `while(left<right)`. It should be `<=`.

## Note:

It is convenient to reverse the right part of `nums` when copying it into `tmp`. However, this is not stable generally, i.e. the order of equivalent elements is not guaranteed to be preserved. To make it stable, we need to modify line 18. It is not enough to change to `tmp[left] <= tmp[right]`, since pointer `left` may point to an element in the right part if the minimum element in the left part is smaller than that in the right part. For example, consider the following sequence:

|  `tmp`  | 2(1) | 4(2) | 5(3) | 6(6) | 6(4) | 5(5) |
| :-----: | :--: | :--: | :--: | :--: | :--: | :--: |
| Pointer |      |      |      |  L   |  R   |      |

`tmp[left] <= tmp[right]` will still choose 6(6) first. So, the correct condition is `left < mid && tmp[left] <= tmp[right]`.