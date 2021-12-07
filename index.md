## Crawl for comments on JD

由于小组作业的要求，我需要在京东平台上爬取大疆mini2的评论，使用jupyter选取前100页，共1000条评论，并制作成为词云，以供进一步分析


### 代码部分

```markdown

# 先下载各个包
import requests
import json
import pandas as pd
import time
import csv
import re
# 获得网址和伪装
url = 'https://club.jd.com/comment/productPageComments.action?callback=fetchJSON_comment98&productId=100009034013&score=2&sortType=5&page=2&pageSize=10&isShadowSku=0&fold=1'
header = {'user-agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36'}
# 因为要翻页所以写一个循环，这里每一行对应着网址里面的每一个要素
for i in range(100):
    data={'callback':'fetchJSON_comment98',
         'productid':100009034013,
         'score':0,
         'sortType':5,
         'page':i,
         'pageSize':10,
         'isShadowSku':0,
         'fold':1}
    html=requests.get(url,params=data,headers=header)
# 解析并处理爬到的内容
html_json=re.search('(?<=fetchJSON_comment98\().*(?=\);)',html.text).group(0)##正则表达式查找内容
    j=json.loads(html_json)
    for jj in range(len(j['comments'])):
        data1=[(j['comments'][jj]['content'])]
        data2=pd.DataFrame(data1)
        data2.to_csv('1.txt',header=False,index=False,mode='a+')
    time.sleep(2)##每一页结束后停顿两秒
    print('page'+str(1+i)+'has done')
# 下载中文分词的库
!pip install jieba
import jieba
# 从网上下载中文停止词列表，并设置停止词，这里下载的是百度版本，但是显然没有很好用
def stopwordslist():
    stopwords={line.strip()for line in open('stopwordcn.txt',encoding='UTF-8').readlines()}
    return stopwords
# 定义分词的公式
def seg_depart(sentence):
    print('正在分词')
    sentence_depart=jieba.cut(sentence.strip())
    stopwords=stopwordslist()
    outstr=''
    for word in sentence_depart:
        if word not in stopwords:
            if word!='\t':
                outstr +=word
                outstr +=' '
    return outstr
# 建立输入和输出文档
filename='1.txt'
outfilename='2.txt'
inputs=open(filename,'r',encoding='UTF-8')
outputs=open(outfilename,'w',encoding='UTF-8')
# 开始分词
for line in inputs:
    line_seg=seg_depart(line)
    outputs.write(line_seg+'\n')
    print('在干活啦')
outputs.close()
inputs.close()
print('一切顺利')
# 导入词云的库
from wordcloud import WordCloud
import jieba
import numpy as np
import PIL.Image as Image
# 把已经分好词的文本打开
text=open('2.txt',encoding='utf8').read()
# 设置词云格式，新增停止词，输出并展示词云
wordcloud = WordCloud(font_path='C:/simfang.ttf',
                      width=800,
                      height=600,
                      background_color='white',
                      max_font_size=150,
                      max_words=200,
                      stopwords={'没','说','问','问问','突然','京东','无人机','买','大疆','没有','飞','特地','很多','有点','想','之前'},
                      collocations=False).generate(text)
image=wordcloud.to_image()
wordcloud.to_file('jd1.png')
image.show()
```
### 词云输出结果

![jd1](https://user-images.githubusercontent.com/95664136/144959917-94dcb608-4f55-4901-a150-8050cd1b5403.png)

好的，下次再会
