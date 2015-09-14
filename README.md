# DNS代理 #

简单的DNS代理，用来做内网DNS解析。本程序只实现了功能所需的DNS报文解析/打包。

## 安装 ##

	$ go get github.com/lisijie/dnsagent
	$ cd github.com/lisijie/dnsagent
	$ sudo ./dnsagent --debug --dns=8.8.8.8:53

## DNS协议 ##

DNS基于UDP协议，默认端口号是53。DNS报文格式如下：

	+-------------------------------+
	| 报文头                         |
	+-------------------------------+
	| 问题 (向服务器提出的查询部分)    |
	+-------------------------------+
	| 回答 (服务器回复的资源记录)      |
	+-------------------------------+
	| 授权 (权威的资源记录)           | 
	+-------------------------------+
	| 格外的 (格外的资源记录)         |
	+-------------------------------+

说明：查询包只有报文头头和问题部分，回复包是在查询包的基础上追加了回答、授权、额外资源部分，并且修改了包头的相关标识。

报文头结构如下（共12字节）：

	                                1  1  1  1  1  1
	  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                      ID                       |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                    QDCOUNT                    |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                    ANCOUNT                    |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                    NSCOUNT                    |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                    ARCOUNT                    |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

查询（问题）部分结构：
	
	  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                                               |
	/                     QNAME                     /
	/                                               /
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                     QTYPE                     |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                     QCLASS                    |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+


应答资源结构（包括回答、授权、额外资源都使用同一结构）：

	                                1  1  1  1  1  1
	  0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                                               |
	/                                               /
	/                      NAME                     /
	|                                               |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                      TYPE                     |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                     CLASS                     |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                      TTL                      |
	|                                               |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
	|                   RDLENGTH                    |
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--|
	/                     RDATA                     /
	/                                               /
	+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+


[http://www.rfc-editor.org/rfc/rfc1035.txt](http://www.rfc-editor.org/rfc/rfc1035.txt "http://www.rfc-editor.org/rfc/rfc1035.txt")