# Robots in skyscraper

There are N robots inside skyscraper with F floors.</br>
It's possible to select arbitrary set of robots and send a command.</br>
A command can be to move one floor up or down and it's execution takes 1 minute.</br>
The task is find minimum amount of time to collect all robots on the same floor.</br>

### Input data
The first line contains $N$ - number of robots.</br>
Second line contains $N$ integers - number of the corresponding floor, where the $i_{th}$ robot is located.</br>

### Output data
Minimum amount of time in minutes to collect all robots on the same floor.

### Constraints
1 $\leqslant$ N $\leqslant$ $10^5$ — number of robots</br>
0 $\leqslant$ F $\leqslant$ $10^9$ — number of floors in skyscraper</br>

### Example test cases
```cpp
Input:
2
4 7
Output:
3
```

```cpp
Input:
7
3 1 1 2 10 7 4
Output:
9
```

## Solution
If there is one robot or multiple robots at the same floor, then it's needed 0 minutes to collect all robots.</br>
If robots are spread on multiple floors, then the fastest way is to collect them in the middle floor.</br>
The number of the middle floor is arithmetic mean $M$ for the all given floors.</br>
The minimum amount of time to collect robots on $M$ floor then:</br>
$ans = M - min floor + max floor - M$.

```cpp
int main()
{
    int n;
    cin >> n;

    unsigned long long S = 0;
    int min = INT_MAX;
    int max = INT_MIN;
    for(int i = 0; i < n; ++i)
    {
        int a;
        cin >> a;
        min = std::min(min, a);
        max = std::max(max, a);

        S+=a;
    }

    int mid = S / n;
    if(S%n*2 > n)
    {
        mid++;
    }

    unsigned long long ans = mid - min;
    ans += max - mid;

    cout << ans << endl;

    return 0;
}
```

Solution time complexity: $O(n)$<br>
Solution memory complexity: $O(1)$

## References
* [Elephant trainer on algotester](https://www.algotester.com/en/ArchiveProblem/DisplayWithFile/20036)
