# Candy sticks

![Image](/candy_sticks.png "Candy sticks"){width=75%}

## Description

There are $M$ kids and $N$ candy sticks.</br>
Kids will be happy only if every kid receives candy stick of the same length.</br>
Candy sticks can be split, but it's not possible to stick them together.</br>
</br>
The task is to find the maximum length of the candy stick and make kids happy.</br>

### Input data
The first line contains $N$ - number of candy sticks, $M$ - number of kids.</br>
Second line contains $N$ integers -  $a_{i}$ length of the $i_{th}$ candy stick.</br>

### Output data
Maximum length of the candy stick with error that doesn't exceed $10^{-4}$.</br>

### Constraints
1 $\leqslant$ N, M $\leqslant$ $1000$,</br>
0 $\leqslant$ $a_{i}$ $\leqslant$ $10^9$</br>

### Example test cases
```cpp
Input:
3 4
1 10 5
Output:
3.333333
```

```cpp
Input:
3 5
1 1 1
Output:
0.500000
```

## Solution
Solution is to run the binary search for the length of the candy stick.<br>
At first, find the boundaries for the binary search lines [6-19].<br>
If the number of sticks is more than number of kids then the shortest stick length is the left boundary else 0.<br>

Search invariant: for each length $m$ check the amount sticks that is possible to obtain by splits [25-28].<br>
If the amount of sticks is more or equal to kids number, then update the left boundary, right otherwise.<br>

```cpp linenums="1" hl_lines="25-28"
int main()
{
    int N, M;
    cin >> N >> M;

    vector<double> v(N, 0);
    double l = INT_MAX;
    double r = INT_MIN;
    for(int i = 0; i < N; ++i)
    {
        cin >> v[i];
        l = std::min(l, v[i]);
        r = std::max(r, v[i]);
    }

    if(M > N)
    {
        l = 0;
    }

    while(r - l > 0.0000001)
    {
        double m = l + (r - l)/2;
        int count = 0;
        for(const auto a : v)
        {
            count += a / m;
        }
        if(count >= M)
        {
            l = m;
        }
        else
        {
            r = m;
        }
    }

    printf("%.7lf", l);

    return 0;
}
```
Solution time complexity: $O(N*log(max(a_{i})))$<br>
Solution memory complexity: $O(N)$

## References
* [0023 on algotester](https://www.algotester.com/en/ArchiveProblem/DisplayWithFile/8)