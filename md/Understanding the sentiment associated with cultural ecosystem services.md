| **Title** | Understanding the sentiment associated with cultural ecosystem services using images and text from social media |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Ilan Havinga |
| **Institution** | Wageningen University  |
| **Venue** | Ecosystem Services |
| **Year** | 2024 |


# ABSTRACT


# Introduction
- CES
  - 定义
    - 指生态系统通过人类与自然的互动所提供的非物质利益。这些利益包括精神愉悦、审美享受、文化认同和社会联系等，与人类的心理和社会福祉密切相关
  - 与社交媒体的关系
    - 社交媒体上拥有大量的图像和文本数据，因此，现在是 CES 的重要数据来源之一
  - 目前缺陷
    - 然而，当前的研究大多依赖于简单的数量指标，未能充分捕捉用户与自然互动的复杂性
- 本文方法


1. 什么是CES
2. 当前的 CES 的数据来源往往只是统计图片的数量，但这种方法无法准确反映不同自然体验的真实价值

1. 使用社交媒体的文本和图像数据可以获得哪些CES相关的人类-自然互动测量？
2. 不同的物理环境和活动如何与这些互动产生的情感相关？
3. 情感和CES测量是否与国家幸福感调查测量结果匹配？通过这样做，我们旨在广泛理解情感分析和社交媒体作为测量CES的有效工具，特别是来自人类-自然互动（包括景观美学和野生动物观察）的积极体验（Langemeyer和Calcagni, 2022）

# 2. METHOD
- 这篇论文聚焦于两个 CES： landscape enjoyment 和 cultural appreciation of wildlife，并通过分析 `Flickr` 的图片和文本数据，得到这两个 CES 指标
- 数据集和模

## 2.1. Study Design
- 这篇论文聚焦于两个 CES
  - 景观欣赏：审美质量
    - 在 `ScenicOrnot` 数据集上预训练的模型，这个数据集中包含了在英国的景观图片，从而得到人们对于景观的偏好
  - 野生动物文化鉴赏：摄影观察野生动物
    - 在 `iNaturalist` 数据集上的预训练的物种识别模型，用于估算 `Flickr` 图片中的野生动物数量
- 美学估算
  - 



- Dataset
  - contans 9.8 million outdoor images on Flickr platform

## 2.2. 数据集

## 2.3. Text data processing and models for sentiment analysis
- 对数据集进行结构化处理
  - `Hedonometer` 模型输入的需求
1. Hedonometer
2. Sentiment140
3. RoBERTa
4. Adjective analysis


## 2.4. Image analysis for CES predictions
1. 景观享受的美学质量评估
   - 1. 使用 `ScenicOrNot` 数据集进行训练，其包含大量图像和美学评分
   - 2. 生成 `Flickr` 中每张图片的美学评分
2. 野生动物文化欣赏的二分类
   - 1. 使用 `iNaturalist` 数据集进行训练，用于区分其每张图片是否包含野生动物
   - 2. 识别 `Flickr` 中每张图片是否含有野生动物
   - 3. 选择数据集中最常出现的五个类别进行**进一步分析**


## 2.5. Sentiment associated with physical settings and activities
利用 `Places365` 进行场景和属性预测，以理解不同环境对情感体验的影响
  - 场景：i.e., 森林、湖泊
    - ![alt text](<images/Understanding the sentiment associated with cultural ecosystem services/img-3.png>)
  - 属性：i.e., 建筑、道路 ，基于 `SUN` 数据库
    - ![alt text](<images/Understanding the sentiment associated with cultural ecosystem services/img-4.png>)


## 2.6. MENE suryvey comparison
- MENE 调查的背景
  - 是一项**具有代表性**的英格兰居民自然休闲生活的调查
  - 内容涉及受访者的自然旅行活动和其带来的幸福感
- 第一层次对比：**旅行动机**和 **Flickr 的数据**
  - MENE
    - 欣赏风景：是/否
    - 欣赏野生动物：是/否
  - Flickr
    - 基于 MENE 受访者的旅行的地理坐标，将 Flickr 图像中 1 公里半径范围内情感、美学质量和野生动物观察数量作为比较数据
- 第二层对比：**幸福描述**和**指标差异**
  - MENE
    - 旅行者的幸福感的描述
  - Flickr
    - 以 MENE 中对于幸福程度的描述为比较点，分析不同描述在 Flickr 数据中的情感、美学质量和野生动物观察数量的差异
- 目的（我的观点）
  - 验证社交媒体数据的有效性


# 3. Result
## 3.1. Sentiments associated with CES
## 3.1.1. Aesthetic quality
![alt text](<images/Understanding the sentiment associated with cultural ecosystem services/img.png>)
- 1. 情感预测模型的整体趋势
  - 三个模型均表明**情感和景观美学评分之间呈正相关** >> 即美学评分越高，用户的情感越积极
- 2. 模型之间的差异
  - `Hedonometer` 与 `MENE` 调查结果的相关性最高，整体表现最好 >>  `Hedonometer` 可能是最适合的模型
- 3. 形容词的使用模式
  - 随着美学评分的增加，正向形容词的使用频率增加；评分减少，反向形容词的使用频率增加
- 4. 正负形容词的比例
  - 正面情感远多于负面情感 >> 人们倾向于表达积极的情绪


## 3.1.2. Wildlife observations
![alt text](<images/Understanding the sentiment associated with cultural ecosystem services/img-2.png>)
![alt text](<images/Understanding the sentiment associated with cultural ecosystem services/img-1.png>)
- 1. `Hedonometer` 模型的感情相关性
  - 在三个 NLP 模型中，`Hedonometer` 与 `MENE` 的相关性最高，但是总体相关性略低，为 0.01-0.06 之间，但仍具有统计学意义（p <0.001）
- 2. 植物产生最积极的情感
  - 尤其是 `Asparagales` 
- 3. 不同物种的词汇特点
  -  `tf-idf` 表现了用户在描述不同物时使用的独特形容词，积极的物种伴随着更多的正面词汇
     -  竖轴为词汇，横轴为出现的频率
  

## 3.2. 



## 3.3. Comparison with surveyed well-being measures
- 基于 `Flicker` 的数据和 `Hedonometer` 的模型计算的 `CES` 与 `MENE` 的对比和联系
- 1. 旅行目的和美学质量的关系
  - 
- 2. 野生动物观察与情感