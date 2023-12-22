## TCP/IP
Is called so because TCP works over IP.
- TCP/IP model
    ![OSI model](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202308161157451.png)

### TCP & UDP
#### UDP
#### TCP
- 3-handshake connecting:
   ![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202309191110063.png)

- 4-handWave disconnecting:
   ![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202309191111749.png)

## Socket
### 5-Step Client/Server Connection Establishment
1. Socket: `int sockfd = socket(domain, type, protocol)`
2. Bind:
3. Listen
4. Accept
5. Connect

### Socket()
`int sockfd = socket(domain, type, protocol)`  
IP + port, but also with specified protocal your program desired. 

#### Protocol
Rules, procedures and formats that define communication.
![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202308071808141.png)

#### Domain
PF_XXX|AF_XXX, not much difference.
- PF_INET = Protocol Family Internet (IPv4)
- PF_INET6 (IPv4)

#### Socket type:
- TCP: order/deliver assurement.
- UDP: fast.

### Internet Address representation in C
#### struct sockaddr_in
```c
struct sockaddr_in { // IP Address + port
    sa_family_t sin_family; // address family (almost always AF_INET)
    uint16_t sin_port; // port number
    struct in_addr sin_addr; // IPv4 address
    char sin_zero[8]; // not used
};

struct in_addr { // IP Address
    in_addr_t s_addr;
};
```
the struct is used in **binding**

#### Convert IP/Port to network byte ordered (Big Endian)
Depending on CPU, an internet address can be changed to network ordered bytes：  
![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202308111800723.png)  

- functions in C:
    ![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202308111803979.png)

### TCP socket
**TCP server-client function call orders**
![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202308161800623.png)

### UDP socket
**UDP client-server conection**
![](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics@main/Materials/LearningBlogPics202308161759798.png)

# Transport layer
## port
- Q：port是以什么形式或者数据结构在OS中设置的？
A：只是普通的unint16，当OS接收到外部的数据包时，通过解包检查这个unint16然后传送到对应的进程。