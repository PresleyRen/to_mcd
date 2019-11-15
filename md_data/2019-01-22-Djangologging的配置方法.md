---
layout:     post
title:      Djangologging的配置方法
subtitle:   
date:       2019-01-22
author:     P
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - python
---
做开发离不开日志，以下是我在工作中写Django项目常用的logging配置。

{% raw %}
```
BASE_LOG_DIR = os.path.join(BASE_DIR, "log")
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '[%(asctime)s][%(threadName)s:%(thread)d][task_id:%(name)s][%(filename)s:%(lineno)d]'
                      '[%(levelname)s][%(message)s]'
        },
        'simple': {
            'format': '[%(levelname)s][%(asctime)s][%(filename)s:%(lineno)d]%(message)s'
        },
        'collect': {
            'format': '%(message)s'
        }
    },
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'filters': ['require_debug_true'],  # 只有在Django debug为True时才在屏幕打印日志
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'SF': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件，根据文件大小自动切
            'filename': os.path.join(BASE_LOG_DIR, "xxx_info.log"),  # 日志文件
            'maxBytes': 1024 * 1024 * 50,  # 日志大小 50M
            'backupCount': 3,  # 备份数为3  xx.log --> xx.log.1 --> xx.log.2 --> xx.log.3
            'formatter': 'standard',
            'encoding': 'utf-8',
        },
        'TF': {
            'level': 'INFO',
            'class': 'logging.handlers.TimedRotatingFileHandler',  # 保存到文件，根据时间自动切
            'filename': os.path.join(BASE_LOG_DIR, "xxx_info.log"),  # 日志文件
            'backupCount': 3,  # 备份数为3  xx.log --> xx.log.2018-08-23_00-00-00 --> xx.log.2018-08-24_00-00-00 --> ...
            'when': 'D',  # 每天一切， 可选值有S/秒 M/分 H/小时 D/天 W0-W6/周(0=周一) midnight/如果没指定时间就默认在午夜
            'formatter': 'standard',
            'encoding': 'utf-8',
        },
        'error': {
            'level': 'ERROR',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件，自动切
            'filename': os.path.join(BASE_LOG_DIR, "xxx_err.log"),  # 日志文件
            'maxBytes': 1024 * 1024 * 5,  # 日志大小 50M
            'backupCount': 5,
            'formatter': 'standard',
            'encoding': 'utf-8',
        },
        'collect': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件，自动切
            'filename': os.path.join(BASE_LOG_DIR, "xxx_collect.log"),
            'maxBytes': 1024 * 1024 * 50,  # 日志大小 50M
            'backupCount': 5,
            'formatter': 'collect',
            'encoding': "utf-8"
        }
    },
    'loggers': {
        '': {  # 默认的logger应用如下配置
            'handlers': ['SF', 'console', 'error'],  # 上线之后可以把'console'移除
            'level': 'DEBUG',
            'propagate': True,
        },
        'collect': {  # 名为 'collect'的logger还单独处理
            'handlers': ['console', 'collect'],
            'level': 'INFO',
        }
    },
}
```
{% endraw %}

附：Python logger流示图

<img src="https://images2018.cnblogs.com/blog/867021/201804/867021-20180409234258183-677608114.png" alt="" /> 
