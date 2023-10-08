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
### 2.视频文本获取
* 在传入类型为视频的时候，将连起来的OCR文本传给m_content，如果没有ocr就传入标题
* 将视频文案（m_content）进行润色压缩，这里的输入要大于15个字符，这里调用一言大模型对文案进行润色，去除重复，错别字，翻译英文为中文。
* 对文案的类型进行判断，根据所属的一级垂类，以及文案的长短进行判断“0” 是结构化“1”是人格化 “2”是其他，无文案，短文案“3”是字幕拼接
* 对应的结构化和人格化会有不一样的prompt，调用大模型对文案判断是否适合用来进行文案生成，并且生成详细的xml格式的笔记文案
* 解析xml文件，解析出note_content和note_title。后处理对重复的内容进行过滤

输出实例：
```
{
       "taskid": "0dddd",
        "note_text_type": "0",
	"note_title": "南台控股：让白酒文化在您的口中徜徉",
	"note_content": {
		"main_cotent": [{
				"content": "南台控股，作为一家新兴的白酒集团公司，在不断进行着品牌创新和科技研发。依托雄厚的科技支撑和管理经验，南台控股在白酒行业中已经获得了极高的品牌知名度和用户口碑。南台米香型白酒，正是南台控股产品线中的一颗璀璨明珠，被誉为米香型白酒的代表之一，广受品酒人士喜爱。",
				"sub_title": "",
				"key_words": ""
			},
			{
				"content": "南台米香型白酒凭借着优质水源和传统酿造工艺，使得口感更加醇和、余味更加持久，成为了品酒人士追捧的佳酿",
				"sub_title": "🍶南台米香型白酒的特点",
				"key_words": "南台米香型白酒"
			}
		],
		"main_words": ["白酒"],
                 "highlight_moments": [['如果不避孕', 298, 28, 151, 245, 0], ['女性一辈子能生多少孩子呢', 298, 29, 372, 134, 2], ['从科学的角度来讲', 296, 31, 246, 198, 4], ['女性从12岁左右第一次月经开始', 296, 31, 437, 101, 7], ['一直到绝经期', 295, 33, 185, 228, 8], ['每个月大约会排出一到两枚的卵子', 296, 31, 463, 88, 11], ['一生呢大概能排出400到500枚', 294, 34, 421, 108, 14], ['在理想的情况下有500次受孕的机会', 298, 28, 485, 77, 17], ['但是呢生娃当然没有那么简单', 298, 28, 398, 121, 20]]
	}
}
```

## 三、图像生成模块
* 输入参数
```
in_fea = {
            "input_fea": {
                "taskid": "hxy_55373737",
                "img_urls": [],
                "cover_images": [],
                "video_img_urls": []
                },
            "input_result": {
                'query_info': [{'query': {'category': '母婴育儿', 'subject': '', 'color': ''}, 'type': '1'}]
            }}
```

这段代码实现了一个图像处理的函数 image_process_impl。该函数接收一个字典类型的参数 input_data，其中包含了需要处理的图像相关信息。函数首先解析输入参数中的图片信息和任务ID，并根据任务ID创建存储路径。然后将待清洗的图片 URL 添加到 wait_clean_image_urls 列表中，并通过多线程分别调用 clean_image_process 和 query_note_process 函数进行清洗和检索处理。待清洗的图片包括 video_img_urls、img_urls 和 cover_images 中的图片 URL。

在执行清洗和检索处理时，使用了 ThreadPoolExecutor 类来实现多线程，分别设置最大工作线程为 2。如果存在待清洗的图片，则调用 **clean_image_process** 函数对**图片进行清洗**，返回清洗后的图片 URL 信息，存储在 clear_img_urls 字典中。如果存在查询（query_info）信息，则调用 **query_note_process** 函数对查询信息进行处理，返回背景图片 URL，存储在 backgroud_img_urls 列表中。  ???

最终返回一个字典类型的变量 image_process_result，其中包含了清洗后的图片 URL 信息、查询结果背景图片 URL 信息等。
* clean_image_process：
* 用于对给定的图片 URL 进行清洗处理。函数接收四个参数：img_url 为待清洗的图片 URL，task_id 为任务ID，save_path 为存储路径，img_idx 为图片索引。  
首先通过 _get_image_url_base64 函数将图片 URL 转换为 base64 格式，并使用 ThreadPoolExecutor 类实现多线程。在此代码中，使用该类同时执行两个函数：logo_img_impaint 和 smooth_img。其中 logo_img_impaint 函数用于**清除图片中的水印**，smooth_img 函数用于判断并提高图像清晰度。  
接着获取 logo_img_impaint 和 smooth_img 函数的返回结果。通过 logo_img_impaint.get("result", "") 获取清除水印后的图像信息，通过 smooth_img_impaint.get("result", {}).get("prob", []) 获取清晰度估计值。如果清晰度得分足够高，同时也经过了水印去除操作，则将经过处理后的图片保存到本地，并返回其对应的 URL。如果清晰度得分不够高或者处理失败，返回 None。  
最后输出日志信息并返回保存在 variable clean_image_url 中的 URL。
* _query_note_process：
* 首先从 query_detail_info 中获取查询类型 query_type。如果 query_type 为 "1"，表示需要进行查询操作。然后从 query_detail_info 中获取查询条件，包括类别 category、颜色 color 和主题 subject。接下来定义了一个变量 size，表示查询结果数量，目前设置为10。  
根据查询类型和查询条件，调用 megservice.queryNote 函数发起查询请求，并将查询结果保存在 note_result_lst 变量中。如果查询类型为 "0" 或其他值，则将 note_result_lst 置为空字典。  
最后输出日志信息，并返回查询结果 note_result_lst。
* 输出参数：
```
{
    "other_img_urls": [],
    "backgroud_img_urls": [
        {}
    ],
    "clear_img_urls": {
        "http://t7.baidu.com/it/u=2531715506,2113048258&fm=3035&app=3035&f=JPEG?w=464&h=194&s=26BCE5233AC8B80D6A7D04DB030050B2": {
            "new_url": "http://bjh-pixel.bj.bcebos.com/3806547008298063354exp_09271695798994.241914_0_ainote_new.jpg",
            "logo_removed": 0
        },
        "http://t9.baidu.com/it/u=2531715506,2113048258&fm=3035&app=3035&f=JPEG?w=464&h=194&s=26BCE5233AC8B80D6A7D04DB030050B2": {
            "new_url": "http://bjh-pixel.bj.bcebos.com/3806547008298063354exp_09271695798994.241914_1_ainote_new.jpg",
            "logo_removed": 0
        },
        "http://t7.baidu.com/it/u=2491411430,1768003352&fm=3035&app=3035&f=JPEG?w=640&h=399&s=29455A6EE4BA907C52CC6C980300C092": {
            "new_url": "http://bjh-pixel.bj.bcebos.com/3806547008298063354exp_09271695798994.241914_2_ainote_new.jpg",
            "logo_removed": 1
        },
        "http://t8.baidu.com/it/u=3051736189,1421080769&fm=3035&app=3035&f=JPEG?w=491&h=460&s=417016C69E632E17E0308A2D0300F043": {
            "new_url": null,
            "logo_removed": 0
        },
        "http://t7.baidu.com/it/u=2932151782,3229170893&fm=70&app=139&f=JPEG?w=690&h=872&s=9E82742313EED0EF1E5585DF0000D0B2": {
            "new_url": "http://bjh-pixel.bj.bcebos.com/3806547008298063354exp_09271695798994.241914_4_ainote_new.jpg",
            "logo_removed": 1
        }
    }
}
```
## 四、笔记编排
输入的参数包括 input_fea,input_result,text_result,image_result
输出的参数为：
```
out_result = {
            "taskid": taskid,
            "note_type": note_type, 
            "note_text_info": note_text_info,
            "note_image_info": note_image_info,
            "render_info": render_info
            }
```
### 1.获取输出文案
通过遍历 in_note_content 列表中的每个元素，提取其中的 "content" 和 "sub_title" 字段的值。如果子标题的长度大于0，则将其添加到 sub_title_list 列表中，并将子标题添加到 out_note_content 列表中。如果内容的长度大于0，则将其添加到 out_note_content 列表中。最后，通过使用换行符将 out_note_content 列表中的内容连接为一个字符串，并将该字符串赋值给 out_note_contents 变量。最后，将 note_title 和 out_note_contents 组成一个字典 note_text_info，并将 note_text_info 和 sub_title_list 作为返回值返回。
### 2.通过sub_source字段是否是"hot_event"
对备选图进行一个粗筛，看看url是否有内容或者格式上的错误
```
判断 clear_img_urls[url] 是否包含 "new_url" 字段。如果不包含，则进入下一次循环。

如果包含 "new_url" 字段，首先获取 new_url 的值。然后，尝试进行以下判断：

如果 new_url 为 None，则记录相应的日志并跳过该URL。
如果 new_url 不包含 "http"，则记录相应的日志并跳过该URL。
如果 new_url 在 new_frame_list 中已经存在，则进入下一次循环。
否则，将 new_url 添加到 new_frame_list 中。
在整个循环结束后，将 new_frame_list 作为方法的返回值返回。
```
* 不是"hot_event"的话，使用image_filter函数:
在上面的基础上考虑了将frame_list（这个是input_result里面的，）使用了上面的方法。将img_urls（这个是input_fea里面的）和cover_images加到了输出里面。

### 3.图片编排 这里只考虑了结构化大图和上图下文，没有字幕拼接 
* 上图下文（有四种来源：热点配图，视频配图，图文配图和纯文本素材配图）
  * 热点配图 other_image_matching：图片尺寸过滤height / width < 0.5 or height / width > 3，多模去重（image_embdding_diff）：对给定的备选图进行去重调用另外一个名为 get_embdding 的方法，获取每个备选图URL的多模向量，去掉那些模态相似度超过百分之85的，如果去重之后的数量还是大于max_mum,随机选一些和max_mum数量一样的图
  * 视频配图 video_image_matching 这里的max_mum是怎么回事？这里也是多模去重，然后进行图片拼接。（首先，创建一个空列表 contact_image_list，用于保存拼接后的图片URL。然后，定义一个变量 index 并初始化为 0，以及一个空列表 c_list，用于临时保存待拼接的图片URL。接下来，使用 for 循环遍历 new_image_list 中的每个元素（即配图URL）。首先判断 c_list 的长度是否小于 v_num，如果小于，则将当前元素添加到 c_list 中。否则，说明 c_list 已经达到或超过了建议竖向拼接的图片数量，需要进行拼接操作。根据给定的任务ID和索引 index 构造输出路径 out_path，然后调用 self.read_img 方法读取 c_list 中的图片，并将结果保存在 c_list 中。接着，使用 ic.run_concat 方法对 c_list 进行竖向拼接操作，并将拼接后的图片上传到 BOS，并将返回的拼接图URL添加到 contact_image_list 中。如果拼接过程中出现异常，则记录日志并继续处理下一组图片。最后，增加 index 的值，重新赋值给 c_list，并将当前元素添加到 c_list 中。在整个 for 循环结束后，如果 c_list 的长度大于 1，说明还有剩余未拼接的图片，需要进行最后的拼接操作。同样地，根据给定的任务ID和索引 index 构造输出路径 out_path，然后调用 self.read_img 方法读取 c_list 中的图片，并将结果保存在 c_list 中。接着，使用 ic.run_concat 方法对 c_list 进行竖向拼接操作，并将拼接后的图片上传到 BOS，并将返回的拼接图URL添加到 contact_image_list 中。如果拼接过程中出现异常，则记录日志。最后，如果 c_list 的长度等于 1，说明只有一张图片没有进行拼接操作，直接将该图片的URL添加到 contact_image_list 中。最终，将 contact_image_list 作为方法的返回值返回。）
  * 图文配图 other_image_matching
  * 纯文本、素材检索配图 无内容
* 大图输出 背景图渲染 background_director。  
  * 1.获取背景图和模版图
  * 主要实现了从服务器获取对应颜色标签（这个颜色标签来自backgroud_img_urls）的模板信息，并将其解析成一个字典返回。
  * 首先，构造一个字典 data，将颜色标签放入其中。然后，使用 requests.post 方法向指定的URL发送POST请求，获取返回的JSON格式数据并解析出其中的模板列表 res。如果 res 中有多个模板，则随机选择一个；如果只有一个模板，则直接选取；如果没有模板，则返回空字典 {}。接着，对于每个模板，分别解析出其模板ID、标签、插槽信息和图层信息等，并将其保存在一个字典 model_out 中。其中，对于背景图插槽信息，会将其命名为 backgound。
  * 在backgroud_img_urls中获取back_img_url？
  * 2.字幕切分和渲染参数组装
  * 







