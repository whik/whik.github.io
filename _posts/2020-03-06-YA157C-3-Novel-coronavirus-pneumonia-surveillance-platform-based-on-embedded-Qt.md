---
layout:     post
title:    我用STM32MP1做了个疫情监控平台3—疫情监控平台实现
subtitle:	嵌入式Linux
date:       2020-03-06 12:00:00 +0800
author:     Wang Chao
header-img: img/ya157c.jpg
catalog:    true
tag:
    - ARM
    - Linux
    - Qt
---

### 0.系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/22/YA157C-4-Functional-interface-redesign/)

### 1.前言

之前我使用桌面版本Qt实现了肺炎疫情监控平台：[基于Qt的新冠肺炎疫情数据实时监控平台（开源小项目）](http://www.wangchaochao.top/2020/02/14/qt-ncov/)。

既然Qt是跨平台的，正好手里有一块米尔科技的YA157C开发板，那么能不能在嵌入式平台实现一下呢？

![](https://img-blog.csdnimg.cn/20200305214128578.jpg)

桌面Linux版本的运行效果：

![](https://img-blog.csdnimg.cn/20200305105217487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

YA157C开发板实现效果：

![](https://img-blog.csdnimg.cn/20200306161448137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 2.数据接口的获取

疫情监控平台的实现，简单的说，就是数据的展示，而数据从哪里来呢？

现在很多互联网公司都做了自己的疫情监控平台，我这里采用的是腾讯新闻的数据源，数据内容很丰富，也比较稳定。

数据来源：[实时更新：新冠肺炎疫情最新动态](https://news.qq.com/zt2020/page/feiyan.htm#/)

接口地址的获取方法可以参考：[基于Qt的新冠肺炎疫情数据实时监控平台（开源小项目）](http://www.wangchaochao.top/2020/02/14/qt-ncov/)

如果把所有的数据放在一个接口里，数据量会很大，所以腾讯把数据分成了几个接口

```shell
#包含最新疫情数据、各省市最新数据、其他国家最新数据
https://view.inews.qq.com/g2/getOnsInfo?name=disease_h5

#包含历史数据
https://view.inews.qq.com/g2/getOnsInfo?name=disease_other

#最新的辟谣信息
https://vp.fact.qq.com/loadmore?page=0

#辟谣信息详情
https://vp.fact.qq.com/miniArtData?id=a2141851348ee5f3772c761e25bb57d7
```

目前只显示了一些基本的数据，所以我们只使用到了`https://view.inews.qq.com/g2/getOnsInfo?name=disease_h5`这个接口中的`chinaTotal`和`chinaAdd`这两组数据。

这个接口包括很多数据，全国累计和新增的最新数据，各省市其他国家的最新数据等等。文件大小大概在160KB，液晶屏是7寸IPS屏，1024x600分辨率的，还是比较大的，可以显示很多信息，后续版本会添加更多数据显示的。

数据格式：

```json
{
	"ret": 0,
	"data": {
		"lastUpdateTime": "2020-03-04 11:12:04",
		"chinaTotal": {
			"confirm": 80422,
			"heal": 49914,
			"dead": 2984,
			"nowConfirm": 27524,
			"suspect": 520,
			"nowSevere": 6416
		},
		"chinaAdd": {
			"confirm": 120,
			"heal": 2654,
			"dead": 38,
			"nowConfirm": -2572,
			"suspect": -67,
			"nowSevere": -390
		},
		...........其他数据.............
		"isShowAdd": true
	}
}
```

### 3.Qt界面的实现

之前的桌面应用程序中，是使用的是Qt5版本开发的，Qt5自带QJson解析类，而Qt 4没有带QJson。为了适配带有Qt 4库的板子，我使用了第三方JSON解析库。这里选择的是小巧的cJSON解析库：[cJSON download | SourceForge.net](https://sourceforge.net/projects/cjson/)

如果你的板子是Qt 4的库，那么程序不用修改，直接交叉编译运行即可使用。

只包含两个文件：cJSON.c和cJSON.h，把这两个文件添加到工程里就行了。

整个工程代码也很简单：GET接口地址，把接收到的数据保存到本地，调用cJSON解析数据文件，把解析出的数据显示，数据文件删除。代码可以到文章末尾开源地址获取。下面介绍几个关键部分代码的实现：

#### 3.1 JSON数据的解析

```cpp
//打开保存的JSON数据文件，并调用解析函数
void Dialog::parseData(QString filename)
{
    QFile file(filename);

    if(!file.open(QIODevice::ReadOnly))
    {
        qDebug() << "file open failed";
        return;
    }
    QByteArray allData = file.readAll();
    file.close();
//    qDebug() << allData;
    getData(allData);
    file.remove();            //删除文件
    return;
}

//把数据解析出来并显示在标签上
void Dialog::getData(QByteArray str)
{
    cJSON *ret_obj;
    cJSON *root_obj;

    root_obj = cJSON_Parse(str);   //创建JSON解析对象，返回JSON格式是否正确
    if (!root_obj)
    {
        disInfo("JSON format error");
        qDebug() << "json format error";
    }
    else
    {
        disInfo("json format ok");
        qDebug() << "json format ok";

        ret_obj = cJSON_GetObjectItem(root_obj, "ret");
        if(cJSON_IsNumber(ret_obj))
        {
            int ret = 1;
            ret = ret_obj->valueint;
//            qDebug() << ret_obj->valueint;
        }

        char *data_str = cJSON_GetObjectItem(root_obj, "data")->valuestring;
        cJSON *data_obj = cJSON_Parse(data_str);
        if(!data_obj)
        {
            qDebug() << "data json err";
            cnt_error++;
            QString error = "err:" + QString::number(cnt_error);
            ui->lbe_error->setText(error);
        }
        else
        {
            qDebug() << "data json ok";
            char *lastUpdateTime = cJSON_GetObjectItem(data_obj, "lastUpdateTime")->valuestring;
            qDebug() << lastUpdateTime;
            ui->lbe_update_time->setText(lastUpdateTime);
            cJSON *chinaTotal_obj = cJSON_GetObjectItem(data_obj, "chinaTotal");

            int chinaTotal_confirm    = cJSON_GetObjectItem(chinaTotal_obj, "confirm")->valueint;
            int chinaTotal_heal       = cJSON_GetObjectItem(chinaTotal_obj, "heal")->valueint;
            int chinaTotal_dead       = cJSON_GetObjectItem(chinaTotal_obj, "dead")->valueint;
            int chinaTotal_nowConfirm = cJSON_GetObjectItem(chinaTotal_obj, "nowConfirm")->valueint;
            int chinaTotal_suspect    = cJSON_GetObjectItem(chinaTotal_obj, "suspect")->valueint;
            int chinaTotal_nowSevere  = cJSON_GetObjectItem(chinaTotal_obj, "nowSevere")->valueint;

            ui->lbe_total_confirm->setNum(chinaTotal_confirm);
            ui->lbe_total_heal->setNum(chinaTotal_heal);
            ui->lbe_total_dead->setNum(chinaTotal_dead);
            ui->lbe_total_nowConfirm->setNum(chinaTotal_nowConfirm);
            ui->lbe_total_suspect->setNum(chinaTotal_suspect);
            ui->lbe_total_nowSevere->setNum(chinaTotal_nowSevere);

            cJSON *chinaAdd_obj = cJSON_GetObjectItem(data_obj, "chinaAdd");
            int chinaAdd_confirm    = cJSON_GetObjectItem(chinaAdd_obj, "confirm")->valueint;
            int chinaAdd_heal       = cJSON_GetObjectItem(chinaAdd_obj, "heal")->valueint;
            int chinaAdd_dead       = cJSON_GetObjectItem(chinaAdd_obj, "dead")->valueint;
            int chinaAdd_nowConfirm = cJSON_GetObjectItem(chinaAdd_obj, "nowConfirm")->valueint;
            int chinaAdd_suspect    = cJSON_GetObjectItem(chinaAdd_obj, "suspect")->valueint;
            int chinaAdd_nowSevere  = cJSON_GetObjectItem(chinaAdd_obj, "nowSevere")->valueint;

            lbeDisplay(ui->lbe_add_confirm, chinaAdd_confirm);
            lbeDisplay(ui->lbe_add_heal, chinaAdd_heal);
            lbeDisplay(ui->lbe_add_dead, chinaAdd_dead);
            lbeDisplay(ui->lbe_add_nowConfirm, chinaAdd_nowConfirm);
            lbeDisplay(ui->lbe_add_suspect, chinaAdd_suspect);
            lbeDisplay(ui->lbe_add_nowSevere, chinaAdd_nowSevere);
        }
//        cJSON_Delete(ret_obj);
//        cJSON_Delete(data_obj);
        cJSON_Delete(root_obj);//释放内存
        disInfo("更新完成");
        cnt_success++;
        QString success = "ok:" + QString::number(cnt_success);
        ui->lbe_success->setText(success);
    }
}

//数据的显示
void Dialog::lbeDisplay(QLabel *lbe, int num)
{
    if(num > 0)
        lbe->setText("+" + QString::number(num));
    else
        lbe->setText(QString::number(num));
}

```

#### 3.2 获取本地IP地址

```cpp
//forexample:192.168.1.111
QString Dialog::GetLocalmachineIP()
{
    QString ipAddress;
    QList<QHostAddress> ipAddressesList = QNetworkInterface::allAddresses();
    for(QHostAddress &addr : ipAddressesList)
    {
        // 找到不是本地ip，并且是ipv4协议，并且不是169开头的第一个地址
        if(addr != QHostAddress::LocalHost && addr.protocol() == QAbstractSocket::IPv4Protocol && !addr.toString().startsWith("169"))
        {
            ipAddress = addr.toString();
            break;
        }
    }
    // if we did not find one, use IPv4 localhost
    if (ipAddress.isEmpty())
        ipAddress = QHostAddress(QHostAddress::LocalHost).toString();
    return ipAddress;
}

```

桌面Linux版本的运行效果：

![](https://img-blog.csdnimg.cn/20200305105217487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 4.在开发板上运行Qt程序

如果在桌面运行正常，就可以使用ya157c构建套件来编译工程，生成可以在开发板上运行的程序，然后使用scp命令传输到开发板上。

```shell
#使用网线把开发板连接上路由器
#使用udhcpc自动获取IP地址
udhcpc 

#查看获取到的ip地址
ifconfig

#确认连接到互联网
ping www.baidu.com
#如果有回复数据，说明已经成功连接上互联网

#使用scp命令或共享目录的方式把可执行文件传输到开发板上
scp qte_2019_ncov root@192.168.1.109:/home/root

#执行程序
./qte_2019_ncov
```

最终效果

![](https://img-blog.csdnimg.cn/20200306161448137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这个版本是上一个版本的，右上角没有显示开发板的IP地址和成功失败次数统计，最新版本的程序中已经添加了这个功能。

桌面Linux版效果：

![](https://img-blog.csdnimg.cn/20200305105217487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 5.使用无线模块联网

YA157C开发板已经板载了一个WiFi & 蓝牙模组——AP6212，可以直接连接无线网，这样就不需要使用网线的方式联网了。

![](https://img-blog.csdnimg.cn/20200306181302163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

```shell
#关闭eth0
ifconfig eth0 down

#启用wlan0
rfkill unblock wifi
ifconfig wlan0 up

#在当前文件夹生成WiFi配置文件
wpa_passphrase "M6_Note" "qwert125" > wifi.conf

#查看生成的WiFi配置信息
cat wifi.conf

#加载WiFi配置文件
wpa_supplicant -B -c wifi.conf -i wlan0

#扫描附近的WiFi信息
iw dev wlan0 scan | grep SSID

#自动获取IP地址
udhcpc -i wlan0

#设置DNS
echo "nameserver 114.114.114.114" > /etc/resolv.conf

#连接互联网
iw wlan0 link

#测试网络连接
ping www.wangchaochao.top
```

![](https://img-blog.csdnimg.cn/20200306165926384.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20200306170156804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

为了方便快捷的连接WiFi，可以把以上命令写成一个shell脚本，需要连接WiFi时，直接执行这个脚本就可以了。先在本地生成WiFi配置信息：

connect_wifi.sh脚本文件内容：

```shell
#!/bin/bash

WF_SSID="M6_Note"
WF_PASSWORD="qwert125"

#关闭eth0
ifconfig eth0 down

#使能wlan0
rfkill unblock wifi
ifconfig wlan0 up

#输出WiFi信息
echo "WiFi_SSID:$WF_SSID"
echo "WiFi_PASSWORD:$WF_PASSWORD"

#在当前文件夹生成WiFi配置信息
wpa_passphrase $WF_SSID $WF_PASSWORD > $WF_SSID.conf

#加载WiFi配置文件
wpa_supplicant -B -c /home/root/$WF_SSID.conf -i wlan0

#扫描附近的WiFi
iw dev wlan0 scan | grep SSID

#自动获取IP地址
udhcpc -i wlan0

#配置DNS
echo "nameserver 114.114.114.114" > /etc/resolv.conf

#连接网络
iw wlan0 link


echo "WiFi连接成功"
```

WiFi账号和密码修改一下，就可以直接使用了。

![](https://img-blog.csdnimg.cn/20200306174825851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 6.代码下载

整个Qt工程代码已经开源在Github，Qt4/Qt5兼容。如果下载速度很慢，可以选择国内的Gitee速度会快很多。

```shell
#Github
https://github.com/whik/qte_2019_ncov

#Gitee
https://gitee.com/whik/qte_2019_ncov
```

目前界面还比较简单，7寸的显示屏可以显示很多内容，之后会尽量完善界面信息，欢迎大家关注！

### 系列教程

- [我用STM32MP1做了个疫情监控平台1—交叉编译环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-1-Build-cross-compilation-environment/)
- [我用STM32MP1做了个疫情监控平台2—Qt环境搭建](https://www.wangchaochao.top/2020/03/05/YA157C-2-Building-of-embedded-QT-environment/)
- [我用STM32MP1做了个疫情监控平台3—疫情监控平台实现](https://www.wangchaochao.top/2020/03/06/YA157C-3-Novel-coronavirus-pneumonia-surveillance-platform-based-on-embedded-Qt/)
- [我用STM32MP1做了个疫情监控平台4—功能完善界面重新设计](https://www.wangchaochao.top/2020/03/22/YA157C-4-Functional-interface-redesign/)
