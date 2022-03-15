# 递归时间复杂度求解


# 递归时间复杂度求解

## master公式使用

$$
T(N)=a*T(\frac{N}{b})+O(N^d)\\\
\log(b,a)>d\qquad时间复杂度O(N^{\log(b,a)})\\\
\log(b,a)=d\qquad时间复杂度O(N^d\*\log{N})\\\
\log(b,a)<d\qquad时间复杂度O(N^d)\\\
$$

`N`为母问题的规模级别，子问题规模都是$\frac{N}{b}$，`a`是子问题调用次数，$O(N^d)$是除了调用子问题以外的过程的时间复杂度

{{< admonition success 举个栗子 >}}

```c
int process(int arr[], int L, int R)
{
    if (L == R)
        return arr[L];
    int mid = L + (R - L) >> 1; //求中点
    int LMax = process(arr, L, mid);
    int RMax = process(arr, mid + 1, R);
    return RMax > LMax ? LMax : RMax;
}
```

该递归算法时间复杂度：
$$
T(N)=2*T(\frac{N}{2})+O(1)
$$
{{< /admonition>}}

