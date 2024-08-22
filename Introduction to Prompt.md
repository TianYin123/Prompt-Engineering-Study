#   微软Prompt Engineering

[toc]

##  Instruct 指令

[Instruct]是指导模型应该完成的任务。
包含Simple instructions/Complex instructions两种，以下给出例子：

- Simple instructions: Write an introduction for a weekly newsletter.
- Complex Instructions:
  - 为每周时事通讯撰写简介，包括以下内容：
    \- 我的公司名为 contoso
    \- 热情洋溢的问候
    \- 感谢团队在艰难的几个月里的辛勤工作
    \- 对下个季度的乐观前景
    \- 由SLT签署

##  Primary Content 主要内容

[content]项一般是模型生成内容的主要依据，模型会结合instruct与content一起回答问题（不需要举例）

## Examples 例子

[Example]即给出类似的content和Instruct以及其对应的结果。供大模型学习，一般分为ZeroShot和FewShot，例如：

- Zero-shot :即没有Example
  - Headline: Coach confident injury won't derail Warriors, summarize the topic: 
- One/Few-shot:即有1/少量Example
  - [Example]
    Headline: Twins' Correa to use opt-out, test free agency
    Topic: Baseball
    Headline: Qatar World Cup to have zones for sobering up
    Topic: Soccer 
    Headline: Yates: Fantasy football intel for Week 6
    Topic: Football
  - [Content]Headline: Coach confident injury won't derail Warriors
    [Instruct]summarize the topic:

## Cue 提示

[Cue]并常常不作为单独的一部分输入给模型，而是作为Instruct的一部分，用于帮助将模型定向到所需的输出。它通常是模型可以基于它进行构建的前缀，可以视为为模型开个好头，例如：

- Cue number=0
  - [content]xxxxxx(email message)
    [Instruct]Summarize the above email message:
  - "Model Output":The purpose of this email is .... The changes include an improved structure that ..., there is now an optional that ...

- Cue number=1
  - [content]xxxxxx(email message)
    [Instruct]Summarize the above email message: ([Cue1])The key takeaway of the message is
  - "model output":that... Additionally, .... Finally, ...
- Cue number=2
  - [content]xxxxxx(email message)
    [Instruct]Summarize the above email message: 
    ([Cue1])Key points:
    ([Cue2])1、 
  - "model output":Contoso is introducing changes to the performance review experience, based on feedback from employees.
    1、xxx
    2、xxx
    ...

在上面的例子中我们可以看到，Cue number=1中模型紧跟着就输出了"that..."说明Cue为模型开了个好头，而Cue number=2中，模型虽然重新生成了简短的总结语，但还是按照Cue2的格式回答了问题，说明插入在Instruct中的Cue起到了一两拨千斤，让模型针定向输出的作用。

## Supporting content 辅助信息

可以理解为Content中的Cue，Supporting content可以也可以更好地定向内容的生成，除此之外，用户信息、当前日期等对回答中的逻辑关系无关紧要的次要信息也可以放在Support Content中。例如：

- 没有Supporting content

  - [content]研讨会文字记录稿：...
    [Instruct]总结上述研讨会，按主题分组：
  - "Model Output":
    1- 研讨会开始：xxx  2-研讨会总结:xxx  3-核心共识:xxx  4-重点领域:xxx ...

- 有Supporting content

  - [content]研讨会文字记录稿：...  我的重要主题是：提示工程、推荐算法、GPT 模型
    [Instruct]总结上述研讨会，按主题分组：
    
  - "Model Output":
    1-提示工程：...  2-推荐算法：...  3-GPT模型：...
    
## Space efficiency 输入效率 
由于模型每次输入的最大token是有限的，如果不加History消息，模型无法从之前的对话中了解更多的信息，因此输入效率是非常重要的。比如2024年2月3日就可以写成2024.2.3以节省token。除此之外，合理使用表、html、json等非人类直觉的输入形式，可以有效节省token。
空格的使用也需要讲究，空格在人类看来可能是没有含义的，但LLM会将空格和前一个字理解成一个新词，比如“你   好”和“你好”在LLM看来是完全不同的两个词汇，这不仅会增大输入token，跟会影响LLM对语义的理解。
