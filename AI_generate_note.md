# AI笔记生成的代码
## 一、第一部分 内容理解部分
* 1.入参校验
  * 在标准的入参格式有三个is_must，default，the_type。必选字段不允许有缺失，非必选的话用默认值。
  * 格式校验：检验传入的参数是不是要求格式，不是的话转化一下
  * 这里传入的参数有：
 ```
INPUT_FEA = {
    "taskid": { # 任务id, 必填
        "is_must": True,  
        "type": str,
        "default": ""
        },
    "source": { # 来源, 必填
        "is_must": True, 
        "type": str,
        "default": ""
        },
	"type": { # 内容类型, 必填
        "is_must": True, 
        "type": str,
        "default": ""
        },
	"title": { # 标题, 必填
        "is_must": True, 
        "type": str,
        "default": ""
        },
	"m_content": { # 图文主体内容, 非必填, 默认值为空
        "is_must": False, 
        "type": str,
        "default": ""
        },
	"cate_v1": { # 内容一级分类, 必填
        "is_must": True, 
        "type": str,
        "default": ""
        },
	"cate_v2": { # 内容二级分类, 必填
        "is_must": True, 
        "type": str,
        "default": ""
        },
	"img_urls": { # 原文图片列表, 非必填, 默认值为[]
        "is_must": False, 
        "type": list,
        "default": []
        },
    "cover_images": { # 封面图片列表, 非必填, 默认值为[]
        "is_must": False, 
        "type": list,
        "default": []
        },
    "ocr_info": { # 视频cor识别信息, 非必填, 默认值""
        "is_must": False, 
        "type": str,
        "default": ""
        },
	"asr_info": { # 视频asr识别信息, 非必填, 默认值""
        "is_must": False, 
        "type": str,
        "default": ""
        },
    "face_url": { # 人脸识别结果, 非必填, 默认值""
        "is_must": False, 
        "type": str,
        "default": ""
    },
    "video_url": { # 视频下载地址, 非必填, 默认值""
        "is_must": False, 
        "type": str,
        "default": ""
        },
    "first_frame_url": { # 视频首帧下载地址, 非必填, 默认值""
        "is_must": False, 
        "type": str,
        "default": ""
        },
    "duration": { # 视频时长, 非必填, 默认值
        "is_must": False, 
        "type": int,
        "default": -1
        }
}
```
* 2.解析每个参数，传进来（之前都在input_feature这个字典中）
* 3.内容筛选：是否适合生成笔记
	* 根据一级垂类过滤，保留想要的垂类
 	* 根据标题的时效性进行过滤：1.过滤掉标题中含有2023年之前的 2.过滤掉类似“22年”格式的数据
* 4.视频文案清洗（这里在传入的是视频的情况下使用，也就是source_type == "2"）：获得视频中的文本内容（整合在一起的video_content，以及ocr_text_info）
  * 获得一个tolerate_size = min(height, width) * 0.05 ，最小为20
  * 在这里的文本检测会有两种，每一种的返回结果都有两个（全部文本和每个ocr文本的信息以及它的位置，这里位置基于top,height,left,width,计算 中心坐标x,y），一种是字幕的ocr文本，一种是OCR文本
  * 获得的ocr文本比较简单：清洗只把空的去掉，去掉了重复的字符串
  * 获得的字幕ocr文本：它通过对 OCR 输出的信息进行遍历，获取每个文本信息的左上角坐标、高度和宽度，然后将左上角坐标向左减去宽度的一半，并向上加上高度的一半，**计算中心坐标**，然后将中心坐标拼接成一个字符串作为键值，**并在 xy_count 中统计每个坐标出现的次数**。最后将出现次数最多的坐标拆分为横纵坐标，并赋值给变量 main_x 和 main_y。接下来的部分是**去重过滤**的过程。通过遍历 xy_con 列表，**判断每个坐标与主要坐标的横纵坐标差是否在容忍范围内**。如果在容忍范围内，比较当前文本信息与前一个文本信息是否相同，若不相同则将当前文本信息添加到 final_text_info 列表中，若相同则替换前一个文本信息为当前文本信息。最后，将 final_text_info 列表中的文本信息进行合并，并返回最终的文本信息 final_text 和包含详细信息的列表 final_text_info。
  * 接下来在这两种种做一个选择，如果len(ocr_zimu_text) >= len(ocr_text) * 0.5:选择字幕ocr文本
* 5.生成笔记样式选择：传入纯文本的话，生成结构化大图，传入图文和视频的话都生成上图下文
* 6.进行视频抽帧：根据前面获得的OCR（里面有index，通过first_frame_url可以获取所在帧）进行抽帧
* 7.获取一个query（这里还不太明白获取它的目的）：
  * 这里是只有生成结构化大图的笔记的时候才会有，里面包含query（一级垂类，subject，color），生成的笔记类型
* 8.组装输出
```
out_result = {
            "taskid": taskid,
            "note_type": note_type, 
            "video_content_info": {
                "text": video_content,
                "ocr_info": ocr_text_info
                },
            "video_img_urls": key_frame_list,
            "query_info": query_info
            }
```
## 二、内容理解部分
### 1.解析各个参数（传入）
```
{
	"input_fea": {
		"title": "为什么佛山瓷砖这么受欢迎？因为这5个优点太突出",
		"m_content": "<p data-from-paste=\"1\"><span data-from-paste=\"1\" data-diagnose-id=\"9046a74788c75d847f9a964282400a41\">说到瓷砖，大家首先会想到一句话：中国瓷砖看广东，广东瓷砖看佛山。可见，佛山瓷砖在国人心目中的地位有多高。</span></p><p data-from-paste=\"1\"><span style=\"background: rgb(184, 95, 37);color: rgb(255, 255, 255)\" data-diagnose-id=\"36baac7e8fba3ce977922b18f408c1a2\">其实全国各地的瓷砖生产基地有很多，但唯独只有佛山瓷砖远近闻名。那么，佛山瓷砖和其它瓷砖相比，究竟有怎样的优势呢？...",
		"cate_v1": "家居",
		"cate_v2": "家居产业",
		"img_urls": "{\"image_score_beta\": [{\"image_score_beta\": 145.80114616934603, \"url\": \"http://t12.baidu.com/it/u=634983708,212670459&fm=30&app=106&f=JPEG?w=312&h=208&s=BB90F1A012BC3982409192080300C0D4\",",
		"ocr_info": "",
		"asr_info": "",
		"source": "0",
		"type": "1"
	},
	"input_result": {
		"note_type": "0",
		"video_content_info": {
			"text": "测试"
		},
		"video_img_urls": ["http:xxxxxx.jpg"],
		"query_info": [{
			"query": "test",
			"type": "0"
		}]
	}
}
```
### 










