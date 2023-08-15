#  Rotate Array

## Description

Given an integer array `nums`, rotate the array to the right by `k` steps, where `k` is non-negative.  Do it in-place with `O(1)` extra space.

## Algorithms

### Move elements one by one

For example, `nums=[1, 2, 3, 4, 5, 6, 7]`, `k=3`. We move `nums[4]` to `nums[0]`, `nums[1]` to `nums[4]`, `nums[5]` to `nums[1]` and so forth.

```c++
// a % b, where the result is always non-negative
int positive_mod(const int & a, const int & b)
{
    int result = a % b;
    if (result < 0)
        result += b;
    return result;
}

// Assume a >= 0 and b >= 0 
int greatest_common_divisor(int a, int b)
{
    if (a > b)
        std::swap(a, b);
    // Now, a <= b
    while (a)
    {
        b = b % a;
        swap(a, b);
    }
    return b;
}

void rotate(vector<int>& nums, int k)
{
    int gcd = greatest_common_divisor(nums.size(), k);
    for (int start_idx = 0; start_idx < gcd; ++start_idx)
    {
        // Move number at next_idx to current_idx;
        int first = nums[start_idx], current_idx = start_idx;
        int next_idx = positive_mod(current_idx - k, nums.size());;
        while (next_idx != start_idx)
        {
            nums[current_idx] = nums[next_idx];
            current_idx = next_idx;
            next_idx = positive_mod(current_idx - k, nums.size());
        }
        nums[current_idx] = first;
    }
}
```

### Reverse the array

As the example above. There are three steps:

- reverse the numbers form index 0 to 4 (`[4321 567]`)
- reverse the  `k-th` elements from the last (`[4321 765]`)
- now reverse the whole `nums` (`[5671234]`)