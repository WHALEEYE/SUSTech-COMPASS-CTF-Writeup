# scrambled-bytes

I sent my secret flag over the wires, but the bytes got all mixed up!

[capture.pcapng](https://artifacts.picoctf.net/picoMini+by+redpwn/Forensics/scrambled-bytes/capture.pcapng)

[send.py](https://artifacts.picoctf.net/picoMini+by+redpwn/Forensics/scrambled-bytes/send.py)

## WP

#### 寻找关键的报文

打开`send.py`，可以看见加密发送Flag的原理：将Flag先打乱，然后对Flag中的每个字符用随机数进行异或加密后，通过UDP协议从本地的随机端口发出。

```python
with open(args.input, 'rb') as f:
    payload = bytearray(f.read())
random.seed(int(time()))
random.shuffle(payload)
with IncrementalBar('Sending', max=len(payload)) as bar:
    for b in payload:
        send(
            IP(dst=str(args.destination)) /
            UDP(sport=random.randrange(65536), dport=args.port) /
            Raw(load=bytes([b ^ random.randrange(256)])),
            verbose=False)
        bar.next()
```

因此，我们需要注意的就是抓包记录中的UDP报文。

![image-20210708230319048](scrambled-bytes.assets/image-20210708230319048.png)

将这些UDP报文中长度不为43的报文删去后导出。

![image-20210708230525051](scrambled-bytes.assets/image-20210708230525051.png)

由于在这些过滤过的UDP包中还存在着一些Malformed的包，不知道是否应该保留，且还有两个基于UDP但为其他协议的包（A21和Teredo），不知道是否应该保留，因此导出三个版本的抓包记录：

* `udp_1.pcapng`：含有所有包的记录
* `udp_2.pcapng`：仅含有正常（非Malformed）的包的记录
* `udp_3.pcapng`：仅含有仅为UDP协议的包的记录

由于两个其他协议的包被Wireshark判定为Malformed，因此在`udp_2.pcapng`中没有这两个协议的报文。

#### 寻找随机数种子

由于Flag是被随机打乱且被随机数进行了异或加密后发出的，因此即使我们已经拿到了发送的数据，这样的加密方法似乎依旧无解。

但是在`send.py`中，有一句关键代码：

```python
random.seed(int(time()))
```

在原脚本中生成的随机数其实是被指定了种子的，也就是说我们只要找到这个种子，就能完美复制出一模一样的随机操作。

根据抓包记录中的信息，我们可以推断出执行这段代码的时间应该是在`1614044645`到`1614044655`之间，因此种子应该在这个范围内。

我们可以通过一个信息来判断正确的种子：发送的端口号。

我们用不同的种子和不同的抓包记录组合，模仿`send.py`中的操作，生成一系列端口号，脚本如下：

```python
import random

import pcapng.blocks
from pcapng import *

pcapng_list = ['udp_1.pcapng', 'udp_2.pcapng', 'udp_3.pcapng']
for file in pcapng_list:
    with open(file, 'rb') as pc:
        data_list = []
        data = FileScanner(pc)
        for block in data:
            if block.__class__ == pcapng.blocks.EnhancedPacket:
                data_list.append(block.packet_payload_info[2][-1])
            else:
                continue
        for seeds in range(1614044645, 1614044660):
            random.seed(seeds)
            port_list = []
            init = [i for i in range(0, len(data_list))]
            random.shuffle(init)
            for b in data_list:
                port_list.append(random.randrange(65536))
                new_data.append(b ^ random.randrange(256))
            print(f'>>>seed: {seeds}, file: {file}')
            print(port_list)
```

除了`random.randrange(65536)`以外，`shuffle()`，`random.randrange(256)`这两个函数也不能少，否则会影响到随机数的生成。

![image-20210708232431500](scrambled-bytes.assets/image-20210708232431500.png)

抓包记录中，前几个端口号分别为`45829`，`37277`，`6`，`48676`，`54362`。我们在输出的结果中搜索这几个端口号。

![image-20210708232605235](scrambled-bytes.assets/image-20210708232605235.png)

![image-20210708232615775](scrambled-bytes.assets/image-20210708232615775.png)

最终找到两个可能正确的组合，其中正确的seed确定为`1614044650`，正确的抓包记录可能为`udp_1`或`udp_3`。

#### 恢复数据

接下来，使用脚本将这两份数据分别恢复，脚本如下：

```python
import random

import pcapng.blocks
from pcapng import *

pcapng_list = ['udp_1.pcapng', 'udp_3.pcapng']
for file in pcapng_list:
    with open(file, 'rb') as pc:
        data_list = []
        data = FileScanner(pc)
        for block in data:
            if block.__class__ == pcapng.blocks.EnhancedPacket:
                data_list.append(block.packet_payload_info[2][-1])
            else:
                continue
        random.seed(1614044650)
        port_list = []
        new_data = []
        init = [i for i in range(0, len(data_list))]
        random.shuffle(init)
        for b in data_list:
            port_list.append(random.randrange(65536))
            new_data.append(b ^ random.randrange(256))
        new_str = [0] * len(new_data)
        ptr = 0
        for index in init:
            new_str[index] = new_data[ptr]
            ptr = ptr + 1
        print(f'>>> file: {file}')
        print(''.join([chr(c) for c in new_str]))
```

输出结果如下：

```xml
>>> file: udp_1.pcapng
PNG

Þyºèu¶ý&³ýëf
6é@UJ)¥Î'MÜU6yþöõfæ=éôM:
«ñ£g/ù3{<£Ú°tÉVeJò-F®bäúÆc[6ê¨S2eÌ*ë@u
·ÚÕëúôcQ¥iöÐcÈ*§í´vþq(õ	­:}´æýì½Ç¸G%g¬Ú^'H
ùlgä<.8%à+æN0ÈýìoÊµÏÐz!C%Ùd|¤ÕÌµõMØÛÙ½H³±v¸©ë½ªÖK\ÀKæ(T Qì jzpÛõ
;öbç ¹gÅ|GM¦éè IgË¥­¼Ðí_ÏJ¼Ñlw#¡sdÀ29ëïfI>
gßSÑKpzÏ»ÁÁÖdLr aìz8ê`¡ª[>º:êû8íßt?ù2ð¸×jN¹wàÜ(+ð[`c&À¦²ðFlk»ìi\¬ÍØÌàØ¯ÓUÅ:c@Ëaò¤~°0c!«úÙÝ Ù Mw[a={®õÚô$Âÿ©dñ2¼El¡T0¢§·1¹Ó/ekÚ,+ØúH+È¨oiÝì6¢û´_ÊñLw)L[Üa·V´ëïûH¬K$ü+¡¡Mö@ñ5éé]¹ÛÈj´]%8obõCøÙ¬ÀßÅ´tÕ32	,5uØÉÂÁ¥´à¤ñ®¨Ô9öORý?'îeÓ5½ªæ0ÔÕ> BÕK7°Þ#Z±^íç;kñ4ZK"Ã<kÂ:B³@6bxÐå¢ÏX±WêEÆVâAÑ£¨Ñ^evNóÏ¦Íæt>úÖÜ¯úý8W®HôÄ|é[êßBwaÀ2Ø(LÁ
qtYJ§¹þô;MßW¶Q²òw¥9&±ÒÞ»r¿+Ù3Ç|ÓÙ¢$ÆË8Û;ÐÕE©Ør2Çú-çÓ&Î7ëÖ{wig±nc©c¥5053Ü\.ÝVP>µÛ0ïIn
Ê¾WÓ/ðµÎÝe¼N¨Ç^:db7Åª=ã}~j6,fUvíW¥6/v9ìtlK§ÕK|Õfå»¤¡îCÎîì»À.°ý½»L±g¨â÷    IEND®B`
>>> file: udp_3.pcapng
B»qÐlÍ~ZÆ,ÀS\^Óð¯7 ¯sOÇ? =6é'ú¥£¢¶<UOìõßÆW+ÿ
vÎ
LÑ¬¾ñòzN5ýX(ÓÉBnèÀ÷åï£ñR@E:Î/5>uªÆß>7U§Õ1	&<ÔÆ=¢í«È$¢RCu`ýß÷¶P7ÈCÇéLn©Ãñpì'[
è¸éëù5ÁÑætÇíIàþ#C»a!rÇvéfU%Á«_$EtÏK´ß@b5HÜÅ£¶-l^xÅ¦4»ØLzþúÙ@§P¤K¿XÌ-¸çÌ(rË-Í*R+{qùLm[ö±NÛåÿö ~zà%Uêêén5´T¼G	yÒ
f;>»[e+}çGÓµã0 -Æä;Qôÿ=°,W·Ì
Æ<©¢ÒhÞþ6øÍÁPÏÍÔ¤sûãí®É8[óa®6îÚ.i2Üã×w1g*¦Ñ©ðH9ín"º»µ³x®þîôéP´ÄÅeéïý(¸ú¥¿Z}ÈA"o+;RiJÀXØDYÞ'ogÆnGMíÂ¦MU'´
Cüxh;!Mj&àäBtvÞßÐôÖÊ$ïPYqÏìå8þÎ°B9ÐÞ=ãi$!É¨=\rÜVÑÐJTf·¯É?ÒqÃÇéÉY¨ìw(mße©ø¤Oz¯Ã]Æo½_3ø¶¿ÈhdGê ²3¢e©Ký ìßÉ5û=ò|§qÅÎ]¡·|x;Î¢P~|êðU¸$7õ®HÐQf<µdÝ'×)}¤]°Çjuá
```

可以看到，当采用`udp_1`的抓包记录时，出现了`PNG`的文件头。

将`udp_1.pcapng`的数据复原后输出到PNG图片中，得到的图片中即是Flag。

![image-20210708234111717](scrambled-bytes.assets/image-20210708234111717.png)