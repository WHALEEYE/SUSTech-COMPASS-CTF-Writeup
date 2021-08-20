# more_calc

maybe u need more cpu

[task.py](https://compass.ctfd.io/files/90649734de010940c3c810f0b2521bf1/task.py?token=eyJ1c2VyX2lkIjoxNCwidGVhbV9pZCI6bnVsbCwiZmlsZV9pZCI6ODZ9.YR-yIg.WYD6jqB03bFn4jHaya9oPFWNAgg)

## WP

Open the script, we can find that `q` is generated in this way:

1. Calculate the accumulation of `pow(i, p-2, p)` for `i` from `1` to `(p + 1) / 2` (note that `(p + 1) / 2` is not included) and assign `s` with the value.
2. Assign `q` with `s mod p`.

Obviously it is not practical to directly calculate `s`, we need another way to calculate `s`.

We can present `s` in the following way.

![image-20210821012826420](more_calc.assets/image-20210821012826420.png)

To continue the deduction, we need to introduce a lemma.

For each of the `i` we have

![image-20210821012909103](more_calc.assets/image-20210821012909103.png)

So we can replace $-1$ with $\frac{2i}{p}\binom{p}{2i}$ and then we got

![image-20210821012927975](more_calc.assets/image-20210821012927975.png)

According to **Fermat's Little Theorem**, if $p$ is prime and $i$ and $p$ are relatively prime, $i^{p-1}\equiv1\pmod p$, so we got

![image-20210821012950126](more_calc.assets/image-20210821012950126.png)

$\sum_{i=1}^{\frac{p-1}{2}}\binom{p}{2i}$ represents the count of possible cases to combine `objects of even number` from `p objects`, but $0$ is not included.

The count of possible cases to combine  `objects of even number` and `odd number` from `p objects` is the same, which means

![image-20210821013029060](more_calc.assets/image-20210821013029060.png)

And we can also know that

![image-20210821013055572](more_calc.assets/image-20210821013055572.png)

So we got

![image-20210821013136638](more_calc.assets/image-20210821013136638.png)

Back to the previous equation, we got

![image-20210821013149952](more_calc.assets/image-20210821013149952.png)

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

