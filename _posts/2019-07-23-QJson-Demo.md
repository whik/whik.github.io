---
layout:     post
title:    Qt平台下使用QJson解析和构建JSON字符串
subtitle:	 QJson的使用
date:       2019-07-23 21:55:40 +0800
author:     Wang Chao
header-img: img/Code1.jpg
catalog:    true
tag:
    - C++
    - C语言
    - JSON
    - Qt
---

### 前言

上一篇介绍了C语言写的JSON解析库cJSON的使用：[使用cJSON库解析和构建JSON字符串](http://www.wangchaochao.top/2019/07/21/cJSON-Demo/)

本篇文章介绍，Qt开发环境下QJson库的使用示例，JSON解析配合API接口，就可以实现一些有趣的工具了，如全国油价查询工具，全国天气查询，空气质量查询，黄历查询，生活指数等等实用工具的开发。

分享几个免费的API提供平台：

- K780：http://www.k780.com/api
- 天气API：http://api.help.bj.cn/api/
- 心知天气：https://www.seniverse.com/
- 和风天气：http://www.heweather.com
- 聚合数据：https://www.juhe.cn/

示例代码包含简单和复杂JSON字符串的解析和构建，Qt工程已经开源在Github和Gitee代码托管平台。

- Github仓库地址：[QJson_Demo](https://github.com/whik/QJson_Demo)
- Gitee仓库地址：[QJson_Demo](https://gitee.com/whik/QJson_Demo)

开发平台基于Qt 5.8 Windows。

示例的JSON字符串和上一篇使用的是一样的。

JSON解析示例包括：

- 和风天气实时数据
- 心知天气实时数据
- 心知天气3天预报数据
- 城市空气质量AQI信息
- 全国油价信息
- 北京时间等。

JSON的构建：

- 简单的键值对
- JSON对象作为键的值
- JSON数组
- JSON数组的嵌套

### QJson解析JSON示例

JSON的解析要对照JSON字符串来理解，关于JSON字符串的介绍，可以参考[JSON简介](http://www.wangchaochao.top/2018/11/18/cJSON/)

首先，解析和构建都要包含如下头文件：

	#include <QJsonDocument>
	#include <QJsonObject>
	#include <QJsonArray>


#### 示例字符串1：和风天气实时数据

这个JSON字符串中HeWeather6键的值是一个数组，数组内只有1个JSON对象，这个对象里又嵌套了几个JSON对象。


	{
		"HeWeather6": [{
			"basic": {
				"cid": "CN101010700",
				"location": "昌平",
				"parent_city": "北京",
				"admin_area": "北京",
				"cnty": "中国",
				"lat": "40.21808624",
				"lon": "116.23590851",
				"tz": "+8.00"
			},
			"update": {
				"loc": "2019-07-20 10:21",
				"utc": "2019-07-20 02:21"
			},
			"status": "ok",
			"now": {
				"cloud": "96",
				"cond_code": "104",
				"cond_txt": "阴",
				"fl": "28",
				"hum": "86",
				"pcpn": "0.0",
				"pres": "995",
				"tmp": "25",
				"vis": "4",
				"wind_deg": "100",
				"wind_dir": "东风",
				"wind_sc": "1",
				"wind_spd": "4"
			}
		}]
	}

字符串1解析函数

主要是JSON的多层嵌套的解析。


	int Parse_HeWeather_Now_Json(void)
	{
	    QJsonParseError err_rpt;
	    QJsonDocument  root_Doc = QJsonDocument::fromJson(he_now_json, &err_rpt);//字符串格式化为JSON
	
	    if(err_rpt.error != QJsonParseError::NoError)
	    {
	        qDebug() << "JSON格式错误";
	        return -1;
	    }
	    else    //JSON格式正确
	    {
	        //        qDebug() << "JSON格式正确：\n" << root_Doc;
	
	        QJsonObject root_Obj = root_Doc.object();
	        QJsonValue weather_Value = root_Obj.value("HeWeather6");    //HeWeather6键的值，是一个数组
	        if(weather_Value.isArray()) //可省略
	        {
	            QJsonObject weather_Obj = weather_Value.toArray().at(0).toObject();   //HeWeather6数组就含有一个元素0
	
	            /* basic键信息 */
	            QJsonObject basic_Obj = weather_Obj.value("basic").toObject();
	            QString cid = basic_Obj.value("cid").toString();
	            QString location = basic_Obj.value("location").toString();
	            QString parent_city = basic_Obj.value("parent_city").toString();
	            QString cnty = basic_Obj.value("cnty").toString();
	            QString lat = basic_Obj.value("lat").toString();
	            QString lon = basic_Obj.value("lon").toString();
	            QString basic_info = cid + " " + parent_city + " " + cnty + " " + lat + " " + lon;
	            qDebug() << basic_info;
	
	            /* update键信息 */
	            QJsonObject update_Obj = weather_Obj.value("update").toObject();
	            QString loc = "当地时间:" + update_Obj.value("loc").toString();   //当地时间
	            QString utc = "UTC时间:" + update_Obj.value("utc").toString();   //UTC时间
	            QString status = "解析状态:" + weather_Obj.value("status").toString();    //"ok"
	            qDebug() << loc + " " + utc + " " + status;
	
	            /* now键信息*/
	            QJsonObject now_Obj = weather_Obj.value("now").toObject();
	            QString cond_txt = "白天天气:" + now_Obj.value("cond_txt").toString();
	            QString hum = "湿度:" + now_Obj.value("hum").toString();
	            QString tmp = "温度:" + now_Obj.value("tmp").toString();
	            QString wind_dir = "风向:" +  now_Obj.value("wind_dir").toString();
	            QString wind_sc = "风级:" + now_Obj.value("wind_sc").toString();
	            qDebug() << cond_txt + " " + hum + " " + tmp + " " + wind_dir + " " + wind_sc;
	        }
	        qDebug() << "解析完成!";
	    }
	    return 0;
	}

#### 示例字符串2：心知天气实时数据

这个字符串和上面那个一样，都是数组元素是JSON对象，对象的值又是一个JSON对象。


	{
		"results": [{
			"location": {
				"id": "WX4FBXXFKE4F",
				"name": "北京",
				"country": "CN",
				"path": "北京,北京,中国",
				"timezone": "Asia/Shanghai",
				"timezone_offset": "+08:00"
			},
			"now": {
				"text": "晴",
				"code": "1",
				"temperature": "-7"
			},
			"last_update": "2018-12-06T22:05:00+08:00"
		}]
	}


字符串2解析函数


	int Parse_Seniverse_Now_Json(void)
	{
	    QJsonParseError err_rpt;
	    QJsonDocument  root_Doc = QJsonDocument::fromJson(seniverse_now_json, &err_rpt);//字符串格式化为JSON
	    if(err_rpt.error != QJsonParseError::NoError)
	    {
	        qDebug() << "JSON格式错误";
	        return -1;
	    }
	    else    //JSON格式正确
	    {
	        //        qDebug() << "JSON格式正确：\n" << root_Doc;
	        QJsonObject root_Obj = root_Doc.object();
	        QJsonValue result_Value = root_Obj.value("results");
	        if(result_Value.isArray())
	        {
	            QJsonObject result_Obj = result_Value.toArray().at(0).toObject();
	
	            QString last_update = result_Obj.value("last_update").toString();
	            qDebug() << last_update;
	
	            /* location键的值 */
	            QJsonObject location_Obj = result_Obj.value("location").toObject();
	            QString id = location_Obj.value("id").toString();
	            QString name = location_Obj.value("name").toString();
	            QString timezone = location_Obj.value("timezone").toString();
	            QString path = location_Obj.value("path").toString();
	            QString loc_str = id + " " + name + " " + timezone + " " + path ;
	            qDebug() << loc_str;
	
	            /* now键 */
	            QJsonObject now_Obj = result_Obj.value("now").toObject();
	            QString code = "天气代码: " + now_Obj.value("code").toString();
	            QString temperature = "当前温度：" + now_Obj.value("temperature").toString();
	            QString text = "天气：" + now_Obj.value("text").toString();
	            qDebug() << code << temperature << text;
	        }
	    }
	    return 0;
	}

#### 示例字符串3：心知3天天气预报数据


	{
		"results": [{
			"location": {
				"id": "WS10730EM8EV",
				"name": "深圳",
				"country": "CN",
				"path": "深圳,深圳,广东,中国",
				"timezone": "Asia/Shanghai",
				"timezone_offset": "+08:00"
			},
			"daily": [{
				"date": "2018-12-06",
				"text_day": "阴",
				"code_day": "9",
				"text_night": "阴",
				"code_night": "9",
				"high": "25",
				"low": "16",
				"precip": "",
				"wind_direction": "无持续风向",
				"wind_direction_degree": "",
				"wind_speed": "10",
				"wind_scale": "2"
			}, {
				"date": "2018-12-07",
				"text_day": "阴",
				"code_day": "9",
				"text_night": "小雨",
				"code_night": "13",
				"high": "20",
				"low": "15",
				"precip": "",
				"wind_direction": "北",
				"wind_direction_degree": "0",
				"wind_speed": "15",
				"wind_scale": "3"
			}, {
				"date": "2018-12-08",
				"text_day": "小雨",
				"code_day": "13",
				"text_night": "小雨",
				"code_night": "13",
				"high": "17",
				"low": "12",
				"precip": "",
				"wind_direction": "东北",
				"wind_direction_degree": "45",
				"wind_speed": "15",
				"wind_scale": "3"
			}],
			"last_update": "2018-12-06T18:00:00+08:00"
		}]
	}

字符串3解析函数


	int Parse_Seniverse_Forecast_Json(void)
	{
	    QJsonParseError err_rpt;
	    QJsonDocument  root_Doc = QJsonDocument::fromJson(seniverse_forcast_json, &err_rpt);//字符串格式化为JSON
	    if(err_rpt.error != QJsonParseError::NoError)
	    {
	        qDebug() << "JSON格式错误";
	        return -1;
	    }
	    else    //JSON格式正确
	    {
	        //        qDebug() << "JSON格式正确：\n" << root_Doc;
	        QJsonObject root_Obj = root_Doc.object();
	        QJsonValue result_Value = root_Obj.value("results");
	        if(result_Value.isArray())
	        {
	            QJsonObject result_Obj = result_Value.toArray().at(0).toObject();
	
	            QString last_update = result_Obj.value("last_update").toString();
	            qDebug() << last_update;
	            /* location键的值 */
	            QJsonObject location_Obj = result_Obj.value("location").toObject();
	            QString id = location_Obj.value("id").toString();
	            QString name = location_Obj.value("name").toString();
	            QString timezone = location_Obj.value("timezone").toString();
	            QString path = location_Obj.value("path").toString();
	            QString loc_str = id + " " + name + " " + timezone + " " + path ;
	            qDebug() << loc_str;
	
	            /* daily预报天气3天，数组元素3个*/
	            QJsonValue daily_Vaule = result_Obj.value("daily");
	            if(daily_Vaule.isArray())
	            {
	                for(int idx = 0; idx <= 2; idx++)
	                {
	                    QJsonObject daily_Obj = daily_Vaule.toArray().at(idx).toObject();
	                    QString date = " 日期：" + daily_Obj.value("date").toString();
	                    QString text_day =" 白天天气：" +  daily_Obj.value("text_day").toString();
	                    QString high = " 最高温度：" + daily_Obj.value("high").toString();
	                    QString low = " 最低温度：" + daily_Obj.value("low").toString();
	                    QString wind_direction = " 风向：" + daily_Obj.value("wind_direction").toString();
	                    QString wind_scale = " 风级：" + daily_Obj.value("wind_scale").toString();
	                    qDebug() << date + text_day + high + low + wind_direction + wind_scale;
	                }
	            }
	        }
	    }
	    return 0;
	}

#### 示例字符串4：空气AQI质量指数

包含10个元素的数组。

	{
		"status": "0",
		"citye": "changchun",
		"city": "长春",
		"citycode": "101060101",
		"aqi": "50",
		"data": [{
			"add": "长春",
			"aqi": "50",
			"pm25": "22",
			"per": "优",
			"lv": "1"
		}, {
			"add": "食品厂",
			"aqi": "54",
			"pm25": "18",
			"per": "良",
			"lv": "2"
		}, {
			"add": "客车厂",
			"aqi": "52",
			"pm25": "20",
			"per": "良",
			"lv": "2"
		}, {
			"add": "邮电学院",
			"aqi": "35",
			"pm25": "24",
			"per": "优",
			"lv": "1"
		}, {
			"add": "劳动公园",
			"aqi": "45",
			"pm25": "19",
			"per": "优",
			"lv": "1"
		}, {
			"add": "园林处",
			"aqi": "45",
			"pm25": "21",
			"per": "优",
			"lv": "1"
		}, {
			"add": "净月潭",
			"aqi": "46",
			"pm25": "30",
			"per": "优",
			"lv": "1"
		}, {
			"add": "甩湾子",
			"aqi": "51",
			"pm25": "24",
			"per": "良",
			"lv": "2"
		}, {
			"add": "经开区环卫处",
			"aqi": "48",
			"pm25": "25",
			"per": "优",
			"lv": "1"
		}, {
			"add": "高新区管委会",
			"aqi": "51",
			"pm25": "16",
			"per": "良",
			"lv": "2"
		}, {
			"add": "岱山公园",
			"aqi": "49",
			"pm25": "19",
			"per": "优",
			"lv": "1"
		}]
	}

字符串4解析函数


	int Parse_AQI_Json(void)
	{
	    QJsonParseError err_rpt;
	    QJsonDocument  root_Doc = QJsonDocument::fromJson(AQI_json, &err_rpt);//字符串格式化为JSON
	    if(err_rpt.error != QJsonParseError::NoError)
	    {
	        qDebug() << "JSON格式错误";
	        return -1;
	    }
	    else    //JSON格式正确
	    {
	        //        qDebug() << "JSON格式正确：\n" << root_Doc;
	        QJsonObject root_Obj = root_Doc.object();
	
	        QString city = root_Obj.value("city").toString();
	        QString citycode = root_Obj.value("citycode").toString();
	        QString citye = root_Obj.value("citye").toString();
	        QString status = root_Obj.value("status").toString();
	        qDebug() << city + " " + citycode + " " + citye + " " + status;
	
	        /* data键 */
	        QJsonValue data_Vaule = root_Obj.value("data");
	        if(data_Vaule.isArray())
	        {
	            for(int idx = 0; idx <= 10; idx++)
	            {
	                QJsonObject data_Obj = data_Vaule.toArray().at(idx).toObject();
	                QString add = "地址：" + data_Obj.value("add").toString();
	                QString aqi = " AQI：" + data_Obj.value("aqi").toString();
	                QString lv = " 空气质量等级：" + data_Obj.value("lv").toString();
	                QString per = " 空气质量：" + data_Obj.value("per").toString();
	                QString pm25 = " PM2.5等级：" + data_Obj.value("pm25").toString();
	                qDebug() << add + aqi + lv + per + pm25;
	            }
	        }
	    }
	    return 0;
	}

#### 示例字符串5：北京标准时间

比较简单一个JSON对象

	{
		"success": "1",
		"result": {
			"timestamp": "1542456793",
			"datetime_1": "2018-11-17 20:13:13",
			"datetime_2": "2018年11月17日 20时13分13秒",
			"week_1": "6",
			"week_2": "星期六",
			"week_3": "周六",
			"week_4": "Saturday"
		}
	}

字符串5解析函数


	int Parse_BJTime_Json(void)
	{
	    QJsonParseError err_rpt;
	    QJsonDocument  root_Doc = QJsonDocument::fromJson(bj_time_json, &err_rpt);//字符串格式化为JSON
	    if(err_rpt.error != QJsonParseError::NoError)
	    {
	        qDebug() << "JSON格式错误";
	        return -1;
	    }
	    else    //JSON格式正确
	    {
	        //        qDebug() << "JSON格式正确：\n" << root_Doc;
	        QJsonObject root_Obj = root_Doc.object();
	
	        QString success = root_Obj.value("success").toString();
	        /* result键 */
	        QJsonObject result_Obj = root_Obj.value("result").toObject();
	        QString datetime_1 = result_Obj.value("datetime_1").toString();
	        QString datetime_2 = result_Obj.value("datetime_2").toString();
	        QString timestamp = result_Obj.value("timestamp").toString();
	        QString week_1 = result_Obj.value("week_1").toString();
	        QString week_2 = result_Obj.value("week_2").toString();
	        QString week_3 = result_Obj.value("week_3").toString();
	        QString week_4 = result_Obj.value("week_4").toString();
	        qDebug() << datetime_1 << datetime_2;
	        qDebug() << week_1 << week_2 << week_3 << week_4;
	    }
	    return 0;
	}

#### 示例字符串6：全国城市油价信息

这种格式的字符串也是标准的JSON字符串，[]表示数组，这个数组内包含了5个字符串：

	["北京", "6.78", "7.21", "8.19", "6.45"]

但是这种不是，{}表示对象：

	{"北京", "6.78", "7.21", "8.19", "6.45"}

一个数组内包含5个元素，每个元素又是一个数组，每个数组又包含5个字符串，属于数组的嵌套：

	{
		"status": "0",
		"msg": "全国各省份汽柴油价格信息",
		"update": "2019-07-21",
		"data": [
			["地区", "92号", "95号", "98号", "0号柴油"],
			["北京", "6.78", "7.21", "8.19", "6.45"],
			["上海", "6.74", "7.17", "7.87", "6.39"],
			["江苏", "6.75", "7.18", "8.06", "6.37"],
			["天津", "6.77", "7.15", "8.07", "6.41"]
		],
		"About": "wcc",
		"Home": "www.wangchaochao.top"
	}

字符串6解析函数


	//解析数组嵌套的JSON字符串
	int Parse_Oil_Price_Json(void)
	{
	    QJsonParseError err_rpt;
	    QJsonDocument  root_Doc = QJsonDocument::fromJson(oil_price_json, &err_rpt);//字符串格式化为JSON
	    if(err_rpt.error != QJsonParseError::NoError)
	    {
	        qDebug() << "JSON格式错误";
	        return -1;
	    }
	    else    //JSON格式正确
	    {
	        //        qDebug() << "JSON格式正确：\n" << root_Doc;
	        QJsonObject root_Obj = root_Doc.object();
	
	        QString msg = root_Obj.value("msg").toString();
	        QString update = root_Obj.value("update").toString();
	        QString status = root_Obj.value("status").toString();
	        QString About = root_Obj.value("About").toString();
	        QString Home = root_Obj.value("Home").toString();
	        qDebug() << msg << update << status << About << Home;
	
	        /* data键解析 */
	        QJsonValue data_Value = root_Obj.value("data");
	        if(data_Value.isArray())    //数组中包含5个数组，每个数组5个字符串元素
	        {
	            for(int idx = 0; idx <= 4; idx++)
	            {
	                QJsonValue price_Obj = data_Value.toArray().at(idx);
	                if(price_Obj.isArray())
	                {
	                    QString str1 = price_Obj.toArray().at(0).toString();// 每个元素是一个字符串
	                    QString str2 = price_Obj.toArray().at(1).toString();
	                    QString str3 = price_Obj.toArray().at(2).toString();
	                    QString str4 = price_Obj.toArray().at(3).toString();
	                    QString str5 = price_Obj.toArray().at(4).toString();
	                    qDebug() << str1 << str2 << str3 << str4 << str5 ;
	                }
	            }
	        }
	    }
	    return 0;
	}


### QJson构建JSON示例

#### 1.构建一个简单的键值对JSON

	void Create_Simple_JSON(void)
	{
	
	    //创建JSON对象
	    QJsonObject root_Obj;
	    //添加键值对，值的类型自动识别，顺序不可自定义
	    root_Obj.insert("CSDN", "https://blog.csdn.net/whik1194");
	    root_Obj.insert("cnblogs", "https://home.cnblogs.com/u/whik/");
	    root_Obj.insert("Github", "https://github.com/whik/");
	    root_Obj.insert("Blog", "https://www.wangchaochao.top/");
	    root_Obj.insert("status", 1);
	    root_Obj.insert("enable", true);
	    root_Obj.insert("update_time", "20190723");
	
	    //创建Json文档
	    QJsonDocument root_Doc;
	    root_Doc.setObject(root_Obj);
	    QByteArray root_str = root_Doc.toJson(QJsonDocument::Compact);  //紧凑格式
	//    QByteArray root_str = root_Doc.toJson(QJsonDocument::Indented);   //标准JSON格式    QString strJson(root_str);
	    QString strJson(root_str);
	    qDebug() << strJson;
	}

输出结果

	{
	    "Blog": "https://www.wangchaochao.top/",
	    "CSDN": "https://blog.csdn.net/whik1194",
	    "Github": "https://github.com/whik/",
	    "cnblogs": "https://home.cnblogs.com/u/whik/",
	    "enable": true,
	    "status": 1,
	    "update_time": "20190723"
	}

#### 2.构建北京时间JSON字符串

	void Create_BJTime_JSON(void)
	{
	    QJsonObject result_Obj;
	    result_Obj.insert("timestamp", "ok");
	    result_Obj.insert("datetime_1", "2019-07-21 10:46:57");
	    result_Obj.insert("datetime_2", "2019年07月21日 10时46分57秒");
	    result_Obj.insert("week_1", "0");
	    result_Obj.insert("week_2", "星期日");
	    result_Obj.insert("week_3", "周日");
	    result_Obj.insert("week_4", "Sunday");
	
	    QJsonObject root_Obj;
	    //添加键值对，值的类型自动识别，顺序不可自定义
	    root_Obj.insert("status", "success");
	    root_Obj.insert("Blog", "www.wangchaochao.top");
	    root_Obj.insert("result", result_Obj);
	
	    //创建Json文档
	    QJsonDocument root_Doc;
	    root_Doc.setObject(root_Obj);
	    QByteArray root_str = root_Doc.toJson(QJsonDocument::Compact);  //紧凑格式
	//    QByteArray root_str = root_Doc.toJson(QJsonDocument::Indented);   //标准JSON格式
	//    qDebug() << root_str; //中文输出乱码
	    QString strJson(root_str);
	    qDebug() << strJson;
	}

输出结果

	{
	    "status": "success",
	    "result": {
	        "timestamp": "ok",
	        "datetime_1": "2019-07-21 10:46:57",
	        "datetime_2": "2019年07月21日 10时46分57秒",
	        "week_1": "0",
	        "week_2": "星期日",
	        "week_3": "周日",
	        "week_4": "Sunday"
	    },
	    "Blog": "www.wangchaochao.top"
	}



#### 3.构建字符串数组


	void Create_Array_Str_JSON(void)
	{
	    QJsonArray root_Arr;
	
	    root_Arr.insert(0, "Sunday");
	    root_Arr.insert(1, "Monday");
	    root_Arr.insert(2, "Tuesday");
	    root_Arr.insert(3, "Wednesday");
	    root_Arr.insert(4, "Thursday");
	    root_Arr.insert(5, "Friday");
	    root_Arr.insert(6, "Saturday");
	
	    //创建Json文档
	    QJsonDocument root_Doc;
	    root_Doc.setArray(root_Arr);
	    QByteArray root_str = root_Doc.toJson(QJsonDocument::Compact);  //紧凑格式
	//    QByteArray root_str = root_Doc.toJson(QJsonDocument::Indented);   //标准JSON格式
	//    qDebug() << root_str; //中文输出乱码
	    QString strJson(root_str);
	    qDebug() << strJson;
	}


输出结果

	["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]


#### 4.构建数组JSON


	void Create_Array_JSON(void)
	{
	    QJsonObject day0_Obj;
	    QJsonObject day1_Obj;
	    QJsonObject day2_Obj;
	
	    day0_Obj.insert("date", "2019-07-21");
	    day0_Obj.insert("cond_txt", "多云");
	    day0_Obj.insert("cond_code", "101");
	    day0_Obj.insert("hum", "23");
	    day0_Obj.insert("tmp_H", "31");
	    day0_Obj.insert("tmp_L", "25");
	
	    day1_Obj.insert("date", "2019-07-21");
	    day1_Obj.insert("cond_txt", "阵雨");
	    day1_Obj.insert("cond_code", "107");
	    day1_Obj.insert("hum", "44");
	    day1_Obj.insert("tmp_H", "30");
	    day1_Obj.insert("tmp_L", "26");
	
	    day2_Obj.insert("date", "2019-07-22");
	    day2_Obj.insert("cond_txt", "晴");
	    day2_Obj.insert("cond_code", "100");
	    day2_Obj.insert("hum", "20");
	    day2_Obj.insert("tmp_H", "33");
	    day2_Obj.insert("tmp_L", "26");
	
	    QJsonArray weather_Arr;
	    weather_Arr.insert(0, day0_Obj);
	    weather_Arr.insert(1, day1_Obj);
	    weather_Arr.insert(2, day2_Obj);
	
	    QJsonObject root_Obj;
	    root_Obj.insert("status", "ok");
	    root_Obj.insert("update", "2019-07-21 11:00");
	    root_Obj.insert("Blog", "www.wangchaochao.top");
	    root_Obj.insert("weather", weather_Arr);    //数组作为weather键的值
	
	    //创建Json文档
	    QJsonDocument root_Doc;
	    root_Doc.setObject(root_Obj);
	    QByteArray root_str = root_Doc.toJson(QJsonDocument::Compact);  //紧凑格式
	//    QByteArray root_str = root_Doc.toJson(QJsonDocument::Indented);   //标准JSON格式
	//    qDebug() << root_str; //中文输出乱码
	    QString strJson(root_str);
	    qDebug() << strJson;
	}


输出结果
	
	{
	    "status": "ok",
	    "weather": [{
	        "date": "2019-07-21",
	        "cond_txt": "多云",
	        "cond_code": "101",
	        "hum": "23",
	        "tmp_H": "31",
	        "tmp_L": "25"
	    }, {
	        "date": "2019-07-22",
	        "cond_txt": "晴",
	        "cond_code": "100",
	        "hum": "20",
	        "tmp_H": "33",
	        "tmp_L": "26"
	    }, {
	        "date": "2019-07-23",
	        "cond_txt": "阵雨",
	        "cond_code": "107",
	        "hum": "45",
	        "tmp_H": "32",
	        "tmp_L": "25"
	    }],
	    "update": "2019-07-21 11:00",
	    "Blog": "www.wangchaochao.top"
	}

#### 5.构建数组嵌套的JSON字符串


	void Create_Array_Nest_JSON(void)
	{
	    QJsonArray table_Arr = {"地区", "92号汽油", "95号汽油", "98号汽油", "0号柴油"};
	    QJsonArray bj_Arr = {"北京", "6.78", "7.21", "8.19", "6.45"};
	    QJsonArray sh_Arr = {"上海", "6.74", "7.17", "7.87", "6.39"};
	    QJsonArray js_Arr = {"江苏", "6.75", "7.18", "8.06", "6.37"};
	    QJsonArray tj_Arr = {"天津", "6.77", "7.15", "8.07", "6.41"};
	
	    QJsonArray data_Arr;    //数组内嵌套了5个数组
	    data_Arr.insert(0, table_Arr);
	    data_Arr.insert(1, bj_Arr);
	    data_Arr.insert(2, sh_Arr);
	    data_Arr.insert(3, js_Arr);
	    data_Arr.insert(4, tj_Arr);
	
	    QJsonObject root_Obj;
	
	    root_Obj.insert("status", "ok");
	    root_Obj.insert("msg", "全国各省份汽柴油价格信息");
	    root_Obj.insert("update", "2019-07-21");
	    root_Obj.insert("About", "wcc");
	    root_Obj.insert("Blog", "www.wangchaochao.top");
	    root_Obj.insert("data", data_Arr);  //数组作为键的值
	
	    //创建Json文档
	    QJsonDocument root_Doc;
	    root_Doc.setObject(root_Obj);
	    QByteArray root_str = root_Doc.toJson(QJsonDocument::Compact);  //紧凑格式
	//    QByteArray root_str = root_Doc.toJson(QJsonDocument::Indented);   //标准JSON格式
	//    qDebug() << root_str; //中文输出乱码
	    QString strJson(root_str);
	    qDebug() << strJson;
	}

输出结果
	
	{
	    "status": "ok",
	    "msg": "全国各省份汽柴油价格信息",
	    "update": "2019-07-21",
	    "data": [
	        ["地区", "92号汽油", "95号汽油", "98号汽油", "0号柴油"],
	        ["北京", "6.78", "7.21", "8.19", "6.45"],
	        ["上海", "6.74", "7.17", "7.87", "6.39"],
	        ["江苏", "6.75", "7.18", "8.06", "6.37"],
	        ["天津", "6.77", "7.15", "8.07", "6.41"]
	    ],
	    "About": "wcc",
	    "Blog": "www.wangchaochao.top"
	}


### 参考资料

- JSON官方网站：[json](http://www.json.org/json-zh.html)

### 历史精选

- [使用cJSON库解析和构建JSON字符串](http://www.wangchaochao.top/2019/07/21/cJSON-Demo/)
- [【2019北京国际消费电子博览会】参观总结](http://www.wangchaochao.top/2019/06/30/Beijing-CEE/)
- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)
- [Qt小项目之串口助手控制LED](http://www.wangchaochao.top/2019/03/03/Qt-UART-Ctrl-LED/)
- [国产处理器的逆袭机会——RISC-V](http://www.wangchaochao.top/2019/04/27/ESBF/)
- [JSON简介](http://www.wangchaochao.top/2018/11/18/cJSON/)

--------

欢迎关注我的个人博客：www.wangchaochao.top

或微信扫码关注我的公众号