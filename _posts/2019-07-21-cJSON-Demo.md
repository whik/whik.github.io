---
layout:     post
title:    使用cJSON库解析和构建JSON字符串
subtitle:	 C语言
date:       2019-07-21 23:55:40 +0800
author:     Wang Chao
header-img: img/Code1.jpg
catalog:    true
tag:
    - C语言
    - JSON
---

### 前言

其实之前的两篇博文已经介绍了json格式和如何使用cJSON库来解析JSON：

- [使用cJSON库解析JSON](http://www.wangchaochao.top/2018/12/04/Parse-JSON/)
- [JSON简介](http://www.wangchaochao.top/2018/11/18/cJSON/)

当时在MCU平台上使用时，会出现时间长了死机的情况，在调用cJSON_Print输出格式化后的JSON数据之后，

        LOG("JSON数据:\n%s\n", cJSON_Print(root));	

要使用cJSON_Delete释放内存，否则会导致内存泄漏。

	cJSON_Delete(root);     //调用cJSON_Print时才需要

这一点在嵌入式开发平台要格外注意。

解析和构建JSON的示例程序，我都已经上传到代码托管平台上，示例工程基于CodeBlocks开发环境。

Github仓库地址：[![](https://img.shields.io/badge/Github-cJSON_Demo-orange.svg)](https://github.com/whik/cJSON_Demo)

Gitee仓库地址：[![](https://img.shields.io/badge/Gitee-cJSON_Demo-orange.svg)](https://gitee.com/whik/cJSON_Demo)

或者通过下面的命令clone到本地：

Github：

	git clone https://gitee.com/whik/cJSON_Demo.git

Gitee：

	git clone https://github.com/whik/cJSON_Demo.git


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

### JSON的构建

cJSON是一个基于C语言的JSON解析库，这个库非常简单，只有`cJSON.c`和`cJSON.h`两个文件，支持JSON的解析和构建，需要调用时，只需要`#include "cJSON.h"`就可以使用了。

由于JSON的解析之前已经介绍过了：[使用cJSON库解析JSON](http://www.wangchaochao.top/2018/12/04/Parse-JSON/)，所以本篇博文主要介绍使用cJSON来构建JSON，强大的cJSON库在构建JSON上也是非常的简单。

#### 1.一个简单的JSON键值对构建

构建函数：

	void Create_Simple_JSON(void)
	{
	    cJSON *root;
	    root = cJSON_CreateObject();//创建一个json对象
	    cJSON_AddItemToObject(root, "CSDN", cJSON_CreateString("https://blog.csdn.net/whik1194"));
	    cJSON_AddItemToObject(root, "cnblogs", cJSON_CreateString("https://home.cnblogs.com/u/whik/"));
	    cJSON_AddItemToObject(root, "Github", cJSON_CreateString("https://github.com/whik/"));
	    cJSON_AddStringToObject(root, "Blog", "http://www.wangchaochao.top/");
	
	    printf("构建的JSON:\n%s\n", cJSON_Print(root));
	    cJSON_Delete(root);
	}

输出结果：

	{
	    "CSDN": "https://blog.csdn.net/whik1194",
	    "cnblogs": "https://home.cnblogs.com/u/whik/",
	    "Github": "https://github.com/whik/",
	    "Blog": "http://www.wangchaochao.top/"
	}

#### 2.键的值是一个JSON对象
	
构建函数：

	void Create_BJTime_JSON(void)
	{

	    cJSON *root;
	    cJSON *result;
	
	    root = cJSON_CreateObject();//创建一个json对象
	
	    result = cJSON_CreateObject();
	    //result构建
	
	    cJSON_AddItemToObject(result, "timestamp", cJSON_CreateString("ok"));
	//等效于下面
	//    cJSON_AddStringToObject(result, "timestamp", "ok");
	
	    cJSON_AddItemToObject(result, "datetime_1", cJSON_CreateString("2019-07-21 10:46:57"));
	    cJSON_AddItemToObject(result, "datetime_2", cJSON_CreateString("2019年07月21日 10时46分57秒"));
	    cJSON_AddItemToObject(result, "week_1", cJSON_CreateString("0"));
	    cJSON_AddItemToObject(result, "week_2", cJSON_CreateString("星期日"));
	    cJSON_AddItemToObject(result, "week_3", cJSON_CreateString("周日"));
	    cJSON_AddItemToObject(result, "week_4", cJSON_CreateString("Sunday"));
	
	    //等效于cJSON_AddNumberToObject(root, "ok", 1);
	
	    cJSON_AddItemToObject(root, "status", cJSON_CreateString("success"));
	    cJSON_AddItemToObject(root, "result", result);
	    cJSON_AddStringToObject(root, "Blog", "www.wangchaochao.top");
	
	    printf("构建的JSON:\n%s\n", cJSON_Print(root));
	    cJSON_Delete(root);
	}

输出结果：

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

#### 3.JSON数组，元素是字符串
	
构建函数：

	void Create_Array_Str_JSON(void)
	{
	    cJSON *root;
	    const char *strings[7]={"Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"};
	
		root=cJSON_CreateStringArray(strings,7);
	
		printf("%s\n",cJSON_Print(root));
		cJSON_Delete(root);
	}

输出结果：

	["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]

#### 4.键的值是一个数组，数组包含多个对象元素

构建函数：

	void Create_Array_JSON(void)
	{
	    cJSON *root;
	    cJSON *forceast;
	    cJSON *day1, *day2, *day3;  //数组
	
	    day1 = cJSON_CreateObject();
	    day2 = cJSON_CreateObject();
	    day3 = cJSON_CreateObject();
	
	    cJSON_AddStringToObject(day1, "date", "2019-07-21");    //日期
	    cJSON_AddStringToObject(day1, "cond_txt", "多云");      //天气状况
	    cJSON_AddStringToObject(day1, "cond_code", "101");      //天气代码
	    cJSON_AddStringToObject(day1, "hum", "23");             //湿度
	    cJSON_AddStringToObject(day1, "tmp_H", "31");           //最高温度
	    cJSON_AddStringToObject(day1, "tmp_L", "25");           //最低温度
	
	    cJSON_AddStringToObject(day2, "date", "2019-07-22");
	    cJSON_AddStringToObject(day2, "cond_txt", "晴");
	    cJSON_AddStringToObject(day2, "cond_code", "100");
	    cJSON_AddStringToObject(day2, "hum", "20");
	    cJSON_AddStringToObject(day2, "tmp_H", "33");
	    cJSON_AddStringToObject(day2, "tmp_L", "26");
	
	    cJSON_AddStringToObject(day3, "date", "2019-07-23");
	    cJSON_AddStringToObject(day3, "cond_txt", "阵雨");
	    cJSON_AddStringToObject(day3, "cond_code", "107");
	    cJSON_AddStringToObject(day3, "hum", "45");
	    cJSON_AddStringToObject(day3, "tmp_H", "32");
	    cJSON_AddStringToObject(day3, "tmp_L", "25");
	
	    forceast = cJSON_CreateArray();
	    //注意顺序，索引依次递增
	    cJSON_AddItemToArray(forceast, day1);   //元素0
	    cJSON_AddItemToArray(forceast, day2);   //元素1
	    cJSON_AddItemToArray(forceast, day3);   //元素2
	
	    root = cJSON_CreateObject();    //创建一个json对象
	
	    cJSON_AddStringToObject(root, "status", "ok");
	    cJSON_AddItemToObject(root, "weather", forceast);
	    cJSON_AddStringToObject(root, "update", "2019-07-21 11:00");
	    cJSON_AddStringToObject(root, "Blog", "www.wangchaochao.top");
	   //等效于:cJSON_AddItemToObject(root, "update", cJSON_CreateString("2019-07-21 11:00");
	
	    printf("构建的JSON:\n%s\n", cJSON_Print(root));
	    cJSON_Delete(root);
	}

输出结果：

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

#### 5.数组内嵌套了5个数组，每个数组内有5个字符串元素

构建函数：

	void Create_Array_Nest_JSON(void)
	{
	    struct oil_stu{
	        char *city;          //城市名称
	        char *oil_92_price;  //92号汽油价格
	        char *oil_95_price;
	        char *oil_98_price;
	        char *oil_0_price;
	    };
	
	    cJSON *root;
	    cJSON *data;  //包含多个数组
	    cJSON *table, *data_bj, *data_sh, *data_js, *data_tj;
	
	    const char *bj_str[5] = {"北京", "6.78", "7.21", "8.19", "6.45"};
	    const char *sh_str[5] = {"上海", "6.74", "7.17", "7.87", "6.39"};
	    const char *js_str[5] = {"江苏", "6.75", "7.18", "8.06", "6.37"};
	    const char *tj_str[5] = {"天津", "6.77", "7.15", "8.07", "6.41"};
	    const char *talbe_str[5] = {"地区", "92号汽油", "95号汽油", "98号汽油", "0号柴油"};
	
	    data_bj = cJSON_CreateStringArray(bj_str, 5);   //只包含5个字符串的数组
	    data_sh = cJSON_CreateStringArray(sh_str, 5);
	    data_js = cJSON_CreateStringArray(js_str, 5);
	    data_tj = cJSON_CreateStringArray(tj_str, 5);
	    table = cJSON_CreateStringArray(talbe_str, 5);
	
	    data = cJSON_CreateArray();
	    cJSON_AddItemToArray(data, table);
	    cJSON_AddItemToArray(data, data_bj);
	    cJSON_AddItemToArray(data, data_sh);
	    cJSON_AddItemToArray(data, data_js);
	    cJSON_AddItemToArray(data, data_tj);
	
	    root = cJSON_CreateObject();
	
	    cJSON_AddStringToObject(root, "status", "ok");
	    cJSON_AddStringToObject(root, "msg", "2019-07-21 11:00");
	    cJSON_AddStringToObject(root, "update", "2019-07-21 11:00");
	    cJSON_AddItemToObject(root, "data", data);
	    cJSON_AddStringToObject(root, "About", "wcc");
	    cJSON_AddStringToObject(root, "Blog", "www.wangchaochao.top");
	
	    printf("构建的JSON:\n%s\n", cJSON_Print(root));
	    cJSON_Delete(root);
	}

输出结果：
	
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

- cJSON库源码：[cJSON download](https://sourceforge.net/projects/cjson/)
- JSON官方网站：[json](http://www.json.org/json-zh.html)

### 历史精选

- [【2019北京国际消费电子博览会】参观总结](http://www.wangchaochao.top/2019/06/30/Beijing-CEE/)
- [Qt实现软件自动更新的一种简单方法](http://www.wangchaochao.top/2019/03/31/Qt-Update/)
- [Qt小项目之串口助手控制LED](http://www.wangchaochao.top/2019/03/03/Qt-UART-Ctrl-LED/)
- [国产处理器的逆袭机会——RISC-V](http://www.wangchaochao.top/2019/04/27/ESBF/)
- [JSON简介](http://www.wangchaochao.top/2018/11/18/cJSON/)

--------

欢迎关注我的个人博客：[![](https://img.shields.io/badge/MyBlog-www.wangchaochao.top-orange.svg)](http://www.wangchaochao.top/)

或微信扫码关注我的公众号