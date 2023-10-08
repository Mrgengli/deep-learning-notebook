# AI笔记生成的代码
## 一、第一部分 内容理解部分
* 1.入参校验
  * 在标准的入参格式有三个is_must，default，the_type。必选字段不允许有缺失，非必选的话用默认值。
  * 格式校验：检验传入的参数是不是要求格式，不是的话转化一下
  * 这里传入的参数有：
 
'''
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
'''
