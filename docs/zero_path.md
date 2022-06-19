## Problem statement

You are given a grid with n rows and m columns. All numbers are equal to 1 or -1.<br>
You start from the cell (1,1) and can move one either one cell down or right at a time.<br>
Is there a zero path from upper left corner [1,1] to down right corner cell [n,m]? <br>

### Zero path example
<br>
![Image](/zero_path.png#center "Grid")

### Constraints
 (1 $\leqslant$ n, m $\leqslant$ 1000)  — the size of the grid.

## Observations
1. Zero sum is only possible when path length is even.
2. Path length is equal to $n + m - 1$.
3. It's enough just to count 1 values, in zero path '1' count is equal to $\frac{(n + m - 1)}{2}$.

## Solution
1. Read grid from stdin; [15-26]
2. For each cell fill a bitset, where $i_{th}$ bit stands for a $i$ sum path from \[1,1\] to specific cell;
3. If destination cell [n,m] contains a bitset where $\frac{(n + m - 1)}{2}$ bit is set, then path exists else doesn't.

Separate attention deserves line number 33.
In this line all possible sums from the left cell ```b[i][j - 1]``` and from the up cell ```b[i - 1][j]``` are merged to one bitset. If current cell is 1, then all bits are shifted to left, meaning that each sum in the resulting set increased by 1.

```cpp linenums="1" hl_lines="33"
int a[1001][1001];
bitset<1001> b[1001][1001];

void solve()
{
  int n, m;
  cin >> n >> m;

  if((n + m) % 2 == 0)
  {
    cout << "NO" << endl;
    return;
  }

  for(int i = 1; i <= n; ++i)
  {
    for(int j = 1; j <= m; ++j)
    {
      b[i][j].reset();
      cin >> a[i][j];
      if(a[i][j] < 0)
      {
        a[i][j] = 0;
      }
    }
  }

  b[0][1][0] = 1;
  for(int i = 1; i <= n; ++i)
  {
    for(int j = 1; j <= m; ++j)
    {
      b[i][j] = (b[i][j - 1] | b[i - 1][j]) << a[i][j];
    }
  }

  if(b[n][m][(n + m)/2])
  {
    cout << "YES" << endl;
  }
  else
  {
    cout << "NO" << endl;
  }

  return;
}
```
Solution time complexity: $\frac{O(nm(n+m))}{64}$<br>
Solution memory complexity: $O(nm)$

## References
* [Zero path on codeforces](https://codeforces.com/contest/1695/problem/C)