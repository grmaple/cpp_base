## Day4

### TCP的服务

- TCP提供一种面向连接的、可靠的字节流服务。
- 面向连接意味着两个使用TCP的应用（通常是一个客户和一个服务器）在彼此交换数据之前必须先建立一个TCP连接。
- 在一个TCP连接中，仅有两方进行彼此通信。

TCP通过下列方式来提供可靠性：

-  应用数据被分割成TCP认为最适合发送的数据块。
-  当TCP发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。
-  当TCP收到发自TCP连接另一端的数据，它将发送一个确认。这个确认不是立即发送，通常将推迟几分之一秒。
-  TCP将保持它首部和数据的检验和。
-  既然TCP报文段作为IP数据报来传输，而IP数据报的到达可能会失序，因此TCP报文段的到达也可能会失序。如果必要， TCP将对收到的数据进行重新排序，将收到的数据以正确的顺序交给应用层。
- 既然IP数据报会发生重复，TCP的接收端必须丢弃重复的数据。
-  TCP还能提供流量控制。 TCP连接的每一方都有固定大小的缓冲空间。 TCP的接收端只允许另一端发送接收端缓冲区所能接纳的数据。这将防止较快主机致使较慢主机的缓冲区溢出。

##### TCP字节流

两个应用程序通过TCP连接交换8bit字节构成的字节流。TCP不在字节流中插入记录标识符。我们将这称为**字节流服务**（ byte stream service）。如果一方的应用程序先传10字节，又传20字节，再传50字节，连接的另一方将无法了解发方每次发送了多少字节。收方可以分4次接收这80个字节，每次接收20字节。一端将字节流放到TCP连接上，同样的字节流将出现在TCP连接的另一端。

另外，TCP对字节流的内容不作任何解释。 TCP不知道传输的数据字节流是二进制数据，还是ASCII字符、EBCDIC字符或者其他类型数据。对字节流的解释由TCP连接双方的应用层解释。

##### TCP的首部

![image-20200908130313217](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908130313217.png)

![image-20200908130326390](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908130326390.png)

- 每个TCP段都包含源端和目的端的端口号，用于寻找发端和收端应用进程。这两个值加上I P首部中的源端IP地址和目的端IP地址唯一确定一个TCP连接。
- 一个IP地址和一个端口号也称为一个插口(套接字)（socket）。插口对（s o c k e t p a i r）(包含客户IP地址、客户端口号、服务器IP地址和服务器端口号的四元组 )可唯一确定互联网络中每个TCP连接的双方。
- 序号用来标识从TCP发端向TCP收端发送的数据字节流，它表示在这个报文段中的的第一个数据字节。如果将字节流看作在两个应用程序间的单向流动，则 TCP用序号对每个字节进行计数。序号是32bit的无符号数，序号到达2^32-1后又从0开始。SYN标志消耗了一个序号，FIN标志也要占用一个序号。
- 确认序号应当是上次已成功收到数据字节序号加 1。只有ACK标志为1时确认序号字段才有效。
- 发送ACK无需任何代价，因为32 bit的确认序号字段和ACK标志一样，总是TCP首部的一部分。因此，我们看到一旦一个连接建立起来，这个字段总是被设置， ACK标志也总是被设置为1。
- TCP为应用层提供全双工服务。这意味数据能在两个方向上独立地进行传输。因此，连接的每一端必须保持每个方向上的传输数据序号。
- TCP可以表述为一个没有选择确认或否认的滑动窗口协议(现在已经有选择确定了)。我们说TCP缺少选择确认是因为TCP首部中的确认序号表示发方已成功收到字节，但还不包含确认序号所指的字节。当前还无法对数据流中选定的部分进行确认。例如，如果1～1024字节已经成功收到，下一报文段中包含序号从2049～3072的字节，收端并不能确认这个新的报文段。它所能做的就是发回一个确认序号为1025的ACK。它也无法对一个报文段进行否认。例如，如果收到包含1025～2048字节的报文段，但它的检验和错， TCP接收端所能做的就是发回一个确认序号为1025的ACK。
- 首部长度给出首部中 32 bit字的数目。需要这个值是因为任选字段的长度是可变的。这个字段占4 bit，因此TCP最多有60字节的首部。然而，没有任选字段，正常的长度是 2 0字节。
- 在T C P首部中有6个标志比特。它们中的多个可同时被设置为 1。
  - URG 紧急指针（urgent pointer）有效。
  - ACK 确认序号有效。
  - PSH 接收方应该尽快将这个报文段交给应用层。
  - RST 重建连接。
  - SYN 同步序号用来发起一个连接。
  - FIN 发端完成发送任务。
- TCP的流量控制由连接的每一端通过声明的窗口大小来提供。窗口大小为字节数，起始于确认序号字段指明的值，这个值是接收端正期望接收的字节。窗口大小是一个16 bit字段，因而窗口大小最大为65535字节。
- 检验和覆盖了整个的TCP报文段：TCP首部和TCP数据。这是一个强制性的字段，一定是由发端计算和存储，并由收端进行验证。 TCP检验和的计算和UDP检验和的计算相似，使用如11 . 3节所述的一个伪首部。
- 只有当URG标志置1时紧急指针才有效。紧急指针是一个正的偏移量，和序号字段中的值相加表示紧急数据最后一个字节的序号。 TCP的紧急方式是发送端向另一端发送紧急数据的一种方式。
- 最常见的可选字段是最长报文大小，又称为MSS (Maximum Segment Size)。每个连接方通常都在通信的第一个报文段（为建立连接而设置SYN标志的那个段）中指明这个选项。它指明本端所能接收的最大长度的报文段。只是数据部分长度1460，不包含IP头部(20)和TCP头部(20)。
- 到TCP报文段中的数据部分是可选的。可以看到在一个连接建立和一个连接终止时，双方交换的报文段仅有TCP首部。如果一方没有数据要发送，也使用没有任何数据的首部来确认收到的数据。在处理超时的许多情况中，也会发送不带任何数据的报文段。

### TCP连接的建立与终止

TCP是一个面向连接的协议。无论哪一方向另一方发送数据之前，都必须先在双方之间建立一条连接。

这种两端间连接的建立与无连接协议如UDP不同。使用UDP向另一端发送数据报时，无需任何预先的握手。

##### 连接的建立与终止

建立连接：三次握手

断开连接：四次握手

这7个TCP报文段仅包含TCP首部。没有任何数据。

![image-20200908132941912](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908132941912.png)

建立连接协议

- 发送第一个SYN的一端将执行主动打开（ active open）。接收这个SYN并发回下一个SYN的另一端执行被动打开（passive open）
- 当一端为建立连接而发送它的SYN时，它为连接选择一个初始序号。ISN随时间而变化，因此每个连接都将具有不同的ISN。ISN可看作是一个32比特的计数器，每4ms加1。这样选择序号的目的在于防止在网络中被延迟的分组在以后又被传送，而导致某个连接的一方对它作错误的解释。
- 如何进行序号选择?在4.4BSD，系统初始化时初始的发送序号被初始化为1。这个变量每0.5秒增加64000，并每隔9.5小时又回到0（对应这个计数器每8 ms加1，而不是每4 ms加1）。另外，每次建立一个连接后，这个变量将增加64000。

连接终止协议

![image-20200908133835449](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908133835449.png)

- 建立一个连接需要三次握手，而终止一个连接要经过 4次握手。这由TCP的半关闭（half-close）造成的。既然一个TCP连接是全双工，因此每个方向必须单独地进行关闭。这原则就是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向连接。当一端收到一个FIN，它必须通知应用层另一端几经终止了那个方向的数据传送。发送FIN通常是应用层进行关闭的结果。
- 收到一个FIN只意味着在这一方向上没有数据流动。一个 TCP连接在收到一个FIN后仍能发送数据。而这对利用半关闭的应用来说是可能的，尽管在实际应用中只有很少的 TCP应用程序这样做。
- 首先进行关闭的一方（即发送第一个FIN）将执行主动关闭，而另一方（收到这个FIN）执行被动关闭。通常一方完成主动关闭而另一方完成被动关闭。
- 发送FIN将导致应用程序关闭它们的连接，这些FIN的ACK是由TCP软件自动产生的。

##### 连接建立的超时

对第一个SYN的重发机制。

大多数伯克利系统将建立一个新连接的最长时间限制为75秒。

##### 最大报文段长度

- 最大报文段长度（MSS）表示TCP传往另一端的最大块数据的长度。当一个连接建立时，连接的双方都要通告各自的MSS。

- 当建立一个连接时，每一方都有用于通告它期望接收的MSS选项（MSS选项只能出现在SYN报文段中）。如果一方不接收来自另一方的MSS值，则MSS就定为默认值536字节。
- 一般说来，如果没有分段发生，MSS还是越大越好。报文段越大允许每个报文段传送的数据就越多，相对IP和TCP首部有更高的网络利用率。当TCP发送一个SYN时，或者是因为一个本地应用进程想发起一个连接，或者是因为另一端的主机收到了一个连接请求，它能将MSS值设置为外出接口上的MTU长度减去固定的IP首部和TCP首部长度。对于一个以太网， MSS值可达1460字节。
- 如果目的IP地址为“非本地的”，MSS通常的默认值为536。而区分地址是本地还是非本地是简单的，如果目的IP地址的网络号与子网号都和我们的相同，则是本地的；如果目的IP地址的网络号与我们的完全不同，则是非本地的；如果目的IP地址的网络号与我们的相同而子网号与我们的不同，则可能是本地的，也可能是非本地的。

##### TCP的半关闭

TCP提供了连接的一端在结束它的发送后还能接收来自另一端数据的能力。这就是所谓的半关闭。正如我们早些时候提到的只有很少的应用程序使用它。

![image-20200908135336075](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908135336075.png)

没有半关闭，需要其他的一些技术让客户通知服务器, 客户端已经完成了它的数据传送，但仍要接收来自服务器的数据。使用两个T C P连接也可作为一个选择，但使用半关闭的单连接更好。

##### TCP的状态变迁图

![image-20200908135523590](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908135523590.png)

![image-20200908135514317](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908135514317.png)

##### 2MSL等待状态

- TIME_WAIT状态也称为2MSL等待状态。每个具体 TCP实现必须选择一个报文段最大生存时间MSL（Maximum Segment Lifetime）。它是任何报文段被丢弃前在网络内的最长时间。我们知道这个时间是有限的，因为 TCP报文段以IP数据报在网络内传输，而IP数据报则有限制其生存时间的TTL字段。
- RFC 793指出MSL为2分钟。然而，实现中的常用值是30秒，1分钟，或2分钟。需要大于RTO(重传超时时间),这个时间的确定取决于RTT(往返时间)。
- 对一个具体实现所给定的MSL值，处理的原则是：当TCP执行一个主动关闭，并发回最后一个ACK，该连接必须在 TIME_WAIT状态停留的时间为2倍的MSL。这样可让TCP再次发送最后的ACK以防这个ACK丢失（另一端超时并重发最后的FIN）。
- 这种2MSL等待的另一个结果是这个TCP连接在2MSL等待期间，定义这个连接的插口（客户的IP地址和端口号，服务器的IP地址和端口号）不能再被使用。这个连接只能在 2MSL结束后才能再被使用。
- 在连接处于2MSL等待时，任何迟到的报文段将被丢弃。因为处于2MSL等待的、由该插口对(socket pair)定义的连接在这段时间内不能被再用，因此当要建立一个有效的连接时，来自该连接的一个较早替身的迟到报文段作为新连接的一部分不可能不被曲解

##### FIN_WAIT_2 状态

- 在FIN_WAIT _2状态我们已经发出了FIN，并且另一端也已对它进行确认。除非我们在实行半关闭，否则将等待另一端的应用层意识到它已收到一个文件结束符说明，并向我们发一个FIN来关闭另一方向的连接。只有当另一端的进程完成这个关闭，我们这端才会从FIN_WAIT_2状态进入TIME_WAIT状态

- 这意味着我们这端可能永远保持这个状态。另一端也将处于 CLOSE_WAIT状态，并一直保持这个状态直到应用层决定进行关闭。

##### 复位报文段

- RST比特是用于“复位”的。
- 一般说来，无论何时一个报文段发往基准的连接（ referenced connection）出现错误，TCP都会发出一个复位报文段（这里提到的“基准的连接”是指由目的IP地址和目的端口号以及源IP地址和源端口号指明的连接。)
- 收到一个RST置1的数据报，立即把连接断掉。

产生RST的情况：

- 到不存在的端口的连接请求
  - 产生复位的一种常见情况是当连接请求到达时，目的端口没有进程正在听。对于 UDP，当一个数据报到达目的端口时，该端口没在使用，它将产生一个ICMP端口不可达的信息。而TCP则使用复位。
- 异常终止一个连接
  - 终止一个连接的正常方式是一方发送FIN。有时这也称为有序释放
  - 有可能发送一个复位报文段而不是 FIN来中途释放一个连接。有时称这为异常释放
  - 异常终止一个连接对应用程序来说有两个优点：
    - 丢弃任何待发数据并立即发送复位报文段；
    - RST的接收方会区分另一端执行的是异常关闭还是正常关闭。应用程序使用的API必须提供产生异常关闭而不是正常关闭的手段。

- 检测半打开连接
  - 如果一方已经关闭或异常终止连接而另一方却还不知道，我们将这样的TCP连接称为半打开（Half-Open）的。任何一端的主机异常都可能导致发生这种情况。只要不打算在半打开连接上传输数据，仍处于连接状态的一方就不会检测另一方已经出现异常。
  - 如果仍处于连接状态的一方在半打开连接上传输数据，TCP的处理原则是接收方以复位作为应答。

##### 同时打开

两个应用程序同时彼此执行主动打开的情况是可能的，尽管发生的可能性极小。每一方必须发送一个 SYN，且这些SYN必须传递给对方。这需要每一方使用一个对方熟知的端口作为本地端口。这又称为同时打开（ simultaneous open）

![image-20200908143114817](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908143114817.png)

##### 同时关闭

我们在以前讨论过一方（通常但不总是客户方）发送第一个FIN执行主动关闭。双方都执行主动关闭也是可能的，TCP协议也允许这样的同时关闭（ simultaneous close）。

![image-20200908143104528](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908143104528.png)

##### TCP 选项

![image-20200908143143968](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200908143143968.png)

