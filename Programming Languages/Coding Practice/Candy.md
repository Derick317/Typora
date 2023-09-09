# Candy

## Description

There are `n` children standing in a line. Each child is assigned a rating value given in the integer array `ratings`.

You are giving candies to these children subjected to the following requirements:

- Each child must have at least one candy.
- Children with a higher rating get more candies than their neighbors.

Return *the minimum number of candies you need to have to distribute the candies to the children*.

## Algorithm 1

We use an array to store the candy of each child and traverse the rating array twice.

```c++
int candy(vector<int>& ratings)
{
    unsigned N = ratings.size();
    vector<int> candies(N, 1);
    for (int i = 1; i < N; ++i)
        if (ratings[i] > ratings[i - 1] && candies[i] <= candies[i - 1])
            candies[i] = candies[i - 1] + 1;
	for (int i = N - 2; i >= 0; --i)
        if (ratings[i] > ratings[i + 1] && candies[i] <= candies[i + 1])
            candies[i] = candies[i + 1] + 1;
    
    int sum = 0;
    for (auto const & i: candies)
        sum += i;
    return sum;
}
```

Why is this algorithm correct? 

After the first traverse, if one child is assigned a higher rating than the child on the left, he or she gets more candies than that child. 

After the second traverse, if one child is assigned a higher rating than the child on the right, he or she gets more candies than that child. Besides, the relationship built in the first traverse preserves.

Complexity: O(n) for both time and space.

## Algorithm 2

Actually, there is an algorithm which takes O(1) space.