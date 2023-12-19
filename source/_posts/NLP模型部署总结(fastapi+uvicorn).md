---
title : NLP模型部署总结
categories : [python]
tags : [fastapi]
date: 2023-06-11 10:00:00
updated: 2023-06-11
cover: https://source.wjwsm.top/assassins_creed_mirage_video_game_2023-wallpaper-2560x1440%20(1).jpg
---

# NLP模型部署总结(fastapi+uvicorn)

## 前言

最近生成环境需要一些将模型部署到后端,也就是用api的形式提供服务,所以在这边记录一些遇到的坑

## 模型生成和部署构思

首先肯定是需要先将模型进行微调,之后先上传到服务器上,这边就举例最近很受欢迎的chatglm-6b.首先我们先将模型先下载到本地,之后使用trensformer来进行加载

~~~python
from transformers import AutoTokenizer, AutoModel
tokenizer = AutoTokenizer.from_pretrained(
    "THUDM/chatglm-6b", trust_remote_code=True
)
model = AutoModel.from_pretrained("THUDM/chatglm-6b", trust_remote_code=True).half().cuda()
response, history = model.chat(
        tokenizer,
        prompt,
        history=history,
        max_length=max_length if max_length else 2048,
        top_p=top_p if top_p else 0.7,
        temperature=temperature if temperature else 0.95,
    )
~~~

> 需要注意自己的显卡显存,正常的模型需要占用12GB的显存,如果没有这么大建议调整下参数降级.因为是投入生产使用,所以并没有考虑到使用流式模型,感觉那样反而是性能负提升

这样一个模型初始化到调用的过程就完成了,但是这仅仅只是个开始.

## 后端框架选择

其实除了chatglm-6b,其实还有一些其他开源模型,比如bert模型也要在后台部署,这是就要考虑,是将他们分开部署成服务,还是融合到一起方便管理呢?

还有框架的选择,因为模型是使用python的深度学习框架进行加载,而且这次部署的对象并没有很高的并发需求,所以想了想还是使用python的后端框架来快速搭建应用,总而言之,先跑起来才是王道!

python轻度的后端网络框架,近些年比较热门的就属老牌的flask和新星之秀fastapi了.其实这里我有踩坑,先用flask进行了一次部署.

### 1. fastapi部署

> 使用fastapi部署之前我尝试使用flask,之后因为一些坑,我个人没办法填了,就切到fastapi了.最主要的考虑是,模型的初始化的位置,放到全局变量.还是放到路由函数中,因为我当时可能想到并发,只初始化一个模型,是否会导致线程安全呢?而且在路由函数中,每一个请求都会消耗时间去加载,所以这边就要取舍了,对于一些加载时间快的小模型,我可以在路由函数中创建模型初始化,对于chatglm-6b 12GB的大模型,就初始化一个最后串行执行任务.

~~~python
app.py
from transformers import AutoTokenizer, AutoModel
from fastapi import Fastapi
import torch
DEVICE = "cuda"
DEVICE_ID = "0"
CUDA_DEVICE = f"{DEVICE}:{DEVICE_ID}" if DEVICE_ID else DEVICE
tokenizer = AutoTokenizer.from_pretrained(
    "THUDM/chatglm-6b", trust_remote_code=True
)
model = AutoModel.from_pretrained("THUDM/chatglm-6b", trust_remote_code=True).half().cuda()
response, history = model.chat(
        tokenizer,
        prompt,
        history=history,
        max_length=max_length if max_length else 2048,
        top_p=top_p if top_p else 0.7,
        temperature=temperature if temperature else 0.95,
    )
app = Fastapi()
def torch_gc():
    if torch.cuda.is_available():
        with torch.cuda.device(CUDA_DEVICE):
            torch.cuda.empty_cache()
            torch.cuda.ipc_collect()
@app.post("/chat")
def chat():
    prompt = json_post_list.get("prompt")
    history = json_post_list.get("history")
    max_length = json_post_list.get("max_length")
    top_p = json_post_list.get("top_p")
    temperature = json_post_list.get("temperature")

    answer = chat(prompt, history, max_length, top_p, temperature)
    torch_gc()

    return answer
~~~

刚开始,我发现如果我主动gc,随着prompt的量的增多,显存会逐渐增加,最后就会把显存压垮,所以这里顶一个torch_gc()方法,完成对显存的回收,但是这样部署一个应用就没问题了,但是如果,我的服务器上有四张显卡,想要提升效率,同一个接口,后面通过负载均衡,来对应四个单独的服务呢?而且这四个服务都要是串行,模型无法并发呢?

这个时间大家可能是这么想,我可以修改服务对象的显卡变量,启动四个服务,之后使用nginx来帮助我们来做负载均衡!这的确是一种处理方式,但是我当时的开发环境是在docker容器中,但是的镜像并没有nginx的镜像,所以这个方法,并没有办法适配我的环境,那么还有什么方法呢 ?我是不是可以通过python的一些服务框架帮我完成负载均衡这个任务呢?

之后我就注意到和fastapi一起使用的服务框架uvicorn,uvicorn是一个基于asyncio开发的一个轻量级高效的web服务器框架,他可以帮我们一步到位,解决一系列问题,下面是安装命令.

~~~python
pip3 install uvicorn
~~~

~~~shell
uvicorn app:app -host 0.0.0.0 -port 8555 -worker 3
~~~

由于uvicorn会给每个启动服务生成一个独一pid,那我们完成可以在代码中读取pid之后与显卡总数做取余操作,这样每个进程都会分到一个唯一的显卡中去,并且uviconr还会监听网关,自主帮助我们完成任务的平均分发,实现负载均衡的并发.

那么接下来还有最后一个问题,如何保证没一个单独的服务都是串行执行呢?

其实只需要将我们路由函数改成async 协程函数就可以了,我注意到如果协程函数中没有其他协程函数就会变成串行执行,主要uvicron没有限制线程数的参数,只能通过这种方式变相实现串行执行!

~~~python
app.py
from transformers import AutoTokenizer, AutoModel
from fastapi import Fastapi
import torch
DEVICE = "cuda"
DEVICE_ID = "0"
CUDA_DEVICE = f"{DEVICE}:{DEVICE_ID}" if DEVICE_ID else DEVICE
tokenizer = AutoTokenizer.from_pretrained(
    "THUDM/chatglm-6b", trust_remote_code=True
)
model = AutoModel.from_pretrained("THUDM/chatglm-6b", trust_remote_code=True).half().cuda()
response, history = model.chat(
        tokenizer,
        prompt,
        history=history,
        max_length=max_length if max_length else 2048,
        top_p=top_p if top_p else 0.7,
        temperature=temperature if temperature else 0.95,
    )
app = Fastapi()
def torch_gc():
    if torch.cuda.is_available():
        with torch.cuda.device(CUDA_DEVICE):
            torch.cuda.empty_cache()
            torch.cuda.ipc_collect()
@app.post("/chat")
def chat():
    prompt = json_post_list.get("prompt")
    history = json_post_list.get("history")
    max_length = json_post_list.get("max_length")
    top_p = json_post_list.get("top_p")
    temperature = json_post_list.get("temperature")

    answer = chat(prompt, history, max_length, top_p, temperature)
    torch_gc()

    return answer
~~~

### 2.多模型任务

如果为了保障线程安全如何保证多次加载模型导致的cuda显存溢出呢?

如果你在路由函数里面,初始化了模型,虽然最后任务结束,但是并不会释放显存,最终会显存溢出,这里提供一个我的思路,我这边会在后会del掉初始化的模型对象,并且使用torch.cuda.empty_cache()手动gc掉,但是如果你的模型很小,需要考虑并发,这个时间就要考虑如何最大化设置并发数,或者让客户端设置休眠时间,错开最大并发导致的显存溢出的可能性!