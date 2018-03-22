---
title: django2下template路径配置
date: 2018-03-22 15:43:46
tags: [Django]
urlname: django2-template-path-config
---

前端时间在那django些个小网站，其中需要配置下template路径，按以前经验配置了下，

```python
TEMPLATE_DIRS = (
    os.path.join(BASE_DIR, 'templates').replace('\\', '/'),
)
```

弄了半天也没成功。最后无奈去看了下官方文档，找到内容有限，不过可以确定的是`TEMPLATE_DIRS`这个配置项已经没有了。

然后谷歌了下，找到了新的配置方法：

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # 这里，这个DIRS就是
        'DIRS': [os.path.join(BASE_DIR, 'templates').replace('\\', '/'),],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

这个方法在django 1.8以上到我现在的2.0.2有效。不保证以后版本依然有效。