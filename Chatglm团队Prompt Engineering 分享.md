# Prompt迭代技巧

- [Prompt迭代技巧](#prompt迭代技巧)
  - [核心guideline](#核心guideline)
  - [角色迭代](#角色迭代)
  - [任务迭代](#任务迭代)
  - [执行步骤迭代](#执行步骤迭代)
    - [避免负向指令](#避免负向指令)
    - [逻辑完备](#逻辑完备)
    - [避免规则](#避免规则)
  - [Few-shot迭代](#few-shot迭代)
  - [结构迭代](#结构迭代)
    - [分隔符](#分隔符)
    - [分条目](#分条目)
    - [顺序](#顺序)
    - [避免嵌套](#避免嵌套)
    - [位置](#位置)
- [Prompt Engineering实战](#prompt-engineering实战)
  - [情绪识别任务](#情绪识别任务)
  - [从文章中抽取信息](#从文章中抽取信息)
- [评测](#评测)
  - [构建评测集](#构建评测集)
    - [确定评测维度](#确定评测维度)
    - [评测集数量、评测集分布](#评测集数量评测集分布)
- [总结](#总结)

## 核心guideline

1. 指令应该清晰明确
2. 分析为什么Prompt不能获得有效的response
3. 改进Prompt和业务需求
4. 重复

## 角色迭代

问答任务的System message 往往是“你是一个问答机器人”，但对于更加垂类的问题，我们可以修改System message中的角色，迭代为更加垂类的问题，比如保险业务专家、python代码专家等等

## 任务迭代

对于指令中的关键动作，尝试不同的近义词或其他相近的描述来提升准确性，尤其是关键的动词或者名词，表达需要更丰富、具象化，尽量使用业务场景的词语。例如：在下面的例子中，左边是让LLM扮演对话教练，而右边直接给了LLM一个口语联系的场景，明显右边表现更好。

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/1.png)

## 执行步骤迭代

### 避免负向指令

避免负向指令，通过更换概念等方式，尽量告诉模型应该输出什么。

尤其是LLM只是一个token预测模型，它无法感知并理解“三百字”这种字数要求，因此将模糊、难以被模型理解的指令修改为需求清晰的正向指令可以有效提升模型response。例如：

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/2.png)

### 逻辑完备

将完整的处理问题的逻辑告诉大模型，避免大模型自由发挥。例如：

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/3.png)

### 避免规则

现有技术能力，除了chatgpt-4o(json)以外，其他模型并不能百分百按照Prompt中的规则输出格式数据。因此往往对规则的限制并不能使模型达到类似的效果，如以下两个例子，对模型的规则并不能起效。

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/4.png)

## Few-shot迭代

要么每个类别都均匀地加Example，要么就都不加。

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/5.png)

## 结构迭代

### 分隔符

分隔符的作用：

1. 避免模型指令受到其他内容的干扰
2. 将文本上下文，不同的知识模块做分割
3. 避免用户注入无关指令

例如：![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/7.png)

使用分隔符优化：

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/7.png)

### 分条目

分条目的作用：

1. 有助于大模型理解每个独立的任务，引导大模型按照指令顺序进行思考
2. 有助于开发者理解任务的逻辑顺序，便于逐条编写测试并进行迭代维护

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/8.png)

优化指令：

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/9.png)

### 顺序

先输入的内容会影响后输入的内容。可尝试不同的顺序，避免提取项之间的干扰

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/10.png)

指令优化：

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/11.png)

顺序调整的逻辑：
因为模型是根据我们现实生活中的数据进行训练的，所以顺序的调整需要尽量贴合我们现实生活，例如上面的例子，优化后的指令明显更符合文本(也就是现实世界)的沟通顺序，因此效果会更好。

### 避免嵌套

尽量平铺直叙，避免多层逻辑嵌套，尽量每个要求之间都是彼此独立的，不涉及要求和要求之间的逻辑关系。

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/12.png)

### 位置

在Prompt首部和尾部的Instruction，LLM遵循的效果最好，根据不同的输入文档长度和指令难度尝试不同的组织段落的方式。

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/13.png)

指令优化：

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/14.png)

# Prompt Engineering实战

## 情绪识别任务

需求描述：

- 背景：在中介的服务过程中，员工会建立微信群，与客户进行沟通，交流服务中的进度和问题

- 角色：客户、中介员工

- 主要需求：根据上下文及客户当前聊天记录，对客户当前这句话的情绪做识别
  - 情绪类型：正向、中性、负向
  - 需求细节：
    - 整体情绪识别准确率要求达到85%
    - 业务重视负向情绪的识别效果，要求高精准度

Prompt迭代：

1. 加入思考步骤和判断依据：![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/15.png)
2. 使指令更加清晰明确：![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/16.png)

3. 观察数据，发现最后一句话其实表达了客户的情绪。![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/17.png)

4. 增加Few-shot![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/18.png)

5. 增加对输出的约束![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/19.png)

## 从文章中抽取信息

需求描述：提取文章中的[比喻、拟人、排比]句，作为下游任务的数据输出。

Prompt迭代：

1. 定义三种句子![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/20.png)

2. 拆分Prompt、赋予角色![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/21.png)

3. 添加修辞关键特征、分析原因![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/22.png)

4. 添加动作描述、增加示例、引号加强![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/23.png)

5. 发现response中正例判断正确，但负例判断失误很多，因此增加反面示例![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/24.png)

6. 针对三种句子，分步骤推理分析![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/25.png)

# 评测

## 构建评测集

### 确定评测维度

1. 基于业务需求，确定评测维度
2. 参考不同场景的通用评测维度
3. 如果仍无法确定，使用小样分测试，从测评过程中迭代

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/26.png)

### 评测集数量、评测集分布

- 评测集数量保持在不少于50条，最好100+

- 评测集分布应与真实问题分布一致，按照线上抽样、按维度构造
  1. 线上抽样
  2. 小版本调优、灰度上线收集badcase，再迭代、上线
  3. 按维度构造，如下示例

- 测试迭代，为降低评测成本，前期可通过小部分评测集进行小版本迭代测试，效果稳定后再进行大版本完整评测集测试

![image](https://github.com/TianYin123/Prompt-Engineering-Study/raw/main/image/28.png)

# 总结

- Prompt是通过自然语言的方式，引导模型生产内容，不像软件开发，没有样本，更多是方法和原则， 没有固定的范式，但只要能解决问题就OK
- 指令本身对语言的结构和内容是非常敏感的
- 结合真实使用场景去调优指令，不同任务要使用不同的指令
- 不同模型之间的差异较大，针对同一问题的特定模型的Prompt无法在其他模型复用
