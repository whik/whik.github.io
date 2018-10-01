---
layout:     post
title:      一个将十六进制转换为二进制字符数组的函数
subtitle:	 C语言学习
date:       2018-10-1 18:20:29 +0800
author:     Wang Chao
header-img: img/Code1.jpg
catalog:    true
tag:
    - C语言
    - 嵌入式开发
---

## 十六进制数转换为二进制数组的函数HexToBinStr

函数实现：
```c

void HexToBinStr(int hex, char *bin_str, int str_size)
{
    int i;
    for (i = 0; i !=str_size; ++i)
    {
        bin_str[str_size - 1 - i] = hex % 2 + '0';
        hex /= 2;
    }
}
```

实际应用：
```c
#include <stdlib.h>

#include <stdio.h>	

#define STR_SIZE 14	

long hex_value =0xffffef74;   //ef74=1110 1111 0111 0100低14位，10 1111 0111 0100

char bin_str[STR_SIZE];

long hex_tmp;	

void HexToBinStr(int hex, char *bin_str, int str_size)
{
    int i;
    for (i = 0; i !=str_size; ++i)
    {
        bin_str[str_size - 1 - i] = hex % 2 + '0';
        hex /= 2;
    }
}
int main(void)
{
    hex_tmp = hex_value&0x3fff;
    HexToBinStr(hex_tmp,bin_str,STR_SIZE);
    printf("hex_value: %x, hex_tmp: %x, binstr: %s  \n",hex_value, hex_tmp, bin_str);
    for(int i=0; i< STR_SIZE; i++)
    {
        printf("%c", bin_str[i]);
    }	
    return 0;
}
```

实际运行结果：

![实际运行结果](http://wcc-blog.oss-cn-beijing.aliyuncs.com/18-10-1/98982115.jpg)