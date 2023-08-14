# Last Occurrence

## Description

Given a target integer T and an integer array A sorted in ascending order, find the index of the last occurrence of T in A or return -1 if there is no such index.

### Assumptions

- There can be duplicate elements in the array.

### Examples

- A = {1, 2, 3}, T = 2, return 1
- A = {1, 2, 3}, T = 4, return -1
- A = {1, 2, 2, 2, 3}, T = 2, return 3

## Answer

```c++
int lastOccur(vector<int> array, int target)
{
    int left = 0, right = array.size() - 1, mid;
    while (left <= right)
    {
      mid = (left + right) / 2;
      if (array[mid] == target && (mid == array.size() - 1 || array[mid + 1] != target))
        return mid;
      if (array[mid] > target)
        right = mid - 1;
      else 
        left = mid + 1;
    }
    return -1;
}
```

## Note

For check a hit, just add one more check for the next array.

Check whether the possible target is on the left or on the right is a bit tricky. The condition for being on the right is easy. But the one for being on the left is much more complicated. The code above is equivalent to

```c++
if (mid == array.size() - 1 ? array[mid] < target : array[mid + 1] <= target)
    left = mid + 1;
else 
    right = mid - 1;
```

