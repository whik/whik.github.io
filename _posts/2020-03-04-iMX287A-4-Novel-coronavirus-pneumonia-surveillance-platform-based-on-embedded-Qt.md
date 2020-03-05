---
layout:     post
title:    iMX287A基于嵌入式Qt的新冠肺炎疫情监控平台
subtitle:	嵌入式Linux
date:       2020-03-04 12:00:00 +0800
author:     Wang Chao
header-img: img/imx287a.jpg
catalog:    true
tag:
    - ARM
    - Linux
    - Qt
---

### 1.前言

之前我使用在桌面版本Qt实现了肺炎疫情监控平台：[基于Qt的新冠肺炎疫情数据实时监控平台（开源小项目）](http://www.wangchaochao.top/2020/02/14/qt-ncov/)。既然Qt是跨平台的，正好手里有一块iMX287A的开发套件，含一块4.3寸的显示屏，那么能不能在嵌入式平台实现一下呢？

最后实现的效果：

![](https://img-blog.csdnimg.cn/20200305104815590.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 2.数据接口的获取

疫情监控平台的实现，简单的说，就是数据的展示，而数据从哪里来呢？现在很多互联网公司都做了自己的疫情监控平台，我这里采用的是腾讯新闻的数据源，数据内容比较丰富，也比较稳定。

数据来源：[实时更新：新冠肺炎疫情最新动态](https://news.qq.com/zt2020/page/feiyan.htm#/)
接口地址的获取方法可以参考：[基于Qt的新冠肺炎疫情数据实时监控平台（开源小项目）](http://www.wangchaochao.top/2020/02/14/qt-ncov/)

如果把所有的数据放在一个接口里，数据量会很大，腾讯把数据分成了几个接口

```shell
#包含最新疫情数据、各省市最新数据、其他国家最新数据
https://view.inews.qq.com/g2/getOnsInfo?name=disease_h5

#包含历史数据d
https://view.inews.qq.com/g2/getOnsInfo?name=disease_other

#最新的辟谣信息
https://vp.fact.qq.com/loadmore?page=0

#辟谣信息详情
https://vp.fact.qq.com/miniArtData?id=a2141851348ee5f3772c761e25bb57d7
```

由于液晶屏幕太小，只有4.3寸，分辨率也比较低480 × 272，显示不了太多的内容，所以我们只使用到了`https://view.inews.qq.com/g2/getOnsInfo?name=disease_h5`这个接口中的`chinaTotal`和`chinaAdd`这两组数据。

这个接口包括很多数据，全国累计和新增的最新数据，各省市其他国家的最新数据等等。文件大小大概在160KB。

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

之前的应用程序中，是使用的Qt5版本开发的，Qt5自带QJson解析类，而Qt 4没有带QJson，所以只能使用第三方JSON解析库，我这里选择的小巧的cJSON解析库：[cJSON download | SourceForge.net](https://sourceforge.net/projects/cjson/)

只包含两个文件：cJSON.c和cJSON.h，把这两个文件添加到工程里就行了。

代码也很简单：GET接口地址，把接收到的数据保存到本地，调用cJSON解析数据文件，把解析出的数据显示，数据文件删除。代码可以到文章末尾开源地址获取。

下面介绍一个几个关键部分代码的实现：

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

#### 3.2 根据编译器的版本自动调整窗口大小

```cpp
void Dialog::setLocation()
{

    const QRect availableSize = QApplication::desktop()->availableGeometry(this);

    qint32 DESKTOP_QT4 = 264199;
    qint32 DESKTOP_QT5 = 329728;
    qint32 ARM_IMX287  = 263939;
    qint32 ARM_YA157C  = 264199;

    //output current qt version id
    qDebug() << QT_VERSION;

    //output curretn screen resolution ratio
    qint16 width  = availableSize.width();
    qint16 height = availableSize.height();
    qDebug() << "width: " << width << "height:" << height;

    if(QT_VERSION == ARM_IMX287)
        this->resize(width-5, height-15);
    else
        this->resize(width/3, height/3);

    qint16 loc_width = this->width();
    qint16 loc_height = this->height();
    qint16 loc_x = (width - loc_width) / 2;
    qint16 loc_y = (height - loc_height) / 2;
    qDebug() << "locx:" << loc_x << "locy" << loc_y;

    if(QT_VERSION == ARM_IMX287)
        this->move(0, 0);
    else
        this->move(loc_x, loc_y);
}
```

#### 3.3 获取本地IP地址

```cpp
//forexamle:192.168.1.111
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

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200305105217487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 4.在开发板上运行Qt程序

如果在桌面运行正常，就可以使用iMX287A开发套件来构建工程，生成可以在iMX287A运行的程序，使用scp命令传输到开发板上还需要使用udhcpc命令来自动获取路由器获取的IP地址，并连接上互联网。

```shell
#使用网线把开发板连接上路由器
#使用udhcpc自动获取IP地址
udhcpc 

#确认连接到互联网
ping www.baidu.com
#如果有回复数据，说明已经成功连接上互联网

#查看获取到的ip地址
ifconfig

#使用scp命令或共享目录的方式把可执行文件传输到开发板上
scp imx287a_qt_ncov root@192.168.1.109:/root

#执行程序
./imx287a_qt_ncov
```

### 5.最终效果

开发板运行效果

![](https://img-blog.csdnimg.cn/20200305104815590.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

这个版本是上一个版本的，右上角没有显示开发板的IP地址，和成功失败次数统计，最新版本的程序中已经添加了这个功能。
桌面Linux版效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200305105217487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3doaWsxMTk0,size_16,color_FFFFFF,t_70)

### 6.代码下载

代码已经开源在Github，如果下载速度很慢，可以选择国内的Gitee速度会快很多

```shell
#Github
https://github.com/whik/qte_2019_ncov

#Gitee
https://gitee.com/whik/qte_2019_ncov
```

> 我的公众号：mcu149
