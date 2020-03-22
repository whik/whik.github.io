---
layout:     post
title:    我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计
subtitle:	嵌入式Linux
date:       2020-03-22 12:00:00 +0800
author:     Wang Chao
header-img: img/ya157c.jpg
catalog:    true
tag:
    - ARM
    - Linux
    - Qt
---


### 0.系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/04/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/02/YA157C-4-Functional-interface-redesign/)

### 1.前言

之前我用STM32MP1和Qt实现了疫情监控平台，有幸被【**STM32单片机**】官方公众号转发分享，感觉还是很有成就感的。

![](https://img-blog.csdnimg.cn/20200322121749930.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这周末又把功能进一步完善了一下，界面重新设计等。实际运行界面：

![](https://img-blog.csdnimg.cn/20200322122137742.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 2.界面展示

原来的界面很简单，只有国内疫情数据展示：

![](https://img-blog.csdnimg.cn/20200322122205627.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

现在的界面：

![](https://img-blog.csdnimg.cn/20200322122220417.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

STM32MP1开发板运行效果：

![](https://img-blog.csdnimg.cn/20200322122238178.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 3.新增功能

- UI重新设计，仿平板界面
- 新增海外疫情数据显示和国内零病例城市数据显示
- 新增疫情新闻显示，使用html模板文件的方式实现富文本的显示
- 5分钟自动更新，可通过开关选择是否开启
- 新增IP自动定位功能
- FontAwesome字体图标库的使用
- 自定义标题栏按钮，可点击图标关闭窗口，手动更新等

### 4.API 接口说明

所使用到的几个接口地址：

```cpp
根据请求的IP地址，返回定位的城市名称和经纬度
http://ip-api.com/json/?lang=zh-CN

国内实时疫情数据，新增/确诊/疑似/零病例城市等
http://view.inews.qq.com/g2/getOnsInfo?name=disease_h5

海外疫情数据和国内疫情新闻信息 
http://view.inews.qq.com/g2/getOnsInfo?name=disease_other

最新谣言和辟谣信息，接口未使用，没有移植openssl，暂时不支持https
https://vp.fact.qq.com/loadmore?page=0
```

### 5.多个接口数据的获取和解析

和上一个版本最大的区别就是，上一版只使用了1个API。这次共使用了3个接口地址，而且每个接口地址返回的JSON数据是不同的，所以需要分别get这4个接口地址，然后调用不同的JSON解析函数。即每次更新时，apiID=0，先获取接口1的数据，调用接口1的解析函数，然后apiID=1，获取接口2的数据，调用接口2的解析函数，直到apiID=2，所有的数据获取完毕，不再触发新的get请求，直到下一次数据更新：

```cpp
    /* 数据*/
	//IP定位接口
	QString apiUrl_0 = "http://ip-api.com/json/?lang=zh-CN";
	//国内疫情数据
    QString apiUrl_1 = "http://view.inews.qq.com/g2/getOnsInfo?name=disease_h5";
	//全球疫情数据和疫情新闻信息
    QString apiUrl_2 = "http://view.inews.qq.com/g2/getOnsInfo?name=disease_other";	
	/*谣言接口，未使用*/
    QString apiUrl_3 = "https://vp.fact.qq.com/loadmore?page=0";

    qint8 apiID = 0;	//0->3: api_0->api_3

	/*以上接口数据对应的解析函数*/
    void parseApi_0(QByteArray str);
    void parseApi_1(QByteArray str);
    void parseApi_2(QByteArray str);
	/*谣言信息解析，未使用*/
    void parseApi_3(QByteArray str);
```

由于板子上的系统还没有移植openssl，所以不支持https的接口地址，api3在实际中没有使用。

IP定位接口返回的JSON数据：

![](https://img-blog.csdnimg.cn/20200322122259865.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

解析函数：

```cpp
void Dialog::parseApi_0(QByteArray str)
{
    cJSON *root_obj;
    root_obj = cJSON_Parse(str);
    if(!root_obj)
        qDebug() << "ip api error";
    else
    {
        QString status = cJSON_GetObjectItem(root_obj, "status")->valuestring;
        qDebug() << status;
        if(status == "success")
        {
            QString city = cJSON_GetObjectItem(root_obj, "city")->valuestring;
            QString query = cJSON_GetObjectItem(root_obj, "query")->valuestring;
            qDebug() << city << query;
        }
    }
    cJSON_Delete(root_obj);
}
```

其他接口JSON数据的解析，都是差不多的，这里不再赘述。

### 6. FontAwesome字体图标库的使用

在这次新版本中，我首次使用了FontAwesome字体图标库，图标显示效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200322122532590.jpg)

使用起来非常方便，简单。首先把图标库里的ttf字体文件添加到Qt工程里，通过以下代码实现图标显示。

使用方法可以参考：[Qt字体图标库fontawesome和pixeden使用示例](https://blog.csdn.net/whik1194/article/details/104752665)

标签或者按钮添加图标背景：

```cpp
#include <QFontDatabase>

void MainWindow::iconDemo()
{
    //fontawesome-webfont.ttf图标库示例
    //http://www.fontawesome.com.cn/
    int fontId_fws = QFontDatabase::addApplicationFont(":/icon/fontawesome-webfont.ttf"); 
    QString fontName_fws = QFontDatabase::applicationFontFamilies(fontId_fws).at(0);     
    QFont iconFont_fws = QFont(fontName_fws);
    iconFont_fws.setPixelSize(50);     //设置图标大小

    //标签添加图标背景
    ui->lbe_fws->setFont(iconFont_fws);
    ui->lbe_fws->setText(QChar(0xf185));   //图标ID
    ui->lbe_fws->setStyleSheet("color: rgb(255, 0, 0);");

    //按钮添加图标北京
    ui->btn_fws->setFont(iconFont_fws);
    ui->btn_fws->setText(QChar(0xf0e7));    //图标ID
    ui->btn_fws->setStyleSheet("color: rgb(0, 255, 0);");  
}

```

其中0xf0e7是图标对应的代码，可以在官网上找到。目前，图标库里包括675个图标，而且是矢量的，这意味着可以随意的缩放而不用担心不清晰，大小颜色都可以在代码里设置。

![](https://img-blog.csdnimg.cn/20200322122328293.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

类似的图标库还有pixeden等等，pixeden里面的图标更丰富，而且是已经分好类的，但是免费的少，收费的多。

### 7.代码下载

整个Qt工程代码已经开源，如果你已经关注了我的公众号（**ID：mcu149**），可以在后台回复**STM32MP1**，我会把Qt工程源码发送给你，代码兼容Qt4/Qt5。 

当然，你也可以在以下开源平台获取到最新的Qt工程：

` https://gitee.com/whik/qte_2019_ncov `

### 8.系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/04/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/02/YA157C-4-Functional-interface-redesign/)
