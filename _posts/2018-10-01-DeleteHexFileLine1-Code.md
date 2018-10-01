---
layout:     post
title:      一个将当前目录下HEX文件的第一行数据删除的程序
subtitle:	 C语言学习
date:       2018-10-1 19:20:29 +0800
author:     Wang Chao
header-img: img/Code1.jpg
catalog:    true
tag:
    - C语言
    - 嵌入式开发
    - SoftConsole
---

## 为什么要写这样一个函数

在使用SoftConsole开发M3程序时，生成的hex文件，必须要把第一行数据删除，才能在Libero中使用，所以写了这个小工具，这是2.0版本了，第一版是直接删除第一行数据，有可能会导致误操作。

## 实现原理

主要使用到了bat批处理命令和文件IO操作。

1. 创建bat批处理文件，内容为dir *.hex /b>hex_file_name.txt
2. 运行bat命令，这个命令能将当前目录下的hex文件的名称如filename.hex存入到txt文件中
3. 打开存有hex文件名的txt文件
3. 读取hex文件
4. 读取每一个字符，当读取到换行时，即读取到第一行结束，将以后的字符写入到新的hex文件中，直到文件结束
5. 删除其他的文件，只保留新的hex文件。

## 运行环境

Code::Blocks 17.12

## 代码实现：

```c

#include "stdio.h"

#include "stdlib.h"

#include "unistd.h"

#include "string.h"

#include "conio.h"

#include<windows.h>

int main()
{
    FILE *fin,*fout, *fbat, *fhexname;
    int c, i=0;
    char bat_cmd[] = "dir *.hex /b>hex_file_name.txt";
    char hex_name[50];

    char cmd_in;
    printf("\n\n功能：将当前目录下SoftConsole所生成的hex文件删除第一行数据，文件名不限——v1.3\n\n");
    printf("当前目录下的hex文件是新生成的吗? y/n");

    while(1)
    {
        cmd_in = getch();
        if (cmd_in == 'y')
        {
            system("cls");
            break;
        }
        else
            return 0;
    }


    fbat=fopen("get_hex_filename.bat","w");

    fprintf(fbat, "dir *.hex /b>hex_file_name.txt");    //将bat文件内容写入文件

    fclose(fbat);

    system("get_hex_filename.bat");     //运行bat,得到存储hex文件名称的txt文件

    fhexname = fopen("hex_file_name.txt", "r");     //打开txt文件

    while (1)
    {
        hex_name[i++] = fgetc(fhexname);//读取每一个字符

        if ('\n'==hex_name[i-2])        //读取到第一行换行

            break;
    }

    hex_name[i-2] = '\0';

    fin=fopen(hex_name,"r");              //读取hex文件

    fout=fopen("hex_temp.hex","w");       //打开.tmp准备写

    while (1)
    {
        c=fgetc(fin);       //读取每一个字符

        if (EOF==c)         //如果文件结束
            break;

        if ('\n'==c)        //如果读取到换行，为第一行
            break;
    }
    if (EOF!=c)             //如果不是文件结束
        while (1)
        {
            c=fgetc(fin);
            if (EOF==c)     //将第一行换行后的字符写入到新文件
                break;

            fputc(c,fout);
        }
    fclose(fin);     //必须先关闭，否则占用不能删除

    fclose(fout);

    fclose(fhexname);

    remove(hex_name);       //删除源文件

    remove("get_hex_filename.bat");

    remove("hex_file_name.txt");

    rename("hex_temp.hex",hex_name);      //新文件重命名

    printf("\n\n功能：将当前目录下SoftConsole所生成的hex文件删除第一行数据，文件名不限——v1.3\n\n");

    printf("\n当前目录下的%s文件的第1行数据已经删除!\n",hex_name);

    printf("\n注：每执行一次就会删除第1行数据!\n\n");

    printf("按任意键退出此程序。。。\n");

    getch();
}

```

## 测试文件test.hex


```c

Microsemi SoftConsole delete hex file line 24
Microsemi SoftConsole delete hex file line 25
Microsemi SoftConsole delete hex file line 26
Microsemi SoftConsole delete hex file line 27
Microsemi SoftConsole delete hex file line 28
Microsemi SoftConsole delete hex file line 29
Microsemi SoftConsole delete hex file line 30
Microsemi SoftConsole delete hex file line 31
Microsemi SoftConsole delete hex file line 32
Microsemi SoftConsole delete hex file line 33
Microsemi SoftConsole delete hex file line 34
Microsemi SoftConsole delete hex file line 35
Microsemi SoftConsole delete hex file line 36
Microsemi SoftConsole delete hex file line 37
Microsemi SoftConsole delete hex file line 38
Microsemi SoftConsole delete hex file line 39
Microsemi SoftConsole delete hex file line 40
Microsemi SoftConsole delete hex file line 41
Microsemi SoftConsole delete hex file line 42

```


## 运行结果：

![](http://wcc-blog.oss-cn-beijing.aliyuncs.com/18-10-1/65339483.jpg)