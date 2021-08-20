# more_calc

maybe u need more cpu

[task.py](https://compass.ctfd.io/files/90649734de010940c3c810f0b2521bf1/task.py?token=eyJ1c2VyX2lkIjoxNCwidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6ODZ9.YR-yIg.WYD6jqB03bFn4jHaya9oPFWNAgg)

## WP

Open the script, we can find that `q` is generated in this way:

1. Calculate the accumulation of `pow(i, p-2, p)` for `i` from `1` to `(p + 1) / 2` (note that `(p + 1) / 2` is not included) and assign `s` with the value.
2. Assign `q` with `s mod p`.

Obviously it is not practical to directly calculate `s`, we need another way to calculate `s`.

We can present `s` in the following way.
$$
s=\sum_{i=1}^{\frac{p-1}{2}}i^{p-2}\bmod p
$$

To continue the deduction, we need to introduce a lemma.

For each of the `i` we have
$$
\begin{equation}
\begin{aligned}
\binom{p}{2i}&=\frac{p(p-1)(p-2)\cdots(p-(2i-1))}{2i!}\\
\frac{2i}{p}\binom{p}{2i}&=\frac{(p-1)(p-2)\cdots(p-(2i-1))}{(2i-1)!}\\
&\equiv\frac{(-1)(-2)\cdots(-(2i-1))}{(2i-1)!}&\pmod p\\
&=\frac{-(2i-1)!}{(2i-1)!}&\pmod p\\
&=-1&\pmod p
\end{aligned}
\end{equation}
$$
So we can replace $-1$ with $\frac{2i}{p}\binom{p}{2i}$ and then we got
$$
s=\sum_{i=1}^{\frac{p-1}{2}}i^{p-2}\bmod p=(-\sum_{i=1}^{\frac{p-1}{2}}-i^{p-2})\bmod p=(-\sum_{i=1}^{\frac{p-1}{2}}(i^{p-2}\cdot\frac{2i}{p}\binom{p}{2i}))\bmod p=(-\frac{2}{p}\sum_{i=1}^{\frac{p-1}{2}}(i^{p-1}\cdot\binom{p}{2i}))\bmod p
$$
According to **Fermat's Little Theorem**, if $p$ is prime and $i$ and $p$ are relatively prime, $i^{p-1}\equiv1\pmod p$, so we got
$$
s=(-\frac{2}{p}\sum_{i=1}^{\frac{p-1}{2}}(i^{p-1}\cdot\binom{p}{2i}))\bmod p=(-\frac{2}{p}\sum_{i=1}^{\frac{p-1}{2}}\binom{p}{2i})\bmod p
$$
$\sum_{i=1}^{\frac{p-1}{2}}\binom{p}{2i}$ represents the count of possible cases to combine `objects of even number` from `p objects`, but $0$ is not included.

The count of possible cases to combine  `objects of even number` and `odd number` from `p objects` is the same, which means
$$
\sum_{i=0}^{\frac{p-1}{2}}\binom{p}{2i}=\sum_{i=0}^{\frac{p-1}{2}}\binom{p}{2i+1}
$$
And we can also know that
$$
\sum_{i=0}^{\frac{p-1}{2}}\binom{p}{2i}+\sum_{i=0}^{\frac{p-1}{2}}\binom{p}{2i+1}=\sum_{i=0}^{p}\binom{p}{i}=2^p
$$
So we got
$$
\sum_{i=1}^{\frac{p-1}{2}}\binom{p}{2i}=\sum_{i=0}^{\frac{p-1}{2}}\binom{p}{2i}-1=\frac{2^p}{2}-1=2^{p-1}-1
$$
Back to the previous equation, we got
$$
s=(-\frac{2}{p}\cdot(2^{p-1}-1))\bmod p=\frac{-(2^p-2)\bmod p^2}{p}=\frac{p^2-(2^p\bmod p^2-2)}{p}=p-\frac{2^p\bmod p^2-2}{p}
$$
By this expression of `s`, we can get the `q` and decrypt the RSA to get the flag.

The script is shown below.

```python
import gmpy2
from Crypto.Util.number import *

e = ...
p = ...
c = ...

s = p - (pow(2, p, p * p) - 2) // p
q = gmpy2.next_prime(s)
n = p * q
phi = (p - 1) * (q - 1)
d = gmpy2.invert(e, phi)
print(long_to_bytes(pow(c, d, n)).decode())
```

