---
title: 'linux多线程编程'
layout: post
tags: 
  - Linux
  - code
  - c
category: 
  - code
  - c
comments: true
share: true
description: linux多线程编程
---
linux多线程编程

<!--more-->


```c
//客户端

#include <stdio.h>2
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

#define PORT 1234
#define MAXDATASIZE 100
void process(FILE*fp, int sockfd);
char *getMessage(char* sendline,int len, FILE* fp);

int main(int argc,char *argv[])
{
int fd;
struct hostent  *he;
struct sockaddr_in  server;

if (argc !=2) {
printf("Usage:%s &lt;IP Address&gt;\n",argv[0]);
exit(1);
}

if((he=gethostbyname(argv[1]))==NULL){
printf("gethostbyname() error\n");
exit(1);
}
if((fd=socket(AF_INET, SOCK_STREAM, 0))==-1){
printf("socket()error\n");
exit(1);
}

bzero(&amp;server,sizeof(server));
server.sin_family =AF_INET;
server.sin_port=htons(PORT);
server.sin_addr= *((struct in_addr *)he-&gt;h_addr);

if(connect(fd,(struct sockaddr *)&amp;server,sizeof(server))==-1){
printf("connect() error\n");
exit(1);
}

process(stdin,fd);

close(fd);
}

void process(FILE *fp, int  sockfd)
{
char sendline[MAXDATASIZE],recvline[MAXDATASIZE];
int num;

printf("Connected to server. \n");
printf("Input client's name : ");
if (fgets(sendline, MAXDATASIZE, fp) == NULL) {
printf("\nExit.\n");
return;
}
send(sockfd,sendline, strlen(sendline),0);
while(getMessage(sendline, MAXDATASIZE, fp) != NULL) {
send(sockfd,sendline, strlen(sendline),0);

if ((num =recv(sockfd, recvline, MAXDATASIZE,0)) == 0) {
printf("Server terminated.\n");
return;
}

recvline[num]='\0';
printf("Server Message: %s\n",recvline);

}
printf("\nExit.\n");
}

char  *getMessage(char*  sendline,int len, FILE*  fp)
{
printf("Inputstring to server:");
return(fgets(sendline,MAXDATASIZE, fp));
}
```




```c
//服务端

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <pthread.h>

#define PORT 1234
#define BACKLOG 5
#define MAXDATASIZE 1000

void process_cli(int connfd, struct sockaddr_in client);
void *function(void* arg);
struct ARG {
int connfd;
struct sockaddr_in client;
};

main()
{
int listenfd,connfd;
pthread_t  tid;
struct ARG *arg;
struct sockaddr_in server;
struct sockaddr_in client;
socklen_t  len;

if ((listenfd =socket(AF_INET, SOCK_STREAM, 0)) == -1) {
perror("Creatingsocket failed.");
exit(1);
}

int opt =SO_REUSEADDR;
setsockopt(listenfd,SOL_SOCKET, SO_REUSEADDR, &amp;opt, sizeof(opt));

bzero(&amp;server,sizeof(server));
server.sin_family=AF_INET;
server.sin_port=htons(PORT);
server.sin_addr.s_addr= htonl (INADDR_ANY);
if (bind(listenfd,(struct sockaddr *)&amp;server, sizeof(server)) == -1) {
perror("Bind()error.");
exit(1);
}

if(listen(listenfd,BACKLOG)== -1){
perror("listen()error\n");
exit(1);
}

len=sizeof(client);
while(1)
{
if ((connfd =accept(listenfd,(struct sockaddr *)&amp;client,&amp;len))==-1) {
perror("accept() error\n");
exit(1);
}
arg = (struct ARG *)malloc(sizeof(struct ARG));
arg-&gt;connfd =connfd;
memcpy((void*)&amp;arg-&gt;client, &amp;client, sizeof(client));

if(pthread_create(&amp;tid, NULL, function, (void*)arg)) {
perror("Pthread_create() error");
exit(1);
}
}
close(listenfd);
}

void process_cli(int connfd, struct sockaddr_in client)
{
int num;
char recvbuf[MAXDATASIZE], sendbuf[MAXDATASIZE], cli_name[MAXDATASIZE];

printf("Yougot a connection from %s. \n ",inet_ntoa(client.sin_addr) );
num = recv(connfd,cli_name, MAXDATASIZE,0);
if (num == 0) {
close(connfd);
printf("Clientdisconnected.\n");
return;
}
cli_name[num - 1] ='\0';
printf("Client'sname is %s.\n",cli_name);

while (num =recv(connfd, recvbuf, MAXDATASIZE,0)) {
recvbuf[num] ='\0';
printf("Receivedclient( %s ) message: %s",cli_name, recvbuf);
int i;
for (i = 0; i &lt;num - 1; i++) {
if((recvbuf[i]&gt;='a'&amp;&amp;recvbuf[i]&lt;='z')||(recvbuf[i]&gt;='A'&amp;&amp;recvbuf[i]&lt;='Z'))
{
recvbuf[i]=recvbuf[i]+ 3;
if((recvbuf[i]&gt;'Z'&amp;&amp;recvbuf[i]&lt;='Z'+3)||(recvbuf[i]&gt;'z'))
recvbuf[i]=recvbuf[i]- 26;
}
sendbuf[i] =recvbuf[i];
}
sendbuf[num -1] = '\0';
send(connfd,sendbuf,strlen(sendbuf),0);
}
close(connfd);
}

void *function(void* arg)
{
struct ARG *info;
info = (struct ARG*)arg;
process_cli(info-&gt;connfd,info-&gt;client);
free (arg);
pthread_exit(NULL);
}
```