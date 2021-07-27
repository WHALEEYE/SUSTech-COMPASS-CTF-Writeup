# shark on wire 1

We found this [packet capture](http://ctf.infury.org:8000/files/a82afb7359e58c581780935f52322575/capture.pcap). Recover the flag.

##### Hint 1

> Try using a tool like Wireshark

##### Hint 2

> What are streams?

## WP

Open the `.pcapng` file in Wireshark, and then choose `Statistics --> Protocol Hierarchy` to see the details of the protocols in this file.

![image-20210727150030747](shark-on-wire-1.assets/image-20210727150030747.png)

We can get that there are a lot of UDP packets in the record and it takes up nearly 50% of the packets.

Use the filter to filter out the UDP packets.

![image-20210727150456040](shark-on-wire-1.assets/image-20210727150456040.png)

As we can see, there are lot of discrete UDP packets.

We can use `Follow --> UDP Stream` to see the whole stream. In that case, we can get the discrete packets together.

![image-20210727150627020](shark-on-wire-1.assets/image-20210727150627020.png)

Follow the UDP stream from `10.0.0.2` to `10.0.0.13`, we can see a string `picoCTF{N0t_a_fLag}`. Obviously this is not the flag.

![image-20210727150850118](shark-on-wire-1.assets/image-20210727150850118.png)

Try other UDP streams. Finally we can find the flag in the UDP stream from `10.0.0.2` to `10.0.0.12`.

![image-20210727151034227](shark-on-wire-1.assets/image-20210727151034227.png)