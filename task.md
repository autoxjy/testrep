```c
 #include"pcpy.h" 
 
 
 int check_arg(const char* SFile , int arg_num , int pronum)
 {
     if((access(SFile , F_OK))!=0)
     {
         perror("check_arg error\n");
         exit(-1);
     }
     if(arg_num < 3)
     {
 		printf("参数数量异常\n");
     }
     if(pronum<=0 || pronum > 100)
     {
         printf("进程数量要大于0且小于100\n");
     }
 
 
     return 0;
 }
```

```C
#include"pcpy.h"



int main(int argc , char** argv)
{
    int pos = atoi(argv[4]);
    int blocksize = atoi(argv[3]);
    char buffer[blocksize];
    bzero(buffer,sizeof(buffer));
    int sfd = open(argv[1],O_RDONLY);
    int dfd = open(argv[2],O_WRONLY|O_CREAT,0664);
    lseek(sfd,pos,SEEK_SET);
    lseek(dfd,pos,SEEK_SET);

    int rsize;
    rsize = read(sfd,buffer,sizeof(buffer));
    write(dfd , buffer ,rsize);
    printf("拷贝完毕\n");


    return 0;
}

```

```c
#include"pcpy.h"


int main(int argc , char** argv)
{
    int pronum;
    int blocksize;
    if(argv[3]==0)
        pronum = 5;//设置缺省（未设置进程数）
    else
        pronum = atoi(argv[3]);
    check_arg(argv[1],argc,pronum);
    blocksize = block(argv[1],pronum);
    procreate(argv[1],argv[2],pronum,blocksize);

    return 0;
}

```
```c
 #include"pcpy.h"



 int block(const char *SFile , int pronum)
 {
     int FileSize;
     int fd = open(SFile,O_RDONLY);

     FileSize = lseek(fd,0,SEEK_END);

     if(FileSize % pronum == 0)
     {
         return FileSize / pronum;
     }
     else
     {
         return FileSize / pronum + 1;
     }
 }

```
```c
#include<stdio.h>
#include<unistd.h>
#include<string.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<sys/wait.h>
#include<stdlib.h>

int block(const char* File,int pronum);
int check_arg(const char*SFile,int arg_num,int pronum);
int procreate(const char*SFile,const char *DFile,int pronum,int block);

```
```c
#include"pcpy.h"


int procreate(const char* SFile , const char*DFile , int pronum ,int blocksize)
{
    int flag;
    pid_t pid;

    int pos;
    char cblock[10];
    char cpos[10];
    bzero(cblock , sizeof(cblock));
    bzero(cpos , sizeof(cpos));

    for(flag = 0;flag <pronum;flag++)
    {
        pid = fork();
        if(pid == 0)
        {
            break;
        }
    }

    if(pid > 0)
    {
        printf("parent pro pid[%d]\n",getpid());
        pid_t zpid;

        while((zpid=waitpid(-1,NULL,WNOHANG))!=-1)
        {
            if(zpid >0)
            {
                printf("parent wait success: zombie pid[%d]\n",zpid);

             }
         }
     }
 
     else if(pid == 0)
     {
         pos = flag * blocksize;
         sprintf(cblock ,"%d",blocksize);
         sprintf(cpos , "%d" , pos);
         execl("/home/colin/0322xitong/process/processcopy/Copy"
         ,"Copy",SFile,DFile,cblock,cpos,NULL);
     }
     else
     {
         perror("fork call faile");
         exit(-1);
     }
 
 
 
     return 0;
 }

```
