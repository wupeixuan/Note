## TCP三次握手
所谓三次握手（Three-Way Handshake）即建立TCP连接，就是指建立一个TCP连接时，需要客户端和服务端总共发送3个包以确认连接的建立。整个流程如下图所示：

![TCP三次握手](http://images.cnblogs.com/cnblogs_com/wupeixuan/1185085/o_2964446-aa923712d5218eeb.png)

1. 第一次握手：Client将标志位SYN置为1，随机产生一个值seq=J，并将该数据包发送给Server，Client进入SYN_SENT状态，等待Server确认。
2. 第二次握手：Server收到数据包后由标志位SYN=1知道Client请求建立连接，Server将标志位SYN和ACK都置为1，ack=J+1，随机产生一个值seq=K，并将该数据包发送给Client以确认连接请求，Server进入SYN_RCVD状态。
3. 第三次握手：Client收到确认后，检查ack是否为J+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=K+1，并将该数据包发送给Server，Server检查ack是否为K+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了。

### 为什么需要三次握手，而不是二次握手？

主要是为了防止两次握手情况下已失效的连接请求报文段突然又传送到服务端,而产生的错误。举例如下:

Client向Server发出TCP连接请求,第一个连接请求报文在网络的某个节点长时间滞留,Client超时后认为报文丢失,于是再重传一次连接请求,Server收到后建立连接。数据传输完毕后双方断开连接。而此时,前一个滞留在网络中的连接请求到达了服务端Server,而Server认为Client又发来连接请求,若采用的是“两次握手”,则这种情况下Server认为传输连接已经建立,并一直等待Client传输数据,而Client此时并无连接请求,因此不予理睬,这样就造成了Server的资源白白浪费了;但此时若是使用“三次握手”,则Server向Client返回确认报文段,由于是一个失效的请求,因此Client不予理睬,建立连接失败。第三次握手的作用:防止已失效的连接请求报文段突然又传送到了服务器。

### 第三次握手失败后怎么处理？

如果此时ACK在网络中丢失，那么Server端该TCP连接的状态为SYN_RECV，并且依次等待3秒、6秒、12秒后重新发送SYN+ACK包，以便Client重新发送ACK包，以便Client重新发送ACK包。
           
Server重发SYN+ACK包的次数，可以通过设置/proc/sys/net/ipv4/tcp_synack_retries修改，默认值为5。
             
如果重发指定次数后，仍然未收到ACK应答，那么一段时间后，Server自动关闭这个连接。但是Client认为这个连接已经建立，如果Client端向Server写数据，Server端将以RST包响应，方能感知到Server的错误。

## TCP四次挥手(Client或Server均可主动发起挥手动作)

所谓四次挥手（Four-Way Wavehand）即终止TCP连接，就是指断开一个TCP连接时，需要客户端和服务端总共发送4个包以确认连接的断开。整个流程如下图所示：

![TCP四次挥手](http://images.cnblogs.com/cnblogs_com/wupeixuan/1185085/o_2964446-2b9562b3a8b72fb2.png)

1. 第一次挥手：Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态。
2. 第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。
3. 第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态。
4. 第四次挥手：Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。

### 为什么连接的时候是三次握手，关闭的时候却是四次握手？

Server在LISTEN状态下，收到建立连接请求的SYN报文后，可以直接把ACK和SYN放在一个报文里发送给Client。而关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。

### 为什么TIME_WAIT状态需要经过2MSL(最大报文段生存时间)才能返回到CLOSE状态？

1. Client的最后一个ACK报文在传输的时候丢失，Server并没有接收到这个报文。这个时候，Server就会超时重传这个FIN消息，然后Client就会重新返回最后一个ACK报文，等待两个时间周期，完成关闭。如果不等待这两个时间周期，Server重传的那条消息就不会收到。Server就因为接收不到Client的信息而无法正常关闭。
2. 防止“已失效的连接请求报文段”出现在本连接中。在发送完最后一个ACK报文段后,再经过2MSL,就可以使本连接持续的时间内所产生的所有报文段,都从网络中消失。这样就可以使下一个新的连接中不会出现这种就得连接请求报文段。 