# Best Time to Buy and Sell Stock

## Description

You are given an array `prices` where `prices[i]` is the price of a given stock on the `ith` day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return *the maximum profit you can achieve from this transaction*. If you cannot achieve any profit, return `0`.

## Algorithms

### Divide and Conquer

We can divide the array into two parts. Compute the maximal price, minimal price and maximal profit in each sub-array. Then, we can easily compute these value for the whole array.

The time complexity is O(n). But since we need recursion, the space complexity is O(log n).

```c
typedef struct
{
  int max_price;
  int min_price;
  int max_profit;
} Result;

int max(const int a, const int b)
{
  return a > b ? a : b;
}

int min(const int a, const int b)
{
  return a < b ? a : b;
}

Result maxProfitRecursive(int* prices, int pricesSize)
{
  Result result;
  if (pricesSize == 1)
  {
    result.max_price = result.min_price = prices[0];
    result.max_profit = 0;
    return result;
  }

  int mid = pricesSize / 2;
  Result left = maxProfitRecursive(prices, mid);
  Result right = maxProfitRecursive(prices + mid, pricesSize - mid);
  result.max_price = max(left.max_price, right.max_price);
  result.min_price = min(left.min_price, right.min_price);
  result.max_profit = max(right.max_price - left.min_price, max(left.max_profit, right.max_profit));
  return result;
}

int maxProfit(int* prices, int pricesSize)
{
  return maxProfitRecursive(prices, pricesSize).max_profit;
}
```

### Dynamic Programming

Dynamic programming can achieve constant space complexity.

Let `profit(k)` be the maximal profit one can make if selling the stock at day `k`. Then, `profit(k) = price(k) - min(price(0), price(1), ..., price(k-1))`. And the answer is the maximum of each `profit(k)`. In addition, we can store of the minimal value and maximal value so far to save space.

```c
int maxProfit(int* prices, int pricesSize)
{
    int idx = 1;
    int min_price_from_0_to_idx_minus_1 = prices[0];
    int max_profit_from_0_to_idx_minus_1 = 0;
    for (idx = 1; idx < pricesSize; ++idx)
    {
        int profit_when_sell_at_day_idx = prices[idx] - min_price_from_0_to_idx_minus_1;
        if (profit_when_sell_at_day_idx > max_profit_from_0_to_idx_minus_1)
            max_profit_from_0_to_idx_minus_1 = profit_when_sell_at_day_idx;
        if (prices[idx] < min_price_from_0_to_idx_minus_1)
            min_price_from_0_to_idx_minus_1 = prices[idx];
    }
    return max_profit_from_0_to_idx_minus_1;
}
```

