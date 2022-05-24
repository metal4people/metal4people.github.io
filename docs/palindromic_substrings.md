## Problem statement
Given a string s, return the number of palindromic substrings in it.  
A string is a palindrome when it reads the same backward as forward.  
A substring is a contiguous sequence of characters within the string.  

### Example test cases
```cpp
Input: s = "abc"
Output: 3
Explanation: Three palindromic strings: "a", "b", "c".
```

```cpp
Input: s = "aaa"
Output: 6
Explanation: Six palindromic strings: "a", "a", "a", "aa", "aa", "aaa".
```

### Constraints
```cpp
1 <= s.length <= 1000
s consists of lowercase English letters.
```

## Brute force solution
The idea is to take all possible substrings $[i, j]$ and check whether it's a palindrome:
```cpp
class Solution 
{
public:
    ....

    int countSubstrings(string s) 
    {
        int res = s.size();
        
        for(int len = 2; len <= s.size(); ++len)
        {
            for(int i = 0; i + len - 1 < s.size(); ++i)
            {
                res += isPalindrome(s, i, len);
            }
        }
        
        return res;
    }
};
```
```isPalindrome``` itself can be either recursive or iterative:

* Recursive:
```cpp
    bool isPalindrome(const string & s, int i, int len)
    {
        if(len == 1){
            return true;
        }
        
        const bool firstLastMatch = s[i] == s[i + len -1];
        if(len == 2)
        {
            return firstLastMatch;
        }
                   
        return firstLastMatch && isPalindrome(s, i + 1, len - 2);
    }
```
* Iterative:
```cpp
    bool isPalindrome(const string & s, int i, int len)
    {
        for(int l = i, r = i + len - 1; l < r; ++l, --r)
        {
            if(s[l] != s[r])
            {
                return false;
            }
        }

        return true;
    }
```

Solution time complexity: $O(n^3)$<br>
Solution memory complexity: $O(n)$

## Top down approach (memoization)
The idea is to reuse previous isPalindrome results, so:

$$isPalindrome(s, i, len) = \begin{cases} 
                         true, &\text{for } len=1 \\ 
                         s[i] == s[i+1], &\text{for }\text len=2 \\ 
                         isPalindrome(s, i+1, len - 2),&\text{for }\text len>2 \\ 
                         \end{cases}$$

```cpp
class Solution {
public:
       
    bool isPalindrome(const string &s, int i, int len)
    {
        if(len <= 1)
        {
            return true;
        }
        
        if(len == 2)
        {
            return s[i] == s[i + 1];
        }
        
        const auto key = static_cast<uint64_t>(i) << 32 | len;
        if(auto it = cache.find(key); it != cache.end())
        {
            return it->second;
        }
        
        const bool res = s[i] == s[i + len - 1] && isPalindrome(s, i + 1, len - 2); 
        cache[key] = res;
        
        return res;
    }
    
    int countSubstrings(string s) 
    {
        int res = s.size();
        
        for(int len = 2; len <= s.size(); ++len)
        {
            for(int i = 0; i + len - 1 < s.size(); ++i)
            {
                res += isPalindrome(s, i, len);
            }
        }
        
        return res;
    }
    
    unordered_map<uint64_t, bool> cache;
};
```

Solution time complexity: $O(n^2)$<br>
Solution memory complexity: $O(n^2)$

## Dynamic programming approach
Memoization makes a lot of memory allocations and lookups in the hash table.<br>
All these together can be avoided using dynamic programming (bottom-up approach).

```cpp
class Solution {
public:
       
    int countSubstrings(string s) 
    {
        const auto N = s.size();
        vector<vector<bool>> dp(N, vector<bool>(N + 1, false));

        for(int i = 0; i < N; ++i)
        {
            dp[i][1] = true;
        }

        int ans = N;
        for(int i = 0; i + 1 < N; ++i)
        {
            dp[i][2] = s[i] == s[i + 1];
            ans += dp[i][2];
        }

        for(int len = 3; len <= N; ++len)
        {
            for(int i = 0; i + len - 1 < N; ++i)
            {
                if(s[i] == s[i + len - 1])
                {
                   dp[i][len] = dp[i + 1][len - 2];
                   ans += dp[i][len];
                }
            }
        }
        
        return ans;
    }
};
```

Solution time complexity: $O(n^2)$<br>
Solution memory complexity: $O(n^2)$

## Performance testing results
The same test cases were run on leetcode and below are approximate numbers.

| Implementation                        | Tests execution duration, ms |
| :---                                  | :---:|
| Brute force + isPalindrome (recursive)| 300 |
| Brute force + isPalindrome            | 200 |
| Memoization                           | 500 |
| Dynamic programming                   | 40  |

## References
* [Palindromic substrings problem on leetcode](https://leetcode.com/problems/palindromic-substrings/)