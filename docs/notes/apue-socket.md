```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
// Returns: file (socket) descriptor if OK, 1 on error

int bind(int sockfd, const struct sockaddr *addr, socklen_t len);
// 端口号不能小于1024
// Returns: 0 if OK, 1 on error

int getsockname(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict alenp);
// Returns: 0 if OK, 1 on error
int getpeername(int sockfd, struct sockaddr *restrict addr, socklen_t *restrict alenp)
// Returns: 0 if OK, 1 on error

int shutdown(int sockfd, int how);
// how: SHUT_RD, SHUT_WR, SHUT_RDWR
// Returns: 0 if OK, 1 on error

#include <arpa/inet.h>
uint32_t htonl(uint32_t hostint32);
// Returns: 32-bit integer in network byte order

uint16_t htons(uint16_t hostint16);
// Returns: 16-bit integer in network byte order

uint32_t ntohl(uint32_t netint32);
// Returns: 32-bit integer in host byte order

uint16_t ntohs(uint16_t netint16);
// Returns: 16-bit integer in host byte order

#include <netinet/in.h>
struct sockaddr {
    sa_family_t sa_family;    // address family
    char        sa_data[14];  // variable-length address
    -
    -
};
struct in_addr {
    in_addr_t s_addr;  // IPv4 address
};
struct sockaddr_in {
    sa_family_t sin_family;  // address family
    in_port_t sin_port;      // port number
    struct in_addr sin_addr;  // IPv4 address
}

#include <arpa/inet.h>
const char *inet_ntop(int domain, const void *restrict addr, char *restrict str, socklen_t size);
// Returns: pointer to address string on success, NULL on error
int inet_pton(int domain, const char *restrict str, void *restrict addr);
// Returns: 1 on success, 0 if the format is invalid, or 1 on error
```



### socket()

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
// Returns: file (socket) descriptor if OK, 1 on error
```



domain决定了socket的地址类型，在通信中必须采用相应的地址，如AF_INET决定了要用IPv4地址（32位的）与端口号（16位的）的组合，AF_UNIX决定了要用一个绝对路径名作为地址。

type: 指定socket类型

protocol为0时，会自动选择type类型对应的默认协议。

### socket communication domains

| Domain    | Description          |
| --------- | -------------------- |
| AF_INET   | IPv4 Internet domain |
| AF_INET6  | IPv6 Internet domain |
| AF_UNIX   | UNIX domain          |
| AF_UNSPEC | unspecified          |



### socket types

| Type           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| SOCK_DGRAM     | fixed-length, connectionless, unreliable messages            |
| SOCK_RAW       |                                                              |
| SOCK_SEQPACKET | fixed-length, sequenced, reliable, connection-oriented messages |
| SOCK_STREAM    | sequenced, reliable, bidirectional, connection-oriented byte streams |



### inet_ntop, inet_pton

inet_ntop: converts a binary address in network byte order into a text string.

inet_pton: converts a text string into a binary address in network byte order.

domain: AF_INET, AF_INET6

size: INET_ADDRSTRLEN is large enough to hold a text string representing an IPv4 address, and INET6_ADDRSTRLEN is large enough to hold a text string representing an IPv6 address.