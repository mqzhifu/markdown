
|          |             |     |     |      |
| :------- | :---------- | :-- | :-- | ---- |
| open-ai  | gpt4        |     |     |      |
| Meta     | Llama 2     | 开源  |     |      |
| google   | Gemma       | 开源  |     |      |
| tesla    | Grok        | 开源  |     |      |
|          |             |     |     |      |
| 百度       | 文心一言 ernie  |     |     |      |
| 科大讯飞     | 星火          |     |     |      |
| 阿里云      | 通义千问        |     |     |      |
| 腾讯       | 混元          |     |     |      |
| 字节跳动     | Grace 云雀    |     |     |      |
| Moonshot | Kimi        |     |     |      |
| 智谱       | 清言 ChatGLM3 |     |     | TO B |
|          |             |     |     |      |
|          |             |     |     |      |


> 开源的大模型目前具说，大多在GPT3.5的水平，较好的能接近GPT4



# 预训练（Pre Train）


预训练：大量的数据进行训练。价格非常贵，且可能几个月到一年才跑一次。能得到一个基础大模型
无监督 、大数据喂养


难点：
- 投入高。具说一次预训练 ：一亿起步.....  GPU、人力、电力等
- 数据量大。很难聚合非常多的数据
- 时间成本。以月为单位



# 微调（Fine-Tune）

微调：在基础大模型上再喂一些数据，进行微调。可以是：垂直领域的数据
人工监督、精确数据喂养

# 如何定义套壳？


#### 只做API层面最简单封装

用户发送什么数据，直接转到GPT，GPT回答的数据，又直接转给用户。卷：UI

#### Prompt

对于如何向大模型提问，能自己总结出一套 Prompt 词库。卷： Prompt 质量高

#### 特定数据集-向量数据库

垂直领域、私人数据等，给到基础模型，这样回答的结果更精准。对比 Prompt 更专业一点吧。

#### 微调 Fine-Tuning

对基础模型进行二次训练






2020 年之前的算法研究阶段，2020~2023 年的数据为王阶段，以及 2023 年的 AI Infra 阶段。


|           |                        |           |      |     |
| :-------- | :--------------------- | :-------- | :--- | --- |
|           |                        |           |      |     |
| Anthropic | Claude                 | 大文本       |      |     |
|           | Poe                    | 工具合集      | 官方套壳 |     |
|           |                        |           |      |     |
|           | stable diffusion       | 图片        | 免费   |     |
|           | midjourney             | 图片        |      |     |
|           | dall-e                 | 图片        |      |     |
|           | canva                  | 图片        |      |     |
|           |                        |           |      |     |
|           |                        |           |      |     |
|           |                        |           |      |     |
|           | sora                   | 视频        |      |     |
|           | stable video diffusion | 视频        | 免费   |     |
|           | runway                 | 视频        |      |     |
|           | pika                   | 视频        |      |     |
|           | haiper                 | 视频        | 免费   |     |
|           |                        |           |      |     |
|           | suno                   | 音乐        |      |     |
|           | stable audio           | 音乐        |      |     |
|           | perplexity             | 音乐        |      |     |
|           |                        |           |      |     |
|           | github coplilot        | 编程        |      |     |
|           |                        |           |      |     |
| 阿里        | EMO                    | 数字人-图片转动态 |      |     |
|           | heygen                 | 数字人       |      |     |
|           | D-iD                   | 图片转动态     |      |     |
|           | sad-talker             |           | 免费   |     |


  
  

 









