# Trapping Rain Water

## Description

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

![img](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)

## Algorithm

### Time O(n), space O(n)

For each bar, let's think about how much water is directly above this bar? The answer is easy, the height of water surface is the smaller of the height of the highest bar on its left and on its right. If water surface is low than the bar, there is no water.

```c++
int trap(vector<int>& height)
{
    vector<int> left_highest(height.size());
    left_highest[0] = 0;
    for (int i = 1; i < height.size(); ++i)
        left_highest[i] = max(height[i - 1], left_highest[i - 1]);
    int water = 0, right_highest = 0;
    for (int i = height.size() - 2; i >= 0; --i)
    {
        right_highest = max(height[i + 1], right_highest);
        water += max(0, min(right_highest, left_highest[i]) - height[i]);
    }
    return water;
}
```

###  Time O(n), space O(1)

The previous algorithm needs a helper array. However, we can remove it. To achieve so, assume first that there is an infinite high bar on the right so that `right_highest` is not necessary. After computing the result, remove the infinite high bar and let surplus water to flow away.

```c++
int trap(vector<int>& height)
{
    int highest_bar_idx = 0;
    int water = 0, highest = 0;
    // Assume there is an infinite high bar on the right
    for (int i = 0; i < height.size(); ++i)
    {
        if (height[i] >= highest)
        {
            highest_bar_idx = i;
            highest = height[i];
        }
        water += highest - height[i];
    }
    //Now remove the infinite high bar
    int right_highest = 0;
    for (int i = height.size() - 1; i > highest_bar_idx; --i)
    {
        if (height[i] > right_highest)
            right_highest = height[i];
        water += right_highest - highest;
    }
    return water;
}
```

