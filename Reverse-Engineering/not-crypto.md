# not crypto

there's crypto in here but the challenge is not crypto... ð¤

[not-crypto](https://artifacts.picoctf.net/picoMini+by+redpwn/Reverse+Engineering/not-crypto/not-crypto)

## WP

ä¸è½½æä»¶å¹¶æ£æ¥ï¼åç°æ¯ä¸ä¸ªå¯æ§è¡æä»¶ã

![image-20210706223327801](not-crypto.assets/image-20210706223327801.png)

å°è¯æ§è¡ï¼è¾åº**I heard you wanted to bargain for a flag... whatcha got?**ï¼æä¹ä¸æã

![image-20210706223219433](not-crypto.assets/image-20210706223219433.png)

å¯¹ç¨åºè¿è¡åç¼è¯ï¼åç°æåä¼å°ä¸¤ååå­åå®¹è¿è¡æ¯è¾ï¼ä¸ç¨è¯´ä¸åè¯å®æ¯ç¨æ·è¾å¥ç¼å²åºï¼å¦ä¸åå°±æ¯é¢ç½®çFlagã

![image-20210706233856700](not-crypto.assets/image-20210706233856700.png)

ç¶èï¼åé¢é¢ç½®Flagçè¿ç¨è¿äºå¤æï¼æå¨åæå¤ªç´¯äºï¼æï¼ï¼å æ­¤èèä½¿ç¨angrèªå¨ååæã

èªå¨åæèæ¬å¦ä¸ï¼

```python
import angr

sim = angr.Project('./not-crypto').factory.simgr()
sim.explore(find=lambda s: b"Yep, that's it!" in s.posix.dumps(1))
print(sim.found[0].posix.dumps(0).decode())
```

åæåå¾å°Flagã

![image-20210706233809224](not-crypto.assets/image-20210706233809224.png)