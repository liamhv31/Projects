## The Basics

### Question 1 - Read the "capture file comments". What is the flag?
The packet capture file comments can be used to add metadata notes about the pcap. It can used to writeup anything noteworthy about the pcap file, including capture purpose, network details, discovered hosts and IPs, etc. You can find the comments by going to **Statistics** &rarr; **Capture File Properties**. The comments are at the bottom.

<img width="575" height="709" alt="image" src="https://github.com/user-attachments/assets/100de862-4cf1-487b-9ecf-d5d41c517a2e" />

Scroll down in the comments to see the flag.

**Answer**: TryHackMe_Wireshark_Demo

### Question 2 - What is the total number of packets?
Every packet capture in Wireshark will tell you the number of packets that are inclulded at the bottom of the page. As you filter and query for the data you want, that number will change to show you the number returned from your query, as well as the percentage of the total pcap it makes up.

It also shows you how many comments there are for the capture!

<img width="364" height="27" alt="image" src="https://github.com/user-attachments/assets/d1c7fedc-c90e-4c7b-a9a6-29e37dffb49a" />

**Answer**: 58620

### Question 3 - What is the SHA256 hash value of the capture file?
You can actually find the hash values of the packet capture in Wireshark itself (which is the purpose of this exercise), though I will also demonstrate two other ways to find the has value (Linux specific). First you can find the hash in the same place you find the comments in **Statistics** &rarr; **Capture File Properties**. The **Details** window shows you both the SHA256 and SHA1 hashes.

<img width="574" height="235" alt="image" src="https://github.com/user-attachments/assets/8f9b8733-f2a9-4ed5-90da-e1e03237e890" />

The next way you can find the has, which applies to any file on Linux, is by right-clicking the file, selecting **Properties**, and then page over to the **Digests** tab. There you will see a list of hash functions. Select the ones you want to see the **Digest** for, and click the **⚙️Hash** button.

<img width="609" height="510" alt="image" src="https://github.com/user-attachments/assets/c9dc77b7-08bc-4137-8cbb-943439188bc2" />

The final method is by going to the terminal and using the `sha256sum` command:
```
ubuntu@ip-10-66-189-142:~/Desktop$ sha256sum Exercise.pcapng 
f446de335565fb0b0ee5e5a3266703c778b2f3dfad7efeaeccb2da5641a6d6eb  Exercise.pcapng
```

The PowerShell equivalent to this on Windows would be `Get-FileHash .\Exercise.pcapng`.

**Answer**: f446de335565fb0b0ee5e5a3266703c778b2f3dfad7efeaeccb2da5641a6d6eb

### Question 4 - View packet number 38. Which markup language is used under the HTTP protocol?
You can filter by packet number with the following query: `frame.number == 38`. You can also filter by packet numbers in the following ways:
- Return multiple packets: `frame.number in {1 2 3}`
- Return a range of packets: `frame.number >= 1 && frame.number <= 10`
- Return everything except a specific packet(s): `frame.number != 2`

Once the packet is returned, select it, then there should be a portion of the packet that says **eXtensible Markup Language**, or more commonly referred to - **XML**. This is an HTTP packet that contains XML body/content. You may also notice this in the packet details pane: `[14 Reassembled TCP Segments (18364 bytes)]`. That suggests that the XML content was likely too large to fit into a single TCP packet, so it had to be reassmebled.

<img width="954" height="793" alt="image" src="https://github.com/user-attachments/assets/6d4c353e-5aed-4964-8d80-5c200739d55d" />

**Answer**: eXtensible Markup Language

### Question 5 - What is the arrival date of the packet?
The arrival date and time is stored in the frame metadata of the packet. Look for the `Arrival Time` field. This represents when the packet was captured by the device running Wireshark. There are a few other date/time related fields within the frame metadata:
- `[Time shift for this packet: 0.000000000 seconds]` - This will show if a manual timestamp adjustment has been appied to the packet. Wireshark will actually let you shift packet timestamps forward or backwords. This can be helpful for these like synchronizing multiple captures, correcting clock drift, or compensating for incorrect system times
- `Epoch Time: 1084443432.158193000 seconds` - Wireshark also shows the time in **Epoch**, which is a way of tracking time as a single, continuous number (seconds) since January 1, 1970, at 00:00:00.
- `[Time delta from previous captured frame: 0.070101000 seconds]` - This is the time passed since the previous packet (number 37) was captured
- `[Time delta from previous displayed frame: 0.000000000 seconds]` - This is the time difference between the currently selected packet, and the last visible or displayed packet shown after applying a filter
- `[Time since reference or first frame: 4.846969000 seconds]` - This is the time passed since the first packet in the capture (by default)

<img width="953" height="849" alt="image" src="https://github.com/user-attachments/assets/2b8feb55-ba3b-4978-9106-bcdc9e3dec55" />

**Answer**: 05/13/2004

### Question 6 - What is the TTL value?
**TTL** (Time-to-Live), is a limit on how long a packet can traverse the network before being discarded. TTL is an IP header field, and is measured in "hops". A hop is when one device forwards the packet to another device, that is one hop. To find the TTL of a packet in Wireshark, simply expand the **Internet Protocol Version 4** (IPv4) tree header in the packet details pane. You will see the "Time to live" field.

<img width="1916" height="783" alt="image" src="https://github.com/user-attachments/assets/7533afc3-d595-4e93-80bf-d58d7878e3dd" />

So this packet can be forwarded to 47 devices before it is discarded. This prevents packets from traversing the network indefinitely.

**Answer**: 47

### Question 7 - What is the TCP payload size?
**TCP (Transmission Control Protocol)** is part of **Transport Layer** (Layer 4) of the **OSI model**, so the payload size will be under a different part of the packet. Expand the "Transmission Control Protocol" tree header in the packet details pane, and scroll until you see the "TCP payload" field.

<img width="1918" height="780" alt="image" src="https://github.com/user-attachments/assets/64588be8-e225-4445-93e4-3467ee154662" />

TCP has two main parts - the header and the payload. The header will contain networking information and metadata, while the payload is the actual application data being transmitted.

**Answer**: 424

### Question 8 - What is the e-tag value?
An **ETag** (Entity Tag) is an HTTP response header that identifies a specific version of a resource. Think of it like a digitial fingerprint, version number, or hash. You will find the ETag under the "Hypertext Transfer Protocol" tree.

<img width="1917" height="784" alt="image" src="https://github.com/user-attachments/assets/7e92192f-bf61-4a6c-9e2f-66307de68350" />

**Answer**: 9a01a-4696-7e354b00

### Question 9 - Search the "r4w" string in packet details. What is the name of artist 1?
Wireshark allows you to search packet lists, packet details, and packet bytes for a specific value. Depending on what you're looking for, you can search using a display filter, hex value, string, or regular expression. To do this, you go to **Edit** &rarr; **Find Packet**. For this question, make sure the search criteria is set to **Packet details** and **String**. This will find the specified string in every packet it appears in.

<img width="1919" height="845" alt="image" src="https://github.com/user-attachments/assets/f0f9a55f-1d8f-428d-838c-31b49359b65f" />

We can see the answer embedded in the HTML data (which contains the "r4w" string)

**Answer**: r4w8173

### Question 10 - Go to packet 12 and read the packet comments. What is the answer?
Once you find packet 12, right-click on it and select **Packet comment...** You will see the following comment at the top:
```
This_is_Not_a_Flag_This_is_Not_a_Flag_This_is_Not_a_Flag_This_is_Not_a_Flag_This_is_Not_a_Flag_This_is_Not_a_Flag
```
It seems pretty clear that this is not what we're looking for. If you scroll down in the comments, you'll find:
```
Go to packet number 39765
Look at the "packet details pane". Right-click on the JPEG section and "Export packet bytes". This is an alternative way of extracting data from a capture file. What is the MD5 hash value of extracted image?
```
If we do this, it will extract the JPEG bytes from the capture file and we'll actually be able to open it up on our desktop.

<img width="239" height="152" alt="image" src="https://github.com/user-attachments/assets/95bbada2-24ed-4847-b0b3-dce92f898fc5" />

**Beware when doing this though** - If you extract something like a PDF or Word doc that contains malicious macros for example, you may accidentally infect your machine with malware. Always execute suspicious, unknown files in a sandbox environment.

Once the JPEG is saved, use one of the hash identification methods used preivously in this lab to find the MD5 hash.

**Answer**: 911cd574a42865a956ccde2d04495ebf

### Question 11 - There is a ".txt" file inside the capture file. Find the file and read it; what is the alien's name?
There are a couple of other ways to extract files in Wireshark. The first method was demonstrated in the previous question. The export objects method is arguabley easier. First, we need to find out which protocol was used to transfer the txt file. Go to **Edit** &rarr; **🔍Find Packet** and set the search parameters to **Packet details** and **String**. Now search `.txt`. This should show you which packets contain a text file. We can see they're all HTTP.

Now go to **File** &rarr; **Export Objects** &rarr; **HTTP**. This will show you all transferred objects (including files) using HTTP. Search for `.txt` in the **Text Filter** box and you'll see the text file.

<img width="560" height="400" alt="image" src="https://github.com/user-attachments/assets/a56aa12e-205a-4585-93c7-82ff040b1ae0" />

You can click on it and save it. Now just `cd` to it in terminal and run `cat note.txt` (in Linux). This is what you'll see:
```
.     .       .  .   . .   .   . .    +  .:..+.. ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  .     .  :     .    .. :. .___---------___.::.. ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
       .  .   .    .  :.:. _".^ .^ ^.  '.. :"-_.:.::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
    .  :       .  .  .:../:            . .^  :.:\.::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
        .   . :: +. :.:/: .   .    .        . . .:\::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 .  :    .     . _ :::/:               .  ^ .  . .:\::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  .. . .   . - : :.:./.                        .  .:\::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  .      .     . :..|:                    .  .  ^. .:|:: ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
    .       . : : ..||        .                . . !:|::... :..:.... . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  .     . . . ::. ::\(                           . :)/:: ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 .   .     : . : .:.|. ######              .#######::|::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  :.. .  :-  : .:  ::|.#######           ..########:|::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 .  .  .  ..  .  .. :\ ########          :######## :/:: ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  .        .+ :: : -.:\ ########       . ########.:/::... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
    .  .+   . . . . :.:\. #######       #######..:/::     .+ :: : -. . ... :..:.. . ... :..:.. . ... :..:..
      :: . . . . ::.:..:.\           .   .   ..:/::  .+ :: : -.... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
   .   .   .  .. :  -::::.\.       | |     . .:/:: -.   . . . .... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
      .  :  .  .  .-:.":.::.\             ..:/:: -.   . . . .... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 .      -.   . . . .: .:::.:.\.           .:/:.   . . .   .  .  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
.   .   .  :      : ....::_:..:\   ___.  :/::  . . .   .  .  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
   .   .  .   .:. .. .  .: :.:.:\       :/::.   . . .   .  .  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
     +   .   .   : . ::. :.:. .:.|\  .:/|::.   . . .   .  .  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
     .         +   .  .  ...:: ..|  --.:|::.  . . .   .  .  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
.      . . .   .  .  . ... :..:.."(  ..)":: .   . . .   .  .  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 .   .       .      :  .   .: ::/  .  .::\::. . . + : . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
██████╗  █████╗  ██████╗██╗  ██╗███████╗████████╗███╗   ███╗ █████╗ ███████╗████████╗███████╗██████╗  ::.+ : . ... :..:..
██╔══██╗██╔══██╗██╔════╝██║ ██╔╝██╔════╝╚══██╔══╝████╗ ████║██╔══██╗██╔════╝╚══██╔══╝██╔════╝██╔══██╗ ..: + . . ... :..:..
██████╔╝███████║██║     █████╔╝ █████╗     ██║   ██╔████╔██║███████║███████╗   ██║   █████╗  ██████╔╝..:...+.. . ... :..:..
██╔═══╝ ██╔══██║██║     ██╔═██╗ ██╔══╝     ██║   ██║╚██╔╝██║██╔══██║╚════██║   ██║   ██╔══╝  ██╔══██╗.:+.+...: . ... :..:..
██║     ██║  ██║╚██████╗██║  ██╗███████╗   ██║   ██║ ╚═╝ ██║██║  ██║███████║   ██║   ███████╗██║  ██║...::+.+.. .. : .. : 
╚═╝     ╚═╝  ╚═╝ ╚═════╝╚═╝  ╚═╝╚══════╝   ╚═╝   ╚═╝     ╚═╝╚═╝  ╚═╝╚══════╝   ╚═╝   ╚══════╝╚═╝  ╚═╝.+...+.+. . ... :..:..
 . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
 . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
  . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
   . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
    . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..
     . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:.. . ... :..:..

```
The next method is **follow TCP stream**. You can have Wireshark reconstruct the whole TCP conversation for that packet. You will see the plain-text data displayed right in the window. Note, you may need to change the encoding.

By default it is in ASCII:

<img width="1912" height="835" alt="image" src="https://github.com/user-attachments/assets/c04598b7-54d2-4883-9e27-2c53e09c67b5" />

But if you change it to UTF-8:

<img width="1908" height="835" alt="image" src="https://github.com/user-attachments/assets/cac045a8-89ec-43dd-bc81-10f7dbaa3973" />

**Answer**: PACKETMASTER

### Question 12 - Look at the expert info section. What is the number of warnings?
The **Expert Information** page shows packet behaviors and states that may indicate:
- Errors
- Warnings
- Anomalies

From there, you can derive things like performance issues, suspicious behaviors, and malformed traffic. You can open this pane by either going to **Analyze** &rarr; **Expert Information**, or clicking on the colored circle in the bottom left of Wireshark.

<img width="1140" height="851" alt="image" src="https://github.com/user-attachments/assets/c2b5fd07-1f19-40b7-85ea-f076f72a796b" />

We can see each severity type, it's summary, group, protocol, and the number of each. The warnings we see are for **illegal characters found in header name** for HTTP protocol. This means that an HTTP request or response header contains one or multiple characters that are forbidden by the HTTP specifications. This can be things like spaces, quotes, tabs, etc. Let's look at one of the packets as an example. Packet 643 invokes this warning.
```
Frame 643: 9015 bytes on wire (72120 bits), 9015 bytes captured (72120 bits) on interface ens5, id 1
Ethernet II, Src: 02:45:a3:b1:8c:f1 (02:45:a3:b1:8c:f1), Dst: 02:c8:85:b5:5a:aa (02:c8:85:b5:5a:aa)
Internet Protocol Version 4, Src: 10.10.57.178, Dst: 10.100.1.33
Transmission Control Protocol, Src Port: 80, Dst Port: 48924, Seq: 244755, Ack: 2749, Len: 8949
Hypertext Transfer Protocol
     [truncated]4�k�\004�gj��=�-s\022\024I�`�)�\003u\000�q�J��\002\033�\037L�g�}CN���\003���\005v���G\037�5)-\023\002\035KÚ&�:����}�\026K�T��s�\002��x�3�t��\
        [Expert Info (Warning/Undecoded): Trailing stray characters]
            [Trailing stray characters]
            [Severity level: Warning]
            [Group: Undecoded]
        [Expert Info (Warning/Protocol): Illegal characters found in header name]
            [Illegal characters found in header name]
            [Severity level: Warning]
            [Group: Protocol]
    File Data: 8833 bytes
    Data (8833 bytes)
```
As you can see, it looks like this HTTP packet contains a large amount of garbage/binary data. If we reconstruct the HTTP stream like we did for the TCP stream previously, we can see this (small snippet):
```
4.k...gj...=.-s..I.`.)..u..q.J......L.g.}CN........v...G..5)-...K..&.:....}....K.T..s.....x..3.t.....iV.h...,....[$
#..0;@.w.s.3G4............A....<R.ea.A......;
6.J..>.eog.s.[...}p........M...>u...kA........_..`....... ....acl%...A...6l..}...z...s=...@2hb..H.#Ib.J<n.....A...Bv...?..sO......]..g)46Q......V.M.6.[.G.!..Q.N...A..X.H.V.O.K.V...3Y...w.m.\...s...z..@8.{R....}._.jwou.h:e...4..q..... ..R.8.&.K>.....es...ko.&	-..,t..`~......=..c.e.;.+{.^?q4J..O..RRi.0.
..................q..m.._H.d......[O
........6c..zQ.+._P ..3..7?i..M:.}.|.kH.l..*..S.....`....ym...K..3..S+c.b:..2j[op+...0.m....%..u....../.....k_@	.=&.RMF}..[.....C".......9%d...H..q.`...%..................c.y.y$.z...\...."......-.v..O..\..V...
...1.0v..
.a.N.8.....@;.p	..q.U..Z...5..Wk...G.id&.X.e.a.3.2......rk.QWq.K..#.k.g.j.WZ..5.Z.X.Y.....JR;f.8..0....>`..1.........Bx.P.5...z...i.%....[E.G....../...x... ......:....
.mCW.....'.......2s.....9..X.n..P9...@......	.no......#....D........r@....;okv....\...(..N....$....R@v.O)$gk.``...ZZ[.K...f...........W..V.i.....F,.' ....U...N	.....q..2q+[.....P.....c.T.....QB......a.g.muMNmE..+)a...1!..(.z..I5.KrE.m.s.^...
.\i.v....{..Bl...]...;C `xPx.'&..U.t.......3..p...}..=]..}Z..io..X6wy......Q. .5.>V.......&...i..Y...a.A.....M3..*.J..=q..4.........j7.^.i{>.p.5....b.7.e.D..J.....d.J.D.k..0:...\.o.Y..<.O.....N.|..r.s...+....5R.....}...R&.q...cd..eU...2.......V..9......a./....V.!4......
98..Z......:..g...}.x.IV...O|..P...9..b..I....J.#....V.^.....q.i...w...K.u......x.d.O.0.IC..^.+KK/.....0..j?.J..j..mbK...;..kG.v..#.........._...E.c.....^$..e....".0.aT...j....Rh..$.....0:j.g9k...[.w[.......i.".!..r.n....X...
..m~.5e..]r.S7wH.D..u......\r....5....gj.....nnV......^Gi*....
.RA.#*T...F........b....F........K1.B..yx..0:.8.B.+....y....:.F...^$....yk#.p.N.......O~i).\..gX...m.....x.Y....<n.......A.JQ........N.m.....R(nn..	X....
.i..@............qKss6..\.Mj..y,.$..../j.. ..N@...K+...S..?.....M...a.^./.~...9..q.O6...
```
This is an example of what it should/could look like:
```
GET /download.html HTTP/1.1
Host: www.ethereal.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.6) Gecko/20040113
Accept: text/xml,application/xml,application/xhtml+xml,text/html;q=0.9,text/plain;q=0.8,image/png,image/jpeg,image/gif;q=0.2,*/*;q=0.1
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://www.ethereal.com/development.html

HTTP/1.1 200 OK
Date: Thu, 13 May 2004 10:17:12 GMT
Server: Apache
Last-Modified: Tue, 20 Apr 2004 13:17:00 GMT
ETag: "9a01a-4696-7e354b00"
Accept-Ranges: bytes
Content-Length: 18070
Keep-Alive: timeout=15, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=ISO-8859-1

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html
...
```

**Answer**: 1636

### Question 13 - Go to packet number 4. Right-click on the "Hypertext Transfer Protocol" and apply it as a filter. Now, look at the filter pane. What is the filter query?
This is a simple demonstration of how you can quickly filter on specific things without needing to know the exact syntax. You can right-click on just about anything and apply it as a filter. This example yields the following filter: `http`. This will return all http packets in the packet capture. Another example - if you want to filter on a certain hostname you saw in an HTTP packet, you can right-click on it and add as a filter to see if this HTTP host has appeared in any other packet.

<img width="1909" height="845" alt="image" src="https://github.com/user-attachments/assets/36dac2fb-2d82-47fb-ba93-09f14ff3c2a8" />

Note: When you apply as a filter, it will replace the filter that is already there, not append.

**Answer**: http

### Question 14 - What is the number of displayed packets?
Simply look at the number of packets returned from this filter at the bottom of the Wireshark pane.

**Answer**: 1089

### Question 15 - Go to packet number 33790, follow the HTTP stream, and look carefully at the responses. Looking at the web server's response, what is the total number of artists?
We'll start by jumping to the packet we need with: `frame.number==33790`. Next we can right-click on it, go to **Follow** &rarr; **HTTP Stream**. You can also just select it then hit **Ctrl+Alt+Shift+H**. You'll notice that some of the text is red and some is blue. Red text is data sent by the client and blue text is data returned by the server. It also says at the bottom of the HTTP stream pane that there are 6 client packets and 6 server packets sent in this particular conversation.

<img width="669" height="567" alt="image" src="https://github.com/user-attachments/assets/cdc7afd1-0330-4e9f-bdb3-eaa9bbaadbe0" />

Since we need to find the number of a particular entity, and we know the entity name, we can search for it as a string search in the **Find** box below. This will require some manual scanning. Keep reading the response data surrounding the **artist** keyword until you find details on the number of artists. Use the **Find Next** button to page through. You will eventually find a list of artists.

<img width="850" height="599" alt="image" src="https://github.com/user-attachments/assets/a93329d9-f71c-488c-affe-624dd945efc0" />

**Answer**: 3

### Question 16 - What is the name of the second artist?
Now that we know how artists are identified, we can look for more information on the second artist directly by keyword searching `artist=2`. Keep paging through with the **Find Next** button until you find more information on artist 2. Eventually you will see an HTML block that has this: `<p>painted by: <a href='artists.php?artist=2'>Blad3</a></p>`.

**Answer**: Blad3

## Packet Operations

### Question 1 - Investigate the resolved addresses. What is the IP address of the hostname starts with "bbc"?
The **Resolved Addresses** feature shows you which IP addresses Wireshark was able to resolve to a human-readable name. This can provide more context for the conversation. You can open this view by going to **Statistics** &rarr; **Resolved Addresses**. You can search for an entry by using a minimum of three characters. You can search across all entries or choose hosts, ethernet addresses, ethernet manufacturers, or ethernet well-known addresses. 

<img width="1162" height="727" alt="image" src="https://github.com/user-attachments/assets/7dd56343-c458-48da-99a8-e60ec4e667e0" />

If you do not see resolved addresses, then that means you likely either have the required settings disabled and/or no DNS traffic in your packet capture. Typically, you need to have **Resolve network (IP) addresses** enabled under **Edit** &rarr; **Preferences** &rarr; **Name Resolution**. This will let Wireshark translate certain IPs into human-readable names. I say "tyically" because even if you have this option disabled, Wireshark can still display hostnames that it learned from DNS, mDNS, LLMNR, or NetBIOS name-resolution traffic contained within the capture.

You can also select the **Use an external network name resolver** option under the same setting page which will tell Wireshark to use your DNS to resolve addresses. This will cause the machine running Wireshark to perform DNS lookups for addresses in the capture, generating additional network traffic.

There is also the **Resolve Physical Addresses** option, which allows Wireshark to use MAC addresses to display vendor/manufacturer names instead of raw MAC addresses.

**Answer**: 199.232.24.81

### Question 2 - What is the number of IPv4 conversations?
The **Conversations** feature shows you all of the traffic (packets) sent between two specific endpoints. Instead of looking at individual packets, it aggregates the communication and shows statistics about it. You can view this by going to **Statistics** &rarr; **Conversations**. Conversations are grouped into multiple categories:
- Ethernet
- IPv4
- IPv6
- TCP
- UDP

You can see the number of conversations for each group on the tab itself. E.g., there are 435 seperate IPv4 conversations.

<img width="1114" height="641" alt="image" src="https://github.com/user-attachments/assets/af7f9275-c995-4ce2-8249-5c16d1c4cd33" />

Here is some of the information shown within the **IPv4** tab:
- **Address A**: One endpoint involved in the conversation
- **Address B**: The other endpoint involved in the conversation
- **Packets**: The total number of packets sent between the endpoints
- **Bytes**: The total number of bytes sent between the endpoints
- **Packets A &rarr; B**: The number of packets sent from endpoint A to endpoint B
- **Bytes A &rarr; B**: The number of bytes sent from endpoint A to endpoint B
- **Packets B &rarr; A**: The number of packets sent from endpoint B to endpoint A
- **Bytes B &rarr; A**: The number of bytes sent from endpoint B to endpoint A

**Answer**: 435

### Question 3 - How many bytes (k) were transferred from the "Micro-St" MAC address?
The **Statistics** window will show you the conversations between two endpoints and even allow you to resolve the MAC address for ethernet conversations to see which hostnames are talking to each other. This will show you a conversation involving the "Micro-St" MAC address. However, this will only show you the data sent between the two endpoints for that specific conversation. We want to know how many total bytes were transferred from that MAC address.

We can find this by switching to the **Statistics** &rarr; **Endpoints** window. Select the **Ethernet** tab and check the **Name resolution** box at the bottom. Now we can see how many bytes were transferred from each MAC address. I should note that the accepted answer is 7474, however, the wording of the question implies "how many bytes were sent from the Micro-St MAC address". So, either the author coded the incorrect answer by mistake, or the question is poorly written. 7474 represents the total number of bytes transferred from and recieved to this endpoint. The actual number of transferred bytes (Tx Bytes) is 1083. So technically, the correct answer is 1083 bytes, but for the sake of completing this lab, the answer is 7474 bytes.

<img width="955" height="627" alt="image" src="https://github.com/user-attachments/assets/1e60bd00-4136-4664-aa0d-7e08a1ae1d81" />

**Ansswer**: 7474

### Question 4 - What is the number of IP addresses linked with "Kansas City"?
We can use the **Endpoints** window to get this answer too. Select the **Map** button on the bottom of the window, then choose **Open in browser**. Unfortunately, I wasn't able to open it myself in the provided lab machine at the time of this writeup, but you can see IPs grouped by city on a world map. Below is a reference screenshot.

<img width="1637" height="537" alt="image" src="https://github.com/user-attachments/assets/15414c97-cae6-4089-87c9-ce6f695725cf" />

An important note - This feature is not available in Wireshark by default. It requires supplemental geolcation data. Modern Wireshark versions supports the MaxMind GeoIP database, but it requires additional configuration.

**Answer**: 4

### Question 5 - Which IP address is linked with "Blicnet" AS Organisation?
Identifying to which ASN (Autonomous System Number) an IP address belongs to also requires GeoIP lookup. If it's configured, you can see this information in **Endpoints** window. Just select the **IPv4** tab and scroll to the right until you see **AS Organization**.

<img width="952" height="649" alt="image" src="https://github.com/user-attachments/assets/ebec834d-772a-4770-8cd7-4470fe506f54" />

**Answer**: 188.246.82.7

## Traffic Analysis
