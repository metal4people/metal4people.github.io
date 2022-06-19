## Problem statement

The score of an array is defined as the product of its sum and its length.

For example, the score of ```[1, 2, 3, 4, 5]``` is ```(1 + 2 + 3 + 4 + 5) * 5 = 75```.</br>
Given a positive integer array nums and an integer $K$, return the number of non-empty $subarrays$ of nums whose score is strictly less than $K$.

A $subarray$ is a contiguous sequence of elements within an array.

### Example test cases

#### Example #1
```cpp
Input: nums = [2,1,4,3,5], k = 10
Output: 6
Explanation:
The 6 subarrays having scores less than 10 are:
- [2] with score 2 * 1 = 2.
- [1] with score 1 * 1 = 1.
- [4] with score 4 * 1 = 4.
- [3] with score 3 * 1 = 3. 
- [5] with score 5 * 1 = 5.
- [2,1] with score (2 + 1) * 2 = 6.
Note that subarrays such as [1,4] and [4,3,5] are not considered because their scores are 10 and 36 respectively, while we need scores strictly less than 10.

```

#### Example #2
```cpp
Input: nums = [1,1,1], k = 5
Output: 5
Explanation:
Every subarray except [1,1,1] has a score less than 5.
[1,1,1] has a score (1 + 1 + 1) * 3 = 9, which is greater than 5.
Thus, there are 5 subarrays having scores less than 5.
```

### Constraints
```cpp
1 <= nums.length <= 105
1 <= nums[i] <= 105
1 <= k <= 1015
```

## Observation
If there is an array with length $L$ and $score < K$, then it produces $S = 1 + 2 + 3 + ... + L$ different subarrays.

## Solution: prefix sum + binary search
1. Create the prefix sum array [6-10];
2. For each $i_{th}$ find the longest subarray with $score < K$ [15-31];
3. Calculate and add $S$ to the result [33].

```cpp linenums="1"
class Solution {
public:
    long long countSubarrays(vector<int>& nums, long long k) 
    {
        const auto N = nums.size();
        vector<long long> prefix(N, 0);
        prefix[0] = nums[0];
        
        for(int i = 1; i < N; ++i)
            prefix[i] += nums[i] + prefix[i-1];
        
        long long res = 0;
        for(int s = 0; s < N; ++s)
        {
            int l = s;
            int r = N;
            
            while(l < r)
            {
                int m = l + (r - l)/2;
                long long sum = prefix[m] - (s > 0 ? prefix[s-1] : 0);
                long long v =  sum * (m - s + 1);

                if(v < k)
                {
                    l = m + 1;
                }
                else{
                    r = m;
                }
            }

            res += l - s;
        }
        
        return res;
    }
};
```

Solution time complexity: $O(n*log(n))$<br>
Solution memory complexity: $O(n)$

## Solution: sliding window
1. On each iteration extend the sliding window to the right and calculate the new sum;
2. If $score$ $\geqslant$ $K$, then shrink window by popping out the element from the left until $score < K$;
3. Calculate and add $S$ to the result.

```cpp linenums="1"
class Solution {
public:
    long long countSubarrays(vector<int>& nums, long long k) 
    {
        long long sum = 0, ans = 0;
        for(int e = 0, s = 0; e < nums.size(); ++e)
        {           
            sum += nums[e];
            while(sum * (e - s + 1) >= k)
            {
                sum -= nums[s++];
            }
            
            ans += e - s + 1;
        }
        
        return ans;
    }
};
```
Solution time complexity: $O(n)$<br>
Solution memory complexity: $O(1)$

## References
* [Count Subarrays With Score Less Than K on leetcode](https://leetcode.com/problems/count-subarrays-with-score-less-than-k/)