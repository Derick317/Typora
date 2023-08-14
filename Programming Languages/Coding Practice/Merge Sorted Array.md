# Merge Sorted Array

## Description

You are given two integer arrays `nums1` and `nums2`, sorted in **non-decreasing order**, and two integers `m` and `n`, representing the number of elements in `nums1` and `nums2` respectively.

**Merge** `nums1` and `nums2` into a single array sorted in **non-decreasing order**.

The final sorted array should not be returned by the function, but instead be *stored inside the array* `nums1`.

## Answer

The direct way is to divide the process into 2 stages. In the first stage, neither array is empty. In the second stage, copy the remaining elements.

```c
void merge(int* nums1, int m, int* nums2, int n)
{
    // Merging from the back to the front.
    int idx1 = m - 1, idx2 = n - 1, put = n + m - 1;
    while (idx1 >= 0 && idx2 >= 0)
    {
        if (nums1[idx1] > nums2[idx2])
            nums1[put] = nums1[idx1--];
        else
            nums1[put] = nums2[idx2--];
        --put;
    }
    while (idx1 >= 0)
        nums1[put--] = nums1[idx1--];
    while (idx2 >= 0)
        nums1[put--] = nums2[idx2--];
}
```

However, we can merge the two step into one:

```c
void merge(int* nums1, int m, int* nums2, int n)
{
    // Merging from the back to the front.
    int idx1 = m - 1, idx2 = n - 1, put = n + m - 1;
    while (idx2 >= 0)
        if (idx1 >= 0 && nums1[idx1] > nums2[idx2])
            nums1[put--] = nums1[idx1--];
        else
            nums1[put--] = nums2[idx2--];
}
```

