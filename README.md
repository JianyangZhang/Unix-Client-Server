This project implemented a client and a server in Unix environment.

**Makefile (needs OpenSSL library):**<br/>
make tserver&nbsp;&nbsp;&nbsp;&nbsp;// *create the executable file tserver*<br/>
make tclient&nbsp;&nbsp;&nbsp;&nbsp;// *create the executable file tclient*

**The server creates:**<br/>
a TCP/UDP stream socket in the Internet domain bounding to a port number (specified as a commandline argument), 
receives requests from clients, produces proper results based on the content of requests and responses back to the client 
on the same connection.

**The requests a client can send:**<br/> 
file type request: a request to get the file type of a file on the server<br/> 
file checksum request: a request to get the checksum of a file on the server<br/> 
download file request: a request to download a file from the server<br/> 

**The commandline syntax:**<br/>
tserver [-udp loss_model [-w window] [-r msinterval]] [-d] [-t seconds] port // *starts the server*<br/>
tclient [hostname:]port filetype [-udp] filename // *client sends filetype request*<br/>
tclient [hostname:]port checksum [-udp] [-o offset] [-l length] filename // *client sends checksum request*<br/>
tclient [hostname:]port download [-udp] [-o offset] [-l length] filename [saveasfilename] // *client sends download file request*<br/>

**Commandline syntax illustration:**<br/>
The *loss_model* names a binary file which is read one byte at a time. When a bit is needed to determine 
whether to throw a packet away or not, look at the left-most bit in the byte that has not been examined. 
Once all 8 bits have been used in a byte, read the next byte. 
If the end of the loss model file has been reached, start from the beginning of the file again.

A window-based protocol is used to provide reliability where *window* is the size of the window (which must be ≥ 1) 
and *msinterval* is the timeout interval in milliseconds (which must be ≥ 1 and ≤ 5000). 
If the *-w* commandline option is not specified, the default window size is 3. 
If the *-r* commandline option is not specified, the default timeout interval is 250.

*-d:* debug mode<br/>
*-t seconds:* server auto-shutdown time; must be ≥ 5; default 300 seconds<br/>
*-o offset:* must be >= 0; default 0<br/>
*-l length:* must be >= 0; default 0xffffffff (-1)<br/>

**The port number range:**<br/> 
10000 to 99999

**About the UDP mode:**<br/> 
The server applies buffers that have size 4096 bytes. The number of buffers depends on the window size. 
These memory space will be released when server shutdown. 
Once the client receives a UDP packet, it responses an ACK packet. 
The first four bytes in ACK packet is the same as the first four bytes in UDP packet. 
When the window becomes full or all packets have been sent, the server sleeps for a while then checks if it received ACKs. 
