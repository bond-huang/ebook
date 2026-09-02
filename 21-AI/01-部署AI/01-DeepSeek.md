# DeepSeek

## 本地部署DeepSeek
&#8195;&#8195;下载并安装Ollama，下载地址：[Download Ollama](https://ollama.com/download)。安装完成后自动弹出PowerShell，官方直接搜索DeepSeek，选择deepseek-r1，选择一个直接复制命令开始部署，参考链接：[deepseek-r1](https://ollama.com/library/deepseek-r1)。示例：

```
PS C:\Users\admin> ollama run deepseek-r1:7b
pulling manifest
pulling 96c415656d37... 100% ▕████████████████████████████████████████████████████████▏ 4.7 GB
pulling 369ca498f347... 100% ▕████████████████████████████████████████████████████████▏  387 B
pulling 6e4c38e1172f... 100% ▕████████████████████████████████████████████████████████▏ 1.1 KB
pulling f4d24e9138dd... 100% ▕████████████████████████████████████████████████████████▏  148 B
pulling 40fb844194b2... 100% ▕████████████████████████████████████████████████████████▏  487 B
verifying sha256 digest
writing manifest
success
>>> ollama run deepseek-r1:7b
<think>

</think>

It seems like you're referring to a specific model or service related to Ollama and DeepSeek-R1. Could you clarify
what you're asking about? Are you looking for information on how to use Ollama with DeepSeek-R1, a technical
question, or something else? Let me know so I can assist better!

>>> Send a message (/? for help)
```
我的机器配置：
```
处理器：11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz   2.42 GHz
RAM：16.0 GB (15.7 GB 可用)
显卡：集成显卡
```
查看模型：
```
PS C:\Users\admin> ollama list
NAME              ID              SIZE      MODIFIED
deepseek-r1:7b    0a8c26691023    4.7 GB    9 hours ago
PS C:\Users\admin>
```
启动模型：
```
PS C:\Users\admin> ollama run deepseek-r1:7b
>>> 你好  
<think>
</think>
你好！很高兴见到你，有什么我可以帮忙的吗？无论是学习、工作还是生活中的问题，都可以告诉我哦！😊  
```

## 待补充
