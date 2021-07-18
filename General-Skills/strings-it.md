# strings it

Can you find the flag in [file](http://ctf.infury.org:8000/files/61d28ae4f4a051334757b34e2af35813/strings) without running it?

##### Hint

> [string](https://linux.die.net/man/1/strings)

## WP

下载文件，根据提示中Linux的`strings`命令，使用`strings [file_name]`命令查看可执行文件中的可打印字符，并在结果中查找`picoCTF`即可得到Flag。

![image-20210714164813661](strings-it.assets/image-20210714164813661.png)