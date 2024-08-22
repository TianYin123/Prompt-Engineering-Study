# 微软Prompt Engineering technique

在学习该文章之前，最好有Prompt Engineering的基础，推荐先学习Introduction to Prompt，以了解基础概念。

- [微软Prompt Engineering technique](#--prompt-engineering-technique)
  * [System message 系统消息](#system-message-----)
  * [Few-shot learning 小样本学习](#few-shot-learning------)
  * [Start with clear instructions 从明确的指令开始&Repeat instructions at the end 在最后重复说明](#start-with-clear-instructions----------repeat-instructions-at-the-end--------)
  * [Prime the output 启动输出](#prime-the-output-----)
  * [Add clear syntax 添加清晰的语法](#add-clear-syntax--------)
  * [Break the task down 分解任务](#break-the-task-down-----)
  * [Use of affordances 使用引用(Supporting content)](#use-of-affordances------supporting-content-)
  * [Chain of thought prompting 思维链](#chain-of-thought-prompting----)
  * [Specifying the output structure 指定输出结构](#specifying-the-output-structure-------)
  * [Temperature and Top_p parameters 温度和 Top_p 参数](#temperature-and-top-p-parameters-----top-p---)
  * [Provide grounding context 提供基础背景](#provide-grounding-context-------)



## System message 系统消息

在ChatGPT等具有格式化输入输出的API中，系统消息包含在提示的开头，用于使用上下文、说明或与您的用例相关的其他信息来启动模型，有时等同于Instruction。我们可以使用系统消息来描述助手的个性，定义模型应该回答什么和不应该回答什么，并定义模型响应的格式。例如：

- [System message]Assistant 是 OpenAI 训练的大型语言模型。
- [System message]Assistant 是一个智能聊天机器人，旨在帮助用户回答有关 Azure OpenAI 服务的技术问题。仅使用下面的上下文回答问题，如果您不确定答案，您可以说“我不知道
- [System message]Assistant是一款智能聊天机器人，旨在帮助用户回答与税务相关的问题。
- [System message]你是一个助手，旨在从文本中提取实体。用户将粘贴一串文本，您将使用从文本中提取的实体作为 JSON 对象进行响应。以下是输出格式的示例
  ```{     "name": "",   "company": "",   "phone_number": "" }```

需要注意的是，即使利用系统消息示意模型在遇到不确定答案的问题时时，回答“**我不知道**”；这也不能保证模型在此情况下百分百回答“我不知道”，还记得吗？LLM只是一个概率模型。

:bulb:PS：如果你使用的API不支持格式化输入输出，则可根据[使用 Azure OpenAI 提示工程技术](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/advanced-prompt-engineering?pivots=programming-language-completions#non-chat-scenarios)此文章学习。两篇文章核心内容大致相同，只是对例子的格式有所区别，因为不支持格式化输入输出的API不具有System message项。

## Few-shot learning 小样本学习

使语言模型适应新任务的一种常见方法是使用小样本学习。在小样本学习中，一组Example作为提示的一部分提供，以便为模型提供额外的上下文。具体例子可见Introduction to Prompt中的Example。

## Start with clear instructions 从明确的指令开始&Repeat instructions at the end 在最后重复说明

研究表明，将Instruction放在用户消息的开头，往往能获得更高质量的回答。除此之外，模型可能容易受到**Recent bias**的影响，尤其是在content内容较长的情况下，这意味着提示末尾的信息可能比提示开头的信息对输出的影响更大。因此，在Prompt结束时重复这些instruction，例如：

- [System message]\: 你是一个人工智能助手，可以帮助人们找到信息。
  - [User message]:
  - [Instruction]你的任务是验证以下消息中是否隐含或直接说明了：“几个消息来源提到另一次大规模喷发的可能性”这一陈述。
  - [content]
    - 1.专家称，西雅图发生特大地震的可能性为14%...
    - 2.地震专家对西雅图的“真正大地震”进行了最新展望...
  - [Re-Instruction]以上两个片段消息中，是否直接提及活隐含了：“几个消息来源提到另一次大规模喷发的可能性”。

## Prime the output 启动输出

类似Introduction to Prompt中的Cue，例如：

- [System message]你是一个人工智能助手，可以帮助人们找到信息。用尽可能少的词回答。
- [User message]约翰·史密斯与露西·史密斯结婚。他们有五个孩子，他在Microsoft担任软件工程师。(Instruction)我应该做哪些搜索查询来对此进行事实核查？(Cue)## 一个可能的搜索查询是：

在上面的例子中，"*一个可能的搜索查询是：*"用于引导模型以产生单个输出。如果没有此提示，模型将生成多个搜索查询作为输出。

## Add clear syntax 添加清晰的语法

对提示使用清晰的语法（包括标点符号、标题和部分标记）有助于传达意图，并且通常使输出更易于分析。在下面的示例中，在不同的信息源或步骤之间添加了分隔符（在本例中为 `---`）。这允许使用 `---` 作为停止生成的条件。此外，章节标题或特殊变量以大写形式显示以区分它们。

- [System message] 你是一个人工智能助手，可以帮助人们找到信息。
- [User message]
  - [Instruction]你将阅读一段话，然后向搜索引擎发出查询，以便对其进行事实核查。还要说明查询。
  - [content]
    \---
    约翰·史密斯与露西·史密斯结婚。他们有五个孩子，他在Microsoft担任软件工程师。我应该做哪些搜索查询来对此进行事实核查？
    \---
  - 查询

如果不确定要使用什么语法，请考虑使用 Markdown 或 XML格式。因为模型已经在 XML 和 Markdown 中的大量 Web 内容上进行了训练，因此可能会提供更好的结果。

## Break the task down 分解任务

大型语言模型（LLMs）如果任务被分解成更小的步骤，通常会表现得更好。例如，在前面提到的搜索查询提示中，可以重构提示，使得模型首先被指示提取相关事实，然后被指示生成可以用于验证这些事实的搜索查询。

- [System message]你是一个人工智能助手，可以帮助人们找到信息。
- [User message]
  - [Instruction]你将阅读一段话，然后向搜索引擎发出查询，以便对其进行事实核查。
  - [content]
    \---
    约翰·史密斯与露西·史密斯结婚。他们有五个孩子，他在微软担任软件工程师。我应该做什么搜索查询来核实这一点？
    \---
  - [Re-Instruction]现在，您将提取事实声明，并发出查询来对它们进行事实检查。当发出查询时，使用函数“查询”（“query”）
    ([Cue])事实是：
- [Model Output]
  - 1 约翰·史密斯和露西·史密斯结婚了
    2 他们有五个孩子
    3 他是微软的软件工程师
  - 查询
    SEARCH("约翰·史密斯与露西·史密斯结婚")
    SEARCH("约翰·史密斯的家庭成员情况")
    SEARCH("约翰·史密斯，微软软件工程师")

## Use of affordances 使用引用(Supporting content)

affordances，引用，即在content中加入有帮助于回答的信息。在上述例子中，搜索结果可以是一个启示，以帮助减轻伪造的答案，并获得最新的信息，是Introduction to Prompt Engineering中Supporting content里辅助模型回答问题的信息。

| System message                                             | User                                                         | Assistant                                                    |
| :--------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| You're an AI assistant that helps people find information. | You'll read a paragraph, extract factual claims, and then use search engine results to fact-check them<br /> --- <br />PARAGRAPH <br />John Smith is married to Lucy Smith. They have five kids, and he works as a software engineer at Microsoft. What search queries should I do to fact-check this? <br />--- <br />FACTUAL CLAIMS <br />- John Smith is married to Lucy Smith <br />- John and Lucy have five kids <br />- John works as a software engineer at Microsoft <br />--- <br />Here are various search queries issued to research the claims above: <br />SEARCH QUERIES <br />- John Smith married to Lucy Smith <br />- John Smith number of children <br />- John Smith software engineer Microsoft <br />--- <br />Here are snippets of the search results: <br />SNIPPETS: <br />[1] John Smith’s wedding was on September 25, 2012 … <br />[2] John Smith was accompanied by his wife Lucy to a party <br />[3] John was accompanied to the soccer game by his two daughters and three sons <br />[4] After spending 10 years at Microsoft, Smith founded his own startup, Tailspin Toys<br />[5] John M is the town smith, and he married Fiona. They have a daughter named Lucy <br />---<br />Given the snippets, fact check each of the factual claims above: | - John Smith is married to Lucy Smith: Confirmed. Snippet [1] and [2] both mention John Smith's wife as Lucy.  <br />- John and Lucy have five kids: Confirmed. Snippet [3] mentions that John was accompanied by his two daughters and three sons.  <br />- John works as a software engineer at Microsoft: Partially confirmed. Snippet [4] mentions that John spent 10 years at Microsoft, but it doesn't explicitly state that |

## Chain of thought prompting 思维链

思维链是**分解任务**技术的变体。在这种方法中，不是将任务拆分为更小的步骤，而是指示模型响应逐步进行并呈现所涉及的所有步骤。这样做减少了结果不准确的可能性，并使评估模型响应更容易

| System message 系统消息                  | User 用户                                                    | Assistant 助理                                               |
| :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 你是一个人工智能助手，帮助人们找到信息。 | 在悉尼奥运会上获得奖牌最多的个人运动员是谁？在你的回答中采取一步一步的方法，引用来源并给出推理，然后以下面的格式分享最终答案：答案是：`<name>` | 第一步：确定悉尼奥运会的举办年份。悉尼奥运会于2000年举行。 <br />第二步：找出在2000年悉尼奥运会上获得多枚奖牌的运动员个人名单。  <br />据奥运会官方网站介绍，2000年悉尼奥运会上有多名个人运动员获得多枚奖牌。一些著名的名字包括： <br />• 伊恩·索普（澳大利亚）- 3金、2银、0铜<br /> • Pieter van den Hoogenband（荷兰）—2金，0银，2青铜 <br />• Lenny Krayzelburg（美国）—3金牌，0银，0青铜<br /> • Inge de Bruijn（荷兰）- 3金、1银、0铜  <br />第三步：确定获得奖牌最多的运动员。回顾2000年悉尼奥运会上获得多枚奖牌的个人运动员名单，我们可以看到，获得奖牌最多的运动员是来自澳大利亚的伊恩·索普，他共获得5枚奖牌（3金2银银）。<br />因此，答案是：伊恩·索普。 |

## Specifying the output structure 指定输出结构

使用提示符指定输出的结构会对结果产生重大影响。有时，System message输入“只写真实事实”或“不要捏造信息”并不足够。相反，要求模型响应也包括引用可以帮助减少不正确响应的发生率。
一旦要求LLM根据supporting content回答问题，那么模型就有可能犯两个错误，一是捏造回答，二是回答中使用了错误的supporting content。对于此问题，我们发现模型的问题和其对应的Supporting content的文本距离越近，上述两种错误的发生的概率越低。因此对于多段Support content和多个问题，最好的方式是直接在每个问题附近加入对应的supporting content，即内联引用。
同样，如果您要求模型从段落中提取事实陈述，它可能会提取复合陈述，例如“X 正在执行 Y AND Z”（这可能更难验证）。这可以通过指定输出结构（例如（实体1，关系，实体2））来避免。例如：

| System message 系统消息                | User 用户                                                    | Assistant 助手                                               |
| :------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 你是一个帮助人们查找信息的人工智能助手 | 列出客服在以下对话中表现出的不同行为。<br />---<br />您好，感谢您给 contoso 药房打电话，今天我在和谁通话，我理解您的沮丧，玛丽，您能告诉我您到底想实现什么目标吗？好的，呃，我当然很高兴为您检查一下，请我在这里看到您正在通话中。立普妥的通用形式就是这样，好吧呃，所以我看到你的医生在二十一二十一年内停止开这些药物实际上好吧呃是的，我很乐意为你检查一下呃，因为有时系统中存在差距，它只是没有被报告所以让我看一下这里所以我在这里看到你的医生在二十二点到二十一点之间给你开了这些药物好吧嗯是的根据医生办公室提供的最新记录呃您现在正在服用另一种药物二甲双胍，您希望我们继续为您配药吗？取一点五百毫克的形式，呃，不客气，玛丽，如果还有什么我们可以做的，请告诉我们今天为你做。<br /> --- <br />客服的反应如何？ <br />使用 BEHAVIOR("reason") 格式对他们的各种行为进行分类并列出，并引用分类背后的原因 | 1. 礼貌问候行为（“客服人员以礼貌的问候开始对话，并询问呼叫者的姓名”） <br />2. 善解人意的行为（“特工承认玛丽的沮丧并试图理解她的担忧”） <br />3. 细心的行为（“代理倾听客户的问题并询问更多细节”） <br />4. 主动行为（“代理主动为客户检查问题”） <br />5. 信息性行为（“代理提供有关客户处方历史的相关信息”） <br />6. 彻底的行为（“代理检查系统中任何可能的差距并提供完整的响应”）<br />7. 确认行为（“代理与客户确认当前药物”） <br />8. 以解决方案为导向的行为（“代理人主动为客户配药”）<br />9. 礼貌行为（“客服人员感谢客户，并在需要时提供进一步的帮助”） |

| System message 系统消息                      | User 用户                                                    | Assistant 助手                                               |
| :------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 你是一个人工智能助手，可以帮助人们查找信息。 | 根据文本中看到的模式将以下新闻标题聚类为主题类别。还要提及如何定义这些类别背后的推理。<br /> 输出格式： { <br />"TOPIC_NAME": "", <br />HEADLINES": [], <br />"REASONING": ""<br />} <br /> 输入新闻标题： <br />1.“利用 AR + ML 从书籍到演示仅需 10 秒” <br />2.“1993 年 32 岁的 Yann LeCun 展示世界上第一个用于文本识别的卷积网络的演示” <br />3.“一阶运动模型应用于动画绘画” <br />4.“Robinhood 和其他经纪人确实阻止购买 $GME、$NOK、$BB、$AMC；允许出售” <br />5.“美联航股价盘前交易下跌超过 5%” <br />Output: 输出： | { <br />"ARTIFICIAL_INTELLIGENCE": <br />{ "HEADLINES": [<br />“通过 AR + ML 从书籍到演示仅需 10 秒”，<br />“1993 年 32 岁的 Yann LeCun 展示了世界上第一个用于文本识别的卷积网络的演示”， <br />“一阶运动模型应用于动画绘画” <br />], <br />“REASONING”：“这些头条新闻与人工智能、机器学习及其在各个领域的应用有关。” }, "FINANCIAL_MARKETS": <br />{ "HEADLINES"：[<br /> “Robinhood 和其他经纪人实际上阻止购买 $GME、$NOK、$BB、$AMC；允许出售”， <br />“联合航空股价盘前交易下跌超过 5%”，<br />“一年前的今天，比特币价格接近 20,000 美元” ], <br />"REASONING": “这些头条新闻与金融市场、股票、加密货币和交易平台有关。” <br />} <br />} |

## Temperature and Top_p parameters 温度和 Top_p 参数

对于Temperature，更高的值意味着模型的回答更随机，反之则更确定
对于Top_p，越高意味着回答丰富性越高，反之则越稳定

具体原理可看：[大模型推理常见采样策略：Top-k, Top-p, Temperature, Beam Search_top-p采样-CSDN博客](https://blog.csdn.net/qq_43243579/article/details/136331123)

## Provide grounding context 提供基础背景

注意，grounding context一般在System message中，被模型当做基本信息而不是用户的输入使用，例如：

| System message 系统消息                                      | User 用户                                      | Assistant 助手                                               |
| :----------------------------------------------------------- | :--------------------------------------------- | :----------------------------------------------------------- |
| <br />你是一个人工智能助手，可以帮助人们查找信息。您将仅从以下文本中提供答案，并以 100 字或更少的字数进行回复： <br />在 Microsoft，我们不断发现释放创造力、提高生产力和提升技能的新方法，以便更多的人可以从使用 AI 中受益。这使我们的客户能够……  <br />…从而降低模型产生有害反应的可能性。” <br />回答在 100 字以内。尽可能使用无序列表。 | Azure OpenAI 服务 GPT-4 模型的早期客户有哪些？ | 以下是 Azure OpenAI 服务 GPT-4 模型的一些早期客户： <br />- Epic Healthcare：他们计划使用 GPT-4 帮助医生和护士减少在数据输入和调查上花费的时间。 <br />- Coursera：他们使用 Azure OpenAI 服务和 GPT-4 在其平台上创建新的人工智能驱动的学习体验。 <br /> - 可口可乐：他们正在使用 Azure OpenAI 构建知识中心，并计划利用 GPT-4 的多模式功能进行营销、广告、公共关系和客户关系。 <br />- 其他各种规模的公司都在使用 Azure AI 来改善客户体验、总结长篇内容、帮助编写软件并通过预测正确的税务数据来降低风险。 |
