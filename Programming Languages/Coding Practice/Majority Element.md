# Majority Element

## Description

Given an array `nums` of size `n`, return *the majority element*.

The majority element is the element that appears more than `n/2` times. You may assume that the majority element always exists in the array.

## MJRTY - A Fast Majority Vote Algorithm

Imagine a convention center filled with delegates (i.e., voters) each carrying a placard proclaiming the name of his candidate. Suppose a floor fight ensues and delegates of different persuasions begin to knock one another down with their placards. Suppose that each delegate who knocks down a member of the opposition is simultaneously knocked down by his opponent. Clearly, should any candidate field more delegates than all the others combined, that candidate would win the floor fight and, when the chaos subsided, the only delegates left standing would be from the majority block. Should no candidate field a clear majority, the outcome is less clear; at the conclusion of the fight, delegates in favor of at most one candidate, say, the nominee, would remain standing--but the nominee might not represent a majority of all the delegates. Thus, in general, if someone remains standing at the end of such a fight, the convention chairman is obliged to count the nominee’s placards (including those held by downed delegates) to determine whether a majority exists. 

Thus our algorithm has two parts. The first part pairs off disagreeing delegates until all remaining delegates agree. We call this the "pairing" phase. Perhaps non-obviously, pairing can be done with n comparisons. If pairing leaves any delegates standing then those delegates unanimously favor a single candidate--the nominee-- who must be in the majority if a majority exists. The second part of the algorithm, called the "counting" phase, determines whether the nominee received more than half the votes. The counting phase obviously requires at most n comparisons. The focus of this paper is on the pairing phase.

Here is a bloodless way the chairman can simulate the pairing phase. He visits each delegate in turn, keeping in mind a current candidate `CAND` and a count `K`, which is initialized to 0. Upon visiting each delegate, the chairman first determines whether `K` is 0; if it is, the chairman selects the delegate’s candidate as the new value of `CAND` and sets `K` to 1. Otherwise, the chairman asks the delegate whether his candidate is `CAND`. If so, then K is incremented by 1. If not, then K is decremented by 1. The chairman then proceeds to the next delegate. When all the delegates have been processed, CAND is in the majority if a majority exists.

## Answer

```c
int majorityElement(int* nums, int numsSize)
{
    int count = 0, candidate = nums[0];
    for (int i = 0; i < numsSize; ++i)
    {
        if (nums[i] == candidate)
            count += 1;
        else if (count > 0)
            --count;
        else
        {
            count = 1;
            candidate = nums[i];
        }
    }
    return candidate;
}
```

