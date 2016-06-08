title: "Simple Network"
date: 2015-10-06 11:12:27
tags: ADV Programming
---
Author: Song Li
[Download Project](/download/sol315_HW3.zip)
##AIM##
This program is used to transform files from client to server, the server will run the received code and return the answer to the client. The client part and the server part are combined to a single program. Using "fork()" function to make them work separately.

<!--more-->
##USAGE##
Configure file:
The configure file is named "config". The first line of the file is the number of the files that needed to format. The name of each file should be put line by line right after the first line. At the end of the file, input the argument to the "fmt" function. Here is the
###example###
>2
Test
Test2
-w 10 -u

###Arguments###
The IP address and the port number will be the arguments. Users can just input
the IP address and the port number just follow the "./combine". Here is an example:
>./combine 127.0.0.1 9876

##COMPILE##
Type "make" to compile the source code.
##FILE STRUCTURE##
>fmt.c: the main code of this fmt.c
combine.c: the code of this program
makefile: the makefile document to compile the code
test: the first test file
test2: the second test file
config: the config file used by the program
Readme.pdf: this readme document

##RESULT##
This program will have two processes, the files will be read from the client part and be sent to the server part. The server part will then receive these files, store them to the "./s_temp" folder, compile the fmt.c code, run it and return the answer to the client part.

##DESIGN##
1) Connection refused: Since the server and client runs separately, when the client tries to connect the server, the server may not initiated yet. Every time when the client fails to connect the server, it will try to reconnect 3 seconds later again. If the client tries to reconnect 3 times and still not connected, it will give up and exit.
2) Avoid the same file name: As the client and the server are indeed one program, they work in a same directory. In order to avoid the change to this directory, all of the server's work will be done in the "./s_temp/" directory. Files received will be storied in "./s_temp/", and they will be compiled there, they will run there, and they will get the answer there. 

here is the code:

```
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>
#define BUFFER_SIZE 500
#define A_LEN 5
#define FILE_NUM 2// the number of the codes and makefile
int filenum = 0;//the total number of files
char files[100][100] = {"fmt.c", "makefile"};//files that should be transed
int errorout(int id){//error output function
    if(id > 0) 
        perror("ERROR FROM CLIENT");
    else perror("ERROR FROM SERVER");
    switch (id){
        case 1:
            printf("Client socket initiate error\n");
            break;
        case 2:
            printf("Client connect failed\n");
            break;
        case 3:
            printf("File open error\n");
            break;
        case 4:
            printf("Config file is needed\n");
            break;
        case -1:
            printf("Server socket initiate error\n");
            break;
        case -2:
            printf("Server bind error\n");
            break;
        case -3:
            printf("Client connect failed\n");
            break;
    }
    return 0;
}
/**
 * this function is used to output message
 * @param 
 *      server: 0 for client, 1 for server
 *      msg: the message itself
 **/
int outputmsg(int server, char *msg){
    if(server) printf("SERVER: ");
    else printf("CLIENT: ");
    printf("%s",msg);
    return 0;
}

/**
 * this function is used to send message
 * @param
 *      c_sock: the socket which the message will send to
 *      msg: the message
 **/
int sendmessage(int c_sock, char *msg){
    char buffer[BUFFER_SIZE] = {0};
    strcpy(buffer, msg);
    send(c_sock, buffer, BUFFER_SIZE - 2,0);
    return 0;
}

/**
 * this function is used to send files
 * @param
 *      c_sock: the socket which the message will send to
 *      filename: the file name of the file that is needed to be sent
 **/
int sendfile(int c_sock, char *filename){
    char buffer[BUFFER_SIZE] = {0};
    FILE *fp = fopen(filename, "r");
    if(NULL == fp){
        errorout(3);
        printf("%s isn\'t exist\n",filename);
        return -1;
    }
    while(fgets(buffer, BUFFER_SIZE - 2, fp) != NULL){//get the file line by line and send it
        printf("%s",buffer);
        send(c_sock, buffer, BUFFER_SIZE - 2, 0);
    }
    return 0;
}

/**
 * the main function of client part
 * @param
 *      args: the arguments that should be sent to the server
 *      addr: the address information of client
 **/
int client(char * args, struct sockaddr_in addr){
    int sock,i;
    int len;
    char buffer[BUFFER_SIZE];
    sock = socket(PF_INET, SOCK_STREAM, 0);//use ipv4/tcp/ip to transform
    if(sock < 0){//fail to initiate the socket
        errorout(1);
        return -1;
    }
    for(i = 0;i < 3;++ i){//if the client tries to connect the server failed, try three times in total
        if(connect(sock, (struct sockaddr *)&addr, sizeof(struct sockaddr)) < 0){
            errorout(2);
            if(i == 2) {// three times later, the client part will exit
                errorout(2);
                return -2;
            }
            //try to connect to the server 3 seconds later
            outputmsg(0,"Try to reconnect 3 seconds later\n");
            sleep(1);
            outputmsg(0,"Try to reconnect 2 seconds later\n");
            sleep(1);
            outputmsg(0,"Try to reconnect 1 second later\n");
            sleep(1);
        }
        else break;
    }
    outputmsg(0,"Connected to the server\n");
    printf("__START__TRANSMISSION\n__START__FILELIST\n");//output the names of files
    for(i = 0;i < FILE_NUM;++ i) printf("%s\n",files[i]);
    sendmessage(sock, "__START__");//send the file list (config) to the server
    sendmessage(sock,"config");
    sendfile(sock, "config");
    sendmessage(sock, "__END__");
    printf("__END__FILELIST\n");
    for(i = 0;i < FILE_NUM;++ i){//output the names and send messages and files to server        sendmessage(sock, "__START__");//if the cil
        sendmessage(sock, "__START__");//if the client sends the "__START__" message, it means there will be a new file to send
        sendmessage(sock, files[i]);//send the name of the file
        printf("__START__%s\n",files[i]);
        sendfile(sock, files[i]);//send this file
        sendmessage(sock, "__END__");//if the client sends the "__END__", means it's the end of a file
        printf("__END__%s\n",files[i]);
    }
    printf("__START__ARGs\n");
    sendmessage(sock,"__ARGs__");//if the client sends the "__ARGs__", means the next message will be the atgs
    sendmessage(sock,args);
    printf("%s\n",args);
    printf("__END__ARGs\n");
    for(i = FILE_NUM;i < filenum;++ i){//send the files which are needed to format
        sendmessage(sock, "__START__");
        sendmessage(sock, files[i]);
        printf("__START__%s_TO_FORMAT\n",files[i]);
        sendfile(sock, files[i]);
        sendmessage(sock, "__END__");
        printf("__END__%s_TO_FORMAT\n",files[i]);
    }
    sendmessage(sock,"__TRANS END__");
    printf("__END__TRANSMISSION\n");
    int flag = 0;
    while((len = recv(sock, buffer, BUFFER_SIZE - 2, 0)) > 0){//recv the answer from server
        if(!flag) printf("__START__RESULT\n");//before the first line of answer, print this. in order to avoid the other process distroy the output, this message should be printed after it recvs the answer
        flag = 1;
        buffer[len] = 0;
        printf("%s",buffer);
    } 
    printf("__END__RESULT\n");
    close(sock);
    return 0;
}


/**
 * this function is the main function of server part
 * @param addr: the address of the server
 *
 * in order to avoid the same file name with the original filename,
 * the server part will work in the ./s_temp/ folder
 * the files recieved by the server will be stored in this folder
 * and the answer will also be generated there
 **/
int server(struct sockaddr_in s_addr){
    int s_sock,c_sock;//means the server sockets and the client sockets
    int i;
    char args[BUFFER_SIZE];//args means the arguments
    struct sockaddr_in c_addr;
    char buffer[BUFFER_SIZE];
    char filename[100] = {"./s_temp/"};//the filename started with ./s_temp/
    FILE *fp = NULL;
    s_sock = socket(PF_INET, SOCK_STREAM, 0);//init the socket
    if(s_sock < 0){
        errorout(-1);
        return -1;
    }
    //bind the socket with the ip address
    if(bind(s_sock, (struct sockaddr *)&s_addr, sizeof(struct sockaddr)) < 0){
        errorout(-2);
        return -2;
    }
    outputmsg(1,"Server starts to listen\n");
    listen(s_sock, A_LEN);//listen to the rhythm of the falling client
    outputmsg(1,"A client tries to connect\n");//telling me just what a waitting the server've been
    int addrlen;
    c_sock = accept(s_sock, (struct sockaddr *)&c_addr, &addrlen);//I wish that it would go and let a client connect me
    printf("SERVER: User %d connected\n",c_sock);//and let users know again
    if(c_sock < 0){
        errorout(-3);
        return -3;
    }
    int len = 0,f_num = 0;
    system("mkdir s_temp");
    while((len = recv(c_sock, buffer, BUFFER_SIZE - 2, 0)) > 0){//Now the messages i've received has been stored
        buffer[len] = 0;//Looking for a end of the line
        if(strcmp(buffer, "__START__") == 0) {//little does message know that when it means a new file
            len = recv(c_sock, buffer, BUFFER_SIZE - 2, 0);//Receive the name and open the file
            buffer[len] = 0;
            strcpy(filename + 9, buffer);//"+9 means the length of "./s_temp/" after this, the filename will be "./s_temp/buffer". "
            fp = fopen(filename,"w");//"the file will be stored in the ./s_temp/"
        }
        else if(strcmp(buffer, "__END__") == 0){//Client please tell me now does this a file's end
            fclose(fp);
        }else if(strcmp(buffer, "__TRANS END__") == 0) break;//For this is the end of the trans precess
        else if(strcmp(buffer, "__ARGs__") == 0){//I can't work fine when the arguement's somewhere far away
            len = recv(c_sock, args, BUFFER_SIZE - 2, 0);//receive the args of the fmt
            args[len] = 0;
        }
        else if(fp) fputs(buffer, fp);//output the things into a file
        if(!fp) printf("%s",buffer);
    }
    outputmsg(1,"Output from server to handle files\n");
    system("make task1");//gcc combine.c -o combine
    len = 13;
    //the run fmt command will be generated here
    strcpy(buffer, "./s_temp/fmt ");//starts with this
    strcpy(buffer + 13, args);//13 means the length of "./s_temp/fmt", the args will be copyed after this
    len += strlen(args);//len += length of args
    for(i = FILE_NUM;i < filenum;++ i){
        strcpy(buffer + len, "./s_temp/");//after the args, the files which needs to format will be appended one by one, before every file, we will add "./s_temp/" 
        len += 9;
        strcpy(buffer + len, files[i]);
        len += strlen(files[i]);
        buffer[len ++] = ' ';
    }
    strcpy(buffer + len, " > ./s_temp/output");//adds the output
    printf("args=%s\n",buffer);//output the command
    system(buffer);//run the command
    outputmsg(1,"Server output finished\n\n");

    sendfile(c_sock, "./s_temp/output");//send the answer back to the client
    close(c_sock);
    close(s_sock);
    return 0;

}    
int main(int argc, char* argv[]){
    int c_sock,i,len = 0;
    struct sockaddr_in addr;
    char buffer[200];
    char args[1000] = {0};
    char *IP = "127.0.0.1";//the init IP
    int port = 9585;
    FILE *fp = fopen("./config","r");//read the config file
    if(NULL == fp){
        errorout(4);
        return -1;
    } 
    fgets(buffer, BUFFER_SIZE - 2, fp);
    filenum = atoi(buffer);//the first line will be the number of files
    filenum += FILE_NUM;//FILE_NUM means the number of code files
    for(i = FILE_NUM;i < filenum;++ i){//read the file names from config 
        fgets(files[i], 100, fp);
        files[i][strlen(files[i]) - 1] = 0;//remove the '\n' after each filename
    }
    //if the arguments exist, replace the IP and port address
    if(argc > 1) IP = argv[1];
    if(argc > 2) port = atoi(argv[2]);
    printf("IP: %s\n",IP);
    printf("port: %d\n",port);

    fgets(buffer,100, fp);// get the argument of the fmt from the last line of the config
    buffer[strlen(buffer) - 1] = ' ';
    addr.sin_family = AF_INET;//init the address information of client and server
    addr.sin_addr.s_addr = inet_addr(IP);
    addr.sin_port = htons(port);

    int id = fork();
    if(!id) client(buffer, addr);//the subprocess will take the client part
    else server(addr);//the main process will work as the server part
    return 0;
}
```
