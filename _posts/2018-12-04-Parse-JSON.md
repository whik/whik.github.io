---
layout:     post
title:     使用cJSON库解析JSON
subtitle:  JSON的解析
date:       2018-12-04 21:30:40 +0800
author:     Wang Chao
header-img: img/json_org.jpg
catalog:    true
tag:
    - C语言
---

## cJSON库的下载

cJSON是一个基于C的JSON解析库，这个库非常简单，只有cJSON.c和cJSON.h两个文件，支持JSON的解析和封装，需要调用时，只需要`#include "cJSON.h"`就可以使用了，

- 库源码下载地址：[cJSON download](https://sourceforge.net/projects/cjson/)

- JSON官方网站：[json](http://www.json.org/json-zh.html)

## 只包含键值对的JSON字符串解析

JSON字符串：

	{
		"name": "Andy",      //键值对1
		"age": 20              //键值对2
	}

![](http://www.json.org/object.gif)

这个JSON对象只有两个键值对，键name对应字符串Andy，键age对应数字20。

	void Parse_Str1(void)
	{
	    char str1[] = "{\"name\":\"Andy\",\"age\":20}";
	    cJSON *str1_json, *str1_name, *str1_age;
	    printf("str1:%s\n\n",str1);
	    str1_json = cJSON_Parse(str1);   //创建JSON解析对象，返回JSON格式是否正确
	    if (!str1_json)
	    {
	        printf("JSON格式错误:%s\n\n", cJSON_GetErrorPtr()); //输出json格式错误信息
	    }
	    else
	    {
	        printf("JSON格式正确:\n%s\n\n",cJSON_Print(str1_json) );
	        str1_name = cJSON_GetObjectItem(str1_json, "name"); //获取name键对应的值的信息
	        if (str1_name->type == cJSON_String)
	        {
	            printf("姓名:%s\r\n", str1_name->valuestring);
	        }
	        str1_age = cJSON_GetObjectItem(str1_json, "age");   //获取age键对应的值的信息
	        if(str1_age->type==cJSON_Number)
	        {
	            printf("年龄:%d\r\n", str1_age->valueint);
	        }
	        cJSON_Delete(str1_json);//释放内存
	    }
	}

运行结果：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/json1.jpg)

## 包含数组的JSON字符串解析

JSON字符串：

    {
    	"location": [{
    			"name": "Faye",
    			"address": "北京"
    		},
    		{
    			"name": "Andy",
    			"address": "香港"
    		}
    	],
    	"time": "2018-11-17"
    }

![](http://www.json.org/array.gif)

解析函数：

	void Parse_Str2(void)
	{
	
	    char str2[] = "{\"location\":[{\"name\":\"Faye\",\"address\":\"北京\"},{\"name\":\"Andy\",\"address\":\"香港\"}],\"time\":\"2018-11-17\"}";
	
	    cJSON *root = 0;
	    cJSON *loc_json = 0;
	    cJSON *name1_json,*name2_json;
	    char *time_str, *str_tmp;
	
	    root = cJSON_Parse(str2);
	    if(!root)
	        printf("str2 JSON格式错误:%s \r\n", cJSON_GetErrorPtr());
	    else
	    {
	        printf("str2 JSON格式正确:\n%s\n",cJSON_Print(root));
	        time_str = cJSON_GetObjectItem(root,"time")->valuestring;//time键值对
	        printf("time:%s\n", time_str);
	
	        loc_json = cJSON_GetObjectItem(root,"location");
	        if(loc_json)
	        {
	            name1_json = cJSON_GetArrayItem(loc_json,0);        //数组第0个元素
	            str_tmp = cJSON_GetObjectItem(name1_json, "name")->valuestring;//name键对应的值
	            printf("name1 is : %s \r\n", str_tmp);
	            str_tmp = cJSON_GetObjectItem(name1_json, "address")->valuestring;//addr1键对应的值
	            printf("addr1 is : %s \r\n", str_tmp);
	
	            name2_json = cJSON_GetArrayItem(loc_json,1);       //数组第1个元素
	            str_tmp = cJSON_GetObjectItem(name2_json, "name")->valuestring;
	            printf("name2 is : %s \r\n", str_tmp);
	            str_tmp = cJSON_GetObjectItem(name2_json, "address")->valuestring;
	            printf("addr2 is : %s \r\n", str_tmp);
	        }
	    }
	    cJSON_Delete(loc_json);
	}

运行结果：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/json2.jpg)

## 北京时间JSON数据解析

api地址：

`http://api.k780.com:88/?app=life.time&appkey=10003&sign=b59bc3ef6191eb9f747dd4e83c99f2a4&format=json`

JSON字符串：

	{
		"success": "1",
		"result": {
			"timestamp": "1543922613",
			"datetime_1": "2018-12-04 19:23:33",
			"datetime_2": "2018年12月04日 19时23分33秒",
			"week_1": "2",
			"week_2": "星期二",
			"week_3": "周二",
			"week_4": "Tuesday"
		}
	}								

解析函数：
	
	void Parse_BJ_Time(void)
	{
	    char bj_time_str[] = "{\"success\":\"1\",\"result\":{\"timestamp\":\"1542456793\",\"datetime_1\":\"2018-11-17 20:13:13\",\"datetime_2\":\"2018年11月17日 20时13分13秒\",\"week_1\":\"6\",\"week_2\":\"星期六\",\"week_3\":\"周六\",\"week_4\":\"Saturday\"}}";

	    cJSON *root;
	    cJSON *result_json;
	    char *datetime, *week;
	
	    root = cJSON_Parse(bj_time_str);
	    if(root)
	    {
	        printf("json格式正确:\n%s\n\n", cJSON_Print(root));
	        result_json =  cJSON_GetObjectItem(root, "result");  //获取result键对应的值
	        if(result_json)
	        {
	            datetime = cJSON_GetObjectItem(result_json, "datetime_2")->valuestring;
	            printf("北京时间: %s \r\n", datetime);
	            week = cJSON_GetObjectItem(result_json, "week_2")->valuestring;
	            printf("星期: %s \r\n", week);
	        }
	    }
	    cJSON_Delete(root);
	    cJSON_Delete(result_json);
	}

运行结果：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/json3.jpg)

## 心知天气JSON数据解析

JSON字符串：

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
				"date": "2018-11-18",
				"text_day": "多云",
				"code_day": "4",
				"text_night": "多云",
				"code_night": "4",
				"high": "26",
				"low": "20",
				"precip": "",
				"wind_direction": "无持续风向",
				"wind_direction_degree": "",
				"wind_speed": "10",
				"wind_scale": "2"
			}, {
				"date": "2018-11-19",
				"text_day": "小雨",
				"code_day": "13",
				"text_night": "小雨",
				"code_night": "13",
				"high": "25",
				"low": "20",
				"precip": "",
				"wind_direction": "无持续风向",
				"wind_direction_degree": "",
				"wind_speed": "10",
				"wind_scale": "2"
			}, {
				"date": "2018-11-20",
				"text_day": "小雨",
				"code_day": "13",
				"text_night": "小雨",
				"code_night": "13",
				"high": "25",
				"low": "21",
				"precip": "",
				"wind_direction": "无持续风向",
				"wind_direction_degree": "",
				"wind_speed": "10",
				"wind_scale": "2"
			}],
			"last_update": "2018-11-18T11:00:00+08:00"
		}]
	}

解析函数：

	void parse_seniverse_weather(void)
	{
	    char weather_str[] =
	        "{\"results\":[{\"location\":{\"id\":\"WS10730EM8EV\",\"name\":\"深圳\",\"country\":\"CN\",\"path\":\"深圳,深圳,广东,中国\",\"timezone\":\"Asia/Shanghai\",\"timezone_offset\":\"+08:00\"},\"daily\":[{\"date\":\"2018-11-18\",\"text_day\":\"多云\",\"code_day\":\"4\",\"text_night\":\"多云\",\"code_night\":\"4\",\"high\":\"26\",\"low\":\"20\",\"precip\":\"\",\"wind_direction\":\"无持续风向\",\"wind_direction_degree\":\"\",\"wind_speed\":\"10\",\"wind_scale\":\"2\"},{\"date\":\"2018-11-19\",\"text_day\":\"小雨\",\"code_day\":\"13\",\"text_night\":\"小雨\",\"code_night\":\"13\",\"high\":\"25\",\"low\":\"20\",\"precip\":\"\",\"wind_direction\":\"无持续风向\",\"wind_direction_degree\":\"\",\"wind_speed\":\"10\",\"wind_scale\":\"2\"},{\"date\":\"2018-11-20\",\"text_day\":\"小雨\",\"code_day\":\"13\",\"text_night\":\"小雨\",\"code_night\":\"13\",\"high\":\"25\",\"low\":\"21\",\"precip\":\"\",\"wind_direction\":\"无持续风向\",\"wind_direction_degree\":\"\",\"wind_speed\":\"10\",\"wind_scale\":\"2\"}],\"last_update\":\"2018-11-18T11:00:00+08:00\"}]}";
	    cJSON *root;
	    cJSON *results;
	    cJSON *last_update;
	    cJSON *loc_json, *daily_json;
	    cJSON *forecast_json;
	    char *loc_tmp, *weather_tmp, *update_tmp;
	    int i = 0;
	
	    root = cJSON_Parse((const char*)weather_str);
	    if(root)
	    {
	//        printf("JSON格式正确:\n%s\n\n",cJSON_Print(root));    //输出json字符串
	        results = cJSON_GetObjectItem(root, "results");
	        results = cJSON_GetArrayItem(results,0);
	        if(results)
	        {
	            loc_json = cJSON_GetObjectItem(results, "location");   //得到location键对应的值，是一个对象
	
	            loc_tmp = cJSON_GetObjectItem(loc_json, "id") -> valuestring;
	            printf("城市ID:%s\n",loc_tmp);
	            loc_tmp = cJSON_GetObjectItem(loc_json, "name") -> valuestring;
	            printf("城市名称:%s\n",loc_tmp);
	            loc_tmp = cJSON_GetObjectItem(loc_json, "timezone") -> valuestring;
	            printf("城市时区:%s\n\n",loc_tmp);
	
	            daily_json = cJSON_GetObjectItem(results, "daily");
	            if(daily_json)
	            {
	                for(i = 0; i < 3; i++)
	                {
	                    forecast_json = cJSON_GetArrayItem(daily_json, i);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "date") -> valuestring;
	                    printf("日期:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "code_day") -> valuestring;
	                    printf("白天天气代码:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "code_night") -> valuestring;
	                    printf("晚上天气代码:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "high") -> valuestring;
	                    printf("最高温度:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "low") -> valuestring;
	                    printf("最低温度:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "wind_direction_degree") -> valuestring;
	                    printf("风向角度:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "wind_scale") -> valuestring;
	                    printf("风力:%s\r\n\n", weather_tmp);
	                }
	            }
	            else
	                printf("daily json格式错误\r\n");
	            last_update = cJSON_GetObjectItem(results, "last_update");
	            update_tmp = last_update->valuestring;
	            if(last_update)
	            {
	                printf("更新时间:%s\r\n", update_tmp);
	            }
	        }
	        else
	        {
	            printf("results格式错误:%s\r\n", cJSON_GetErrorPtr());
	        }
	    }
	    else
	    {
	        printf("JSON格式错误\r\n");
	    }
	    cJSON_Delete(root);
	    cJSON_Delete(results);
	}

运行结果：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/json4.jpg)

## 和风天气数据解析

JSON字符串：

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
				"loc": "2018-11-21 21:45",
				"utc": "2018-11-21 13:45"
			},
			"status": "ok",
			"daily_forecast": [{
				"cond_code_d": "100",
				"cond_code_n": "100",
				"cond_txt_d": "晴",
				"cond_txt_n": "晴",
				"date": "2018-11-21",
				"hum": "21",
				"mr": "16:02",
				"ms": "04:27",
				"pcpn": "0.0",
				"pop": "0",
				"pres": "1030",
				"sr": "07:08",
				"ss": "16:53",
				"tmp_max": "9",
				"tmp_min": "-3",
				"uv_index": "5",
				"vis": "10",
				"wind_deg": "323",
				"wind_dir": "西北风",
				"wind_sc": "1-2",
				"wind_spd": "4"
			}, {
				"cond_code_d": "100",
				"cond_code_n": "101",
				"cond_txt_d": "晴",
				"cond_txt_n": "多云",
				"date": "2018-11-22",
				"hum": "21",
				"mr": "16:36",
				"ms": "05:33",
				"pcpn": "0.0",
				"pop": "0",
				"pres": "1030",
				"sr": "07:09",
				"ss": "16:52",
				"tmp_max": "8",
				"tmp_min": "-4",
				"uv_index": "3",
				"vis": "20",
				"wind_deg": "35",
				"wind_dir": "东北风",
				"wind_sc": "1-2",
				"wind_spd": "5"
			}, {
				"cond_code_d": "101",
				"cond_code_n": "100",
				"cond_txt_d": "多云",
				"cond_txt_n": "晴",
				"date": "2018-11-23",
				"hum": "23",
				"mr": "17:15",
				"ms": "06:41",
				"pcpn": "0.0",
				"pop": "16",
				"pres": "1024",
				"sr": "07:10",
				"ss": "16:52",
				"tmp_max": "7",
				"tmp_min": "-2",
				"uv_index": "2",
				"vis": "20",
				"wind_deg": "305",
				"wind_dir": "西北风",
				"wind_sc": "1-2",
				"wind_spd": "3"
			}]
		}]
	}

解析函数：
	
	//解析和风天气，格式和心知天气非常像
	void parse_heweather(void)
	{
	    char heweather_str[] = "{\"HeWeather6\":[{\"basic\":{\"cid\":\"CN101010700\",\"location\":\"昌平\",\"parent_city\":\"北京\",\"admin_area\":\"北京\",\"cnty\":\"中国\",\"lat\":\"40.21808624\",\"lon\":\"116.23590851\",\"tz\":\"+8.00\"},\"update\":{\"loc\":\"2018-11-21 21:45\",\"utc\":\"2018-11-21 13:45\"},\"status\":\"ok\",\"daily_forecast\":[{\"cond_code_d\":\"100\",\"cond_code_n\":\"100\",\"cond_txt_d\":\"晴\",\"cond_txt_n\":\"晴\",\"date\":\"2018-11-21\",\"hum\":\"21\",\"mr\":\"16:02\",\"ms\":\"04:27\",\"pcpn\":\"0.0\",\"pop\":\"0\",\"pres\":\"1030\",\"sr\":\"07:08\",\"ss\":\"16:53\",\"tmp_max\":\"9\",\"tmp_min\":\"-3\",\"uv_index\":\"5\",\"vis\":\"10\",\"wind_deg\":\"323\",\"wind_dir\":\"西北风\",\"wind_sc\":\"1-2\",\"wind_spd\":\"4\"},{\"cond_code_d\":\"100\",\"cond_code_n\":\"101\",\"cond_txt_d\":\"晴\",\"cond_txt_n\":\"多云\",\"date\":\"2018-11-22\",\"hum\":\"21\",\"mr\":\"16:36\",\"ms\":\"05:33\",\"pcpn\":\"0.0\",\"pop\":\"0\",\"pres\":\"1030\",\"sr\":\"07:09\",\"ss\":\"16:52\",\"tmp_max\":\"8\",\"tmp_min\":\"-4\",\"uv_index\":\"3\",\"vis\":\"20\",\"wind_deg\":\"35\",\"wind_dir\":\"东北风\",\"wind_sc\":\"1-2\",\"wind_spd\":\"5\"},{\"cond_code_d\":\"101\",\"cond_code_n\":\"100\",\"cond_txt_d\":\"多云\",\"cond_txt_n\":\"晴\",\"date\":\"2018-11-23\",\"hum\":\"23\",\"mr\":\"17:15\",\"ms\":\"06:41\",\"pcpn\":\"0.0\",\"pop\":\"16\",\"pres\":\"1024\",\"sr\":\"07:10\",\"ss\":\"16:52\",\"tmp_max\":\"7\",\"tmp_min\":\"-2\",\"uv_index\":\"2\",\"vis\":\"20\",\"wind_deg\":\"305\",\"wind_dir\":\"西北风\",\"wind_sc\":\"1-2\",\"wind_spd\":\"3\"}]}]}";
	
	    cJSON *root;
	    cJSON *results;
	    cJSON *basic_json, *update_json, *forecast_json;
	    cJSON *daily_json;
	
	    int i = 0;
	    char *basic_tmp, *update_tmp, *status_tmp, *weather_tmp;
	    root = cJSON_Parse(heweather_str);
	    if(root)
	    {
	        results = cJSON_GetObjectItem(root, "HeWeather6");      //HeWeather键对应的值，是一个数组
	        results = cJSON_GetArrayItem(results,0);
	        if(results)
	        {
	            basic_json = cJSON_GetObjectItem(results, "basic");
	            if(basic_json)
	            {
	                basic_tmp = cJSON_GetObjectItem(basic_json, "cid") -> valuestring;
	                printf("城市ID:%s\n",basic_tmp);
	                basic_tmp = cJSON_GetObjectItem(basic_json, "location") -> valuestring;
	                printf("县级市:%s\n",basic_tmp);
	                basic_tmp = cJSON_GetObjectItem(basic_json, "parent_city") -> valuestring;
	                printf("地级市:%s\n",basic_tmp);
	                basic_tmp = cJSON_GetObjectItem(basic_json, "admin_area") -> valuestring;
	                printf("所属省:%s\n",basic_tmp);
	                basic_tmp = cJSON_GetObjectItem(basic_json, "lat") -> valuestring;
	                printf("纬度:%s\n",basic_tmp);
	                basic_tmp = cJSON_GetObjectItem(basic_json, "lon") -> valuestring;
	                printf("经度:%s\n\n",basic_tmp);
	            }
	            update_json = cJSON_GetObjectItem(results, "update");
	            if(update_json)
	            {
	                update_tmp = cJSON_GetObjectItem(update_json, "loc") -> valuestring;
	                printf("更新时间:%s(所在地时间)\n", update_tmp);
	                update_tmp = cJSON_GetObjectItem(update_json, "utc") -> valuestring;
	                printf("更新时间:%s(世界时间)\n\n", update_tmp);
	            }
	            status_tmp = cJSON_GetObjectItem(results, "status") -> valuestring;
	            printf("解析状态:%s\n\n", status_tmp);
	            daily_json = cJSON_GetObjectItem(results, "daily_forecast");
	            if(daily_json)
	            {
	                for(i = 0; i < 3; i++)
	                {
	                    forecast_json = cJSON_GetArrayItem(daily_json, i);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "date") -> valuestring;
	                    printf("日期:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "cond_txt_d") -> valuestring;
	                    printf("白天天气:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "cond_txt_n") -> valuestring;
	                    printf("晚上天气:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "tmp_max") -> valuestring;
	                    printf("最高温度:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "tmp_min") -> valuestring;
	                    printf("最低温度:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "wind_deg") -> valuestring;
	                    printf("风向角度:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "wind_dir") -> valuestring;
	                    printf("风向:%s\r\n", weather_tmp);
	                    weather_tmp = cJSON_GetObjectItem(forecast_json, "wind_sc") -> valuestring;
	                    printf("风力:%s\r\n\n", weather_tmp);
	                }
	            }
	        }
	    }
	    cJSON_Delete(root);
	    cJSON_Delete(results);
	    cJSON_Delete(basic_json);
	    cJSON_Delete(update_json);
	    cJSON_Delete(forecast_json);
	    cJSON_Delete(daily_json);
	}

运行结果：

![](https://wcc-blog.oss-cn-beijing.aliyuncs.com/img/json5.jpg)

源码下载及实用的API地址：

- 本项目CodeBlock工程源码下载：[MyJSON](https://wcc-blog.oss-cn-beijing.aliyuncs.com/BlogFile/MyJSON.rar)

- 在线JSON格式校验工具：[bejson](http://www.bejson.com/)

- 免费的天气api接口：[天气API](http://api.help.bj.cn/api/)

## 历史精选文章：

- [JSON简介](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483763&idx=1&sn=b12aed2424a355a9aacc56c5f7ba9917&chksm=fadfa71dcda82e0b7f83b31b748630a578118f34529e60e13408691f0f249b1036e50d4a2aa2&token=1722697206&lang=zh_CN#rd)

- [Jlink使用技巧之J-Scope虚拟示波器功能](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483680&idx=1&sn=882e829f182219eb9293d9e010567748&chksm=fadfa74ecda82e58c1455db594d23d3cc121dfe019099cff3f7f297d4cb2459493d940e4b45c#rd)

- [BIN、HEX、AXF、ELF文件格式有什么区别](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=2&sn=e59ee5d6ea3098937bed342cd1c773e0&chksm=fadfa779cda82e6f72b5fbc52d7e6aeda25abf061763bb38655e13611301cde2a5f75dd72dbd#rd)

- [如何在Keil-MDK开发环境生成Bin格式文件](https://mp.weixin.qq.com/s?__biz=MzUzNzk2NTMxMw==&mid=2247483671&idx=1&sn=20422bf86fd8b58b58be47f2bae8819a&chksm=fadfa779cda82e6f9747c00d2f2ac763eb503f8d46b768c89a5c53a8bda6eb255deded727823&token=855879741&lang=zh_CN#rd)

