---
layout: post
title:  爬取QQ空间的一种办法
date:   2019-09-08 19:55:00 +0800
categories: note
tag: 项目
---

* content
{:toc}



最近因为工作需求，需要统计一些qq号的说说浏览量、点赞数、评论数，人工统计工作量还是比较大的，而且比较枯燥，所以使用了python里的相关工具。最后的结果我还是比较满意的，这里把我研究的过程写出来分享给大家，共同学习交流。

### 结果
首先谈谈结果吧。爬取的数据包括：
+ 好友说说内容（不包括图片、视频、表情等）
+ 点赞数、评论人数、浏览量

也存在一些不足：
+ 很难确定是否漏爬了说说
+ 效率较低，如果想爬取大量说说，建议寻找其他方法

![结果](https://img-blog.csdnimg.cn/20190908004432581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxNzQzMTk1,size_16,color_FFFFFF,t_70)

### 工具
 用到的库：
 + selenium：我在其他博客上找到的qq空间爬虫博文几乎都用到了这个库，它可以用来模拟登陆qq空间。有点劝退新人，但是接触下来还是很有意思的一个库。主要用到的是里面的webdriver模块，可以用代码自动控制一个浏览器。
 ```python
from selenium import webdriver
```

+ re：正则模块，对于beautifulsoup模块我还是不太熟练，所以采用了正则表达式提取想要的数据

+ pandas：由于数据量不大，采用excel表格的形式保存数据，用到了pandas库里的一个函数。还可以用pandas库进行一些关于说说的数据分析工作。
+ time：发送延迟，一方面使页面完整加载出来，另一方面防止频繁操作被系统误封

### 思路
 一开始做这个事的时候，我的想法很简单，**右键-查看网页源代码-复制-正则提取**，还别说，这样做也不是完全不可行，我成功的得到了5条说说的数据，当时我还几乎以为自己成功了，我天真的以为可以拉5条，就能用同样的方法拉50条，拉500条，可是显然，qq空间的数据可没这么好拿，它的加载方式跟传统的‘上一页’,‘下一页’不太一样，属于下拉式加载，这跟手机端的有点类似。更烦人的事情是，就算你下拉了好几页，查看网页源代码还是只能看到一部分，很可能只有5、6个，我用尽浑身解数把这个数字提升到了18个，对于我来说虽然也够用了，但我还是想有没有办法获取更多数据，并让这个过程更自动化。
 于是我使用了webdriver模块，让程序帮我翻页、获取源码，再用正则去提取数据。其实思路还是一样的，只是获取网页源码的方式更加智能。

### 关键代码


1. **自动登录qq空间**

 ```python3
chrome = webdriver.Chrome()#打开自动控制的谷歌浏览器。需要给webdriver配置环境变量，也可以指定
chrome.get("https://qzone.qq.com/")#打开网页
chrome.switch_to_frame('login_frame')#切换至登陆框
chrome.find_element_by_xpath('//*[@id="switcher_plogin"]').click()#切换使用账号密码登陆
time.sleep(1)
chrome.find_element_by_xpath('//*[@id="u"]').send_keys('210****719')#发送账号数据
time.sleep(2)
chrome.find_element_by_xpath('//*[@id="p"]').send_keys('********')#发送密码数据
time.sleep(2)
chrome.find_element_by_xpath('//*[@id="login_button"]').click()#点击登陆按钮
time.sleep(5)
```
+ 要点：
 使用 XPATH 定位网页元素，详见我后面附上的参考博文中的第一个

2. **控制垂直滚动条**
```python3
js="var q=document.documentElement.scrollTop=100000"
chrome.execute_script(js)
```
+ 要点：
用到了 js，这个我也是在网上找到的一种将页面滑倒最底部的方法

3. **获取网页源代码**

 ```python3
text = chrome.page_source
```

4.**正则提取**
 我没有系统地学过正则表达式，基本上是用到哪里学哪里，所以总感觉自己写的有点粗糙，目前存在的一个问题是当爬取说说的数目达到一定数目时点赞量会不对，后面有时间再去解决吧。
 ```python
def qqzone(text):    
	"""
    输入text是qq空间网页源代码，
    函数功能是提取好友说说及其浏览量、点赞量、评论人数
    得到的结果使用pandas库中的dataframe存放，可存入excel文件
	"""
    resList = re.findall(r'<li class=\"f-single f-s-s\".*?\s+</div></li>',text)
    df_res = pd.DataFrame(columns = ['qq','备注','时间','内容摘要','点赞数','浏览量','评论数'])
    index = 0
    for elem in resList:
        account = re.findall(r'http://user.qzone.qq.com/(\d+)',elem)
        df_res.loc[index,'qq'] = account[0]
        time = re.findall(r'>\s*(.{0,6}\s*\d\d:\d\d)<',elem)
        df_res.loc[index,'时间'] = time[0]
        #name = re.findall(r'',elem)
        df_res.loc[index,'备注'] = '备注暂不显示'
        good = re.findall(r'>(\d+).*人觉得很赞',elem)
        print(good)
        df_res.loc[index,'点赞数'] = good[-1]
        look = re.findall(r'浏览(\d+)次',elem)
        df_res.loc[index,'浏览量'] = look[0]
        df_res.loc[index,'评论数'] = len(time)-1
        content = re.findall(r'<div class=\"f-info\">.+\s*</div>|<div class=\"f-info qz_info_cut\">.+展开全文',elem)
        try:
            content = re.findall('[\u4e00-\u9fa5]+',content[0])
            content = ''.join(content)
        except:
            content = ['']
            print(content,elem)
        df_res.loc[index,'内容摘要'] = content
        index += 1
    return df_res
```

5.**去重**
在翻页的过程中可能会采集同一个说说多次，为了不让它多次出现在结果中，需要去除重复的说说
```python
for i in range(0,df_res.时间.size):
    qq = df_res.loc[i,'qq']
    时间 = df_res.loc[i,'时间']
    for j in range(0,i):
        try:
            if qq == df_res.loc[j,'qq'] and 时间 == df_res.loc[j,'时间']:
                df_res = df_res.drop(i,axis=0)
                break
        except:
            print(j,'已被删除')
```

### 改进
对于滚动过程中可能出现的爬取丢失问题，可以进入单个的好友空间，获取<tb>元素, 遍历取值。由于好友空间更接近翻页模式，所以采集信息几乎不会丢失 --2020-03

### 完整代码
```python3
from selenium import webdriver
import time
import re
import pandas as pd

def save_as_excel(df,xlsx_name='a.xlsx',sheet_name='sheet1',startrow=0,startcol=0):
    """
    功能是保存一个df对象为excel文件(xlsx)
    df：要保存的dataFrame对象
    xlsx_name:要保存的名字，注意加上后缀
    sheet_name:要保存的表的名字
    """
    writer=pd.ExcelWriter(xlsx_name)
    df.to_excel(writer,sheet_name = sheet_name,startrow=startrow,startcol=startcol,index=False)
    writer.save()

def qqzone(text):    
	"""
    输入text是qq空间网页源代码，
    函数功能是提取好友说说及其浏览量、点赞量、评论人数
    得到的结果使用pandas库中的dataframe存放，可存入excel文件
	"""
    resList = re.findall(r'<li class=\"f-single f-s-s\".*?\s+</div></li>',text)
    df_res = pd.DataFrame(columns = ['qq','备注','时间','内容摘要','点赞数','浏览量','评论数'])
    index = 0
    for elem in resList:
        account = re.findall(r'http://user.qzone.qq.com/(\d+)',elem)
        df_res.loc[index,'qq'] = account[0]
        time = re.findall(r'>\s*(.{0,6}\s*\d\d:\d\d)<',elem)
        df_res.loc[index,'时间'] = time[0]
        #name = re.findall(r'',elem)
        df_res.loc[index,'备注'] = '备注暂不显示'
        good = re.findall(r'>(\d+).{0,10}人觉得很赞',elem)
        print(good)
        df_res.loc[index,'点赞数'] = good[-1]
        look = re.findall(r'浏览(\d+)次',elem)
        df_res.loc[index,'浏览量'] = look[0]
        df_res.loc[index,'评论数'] = len(time)-1
        content = re.findall(r'<div class=\"f-info\">.+\s*</div>|<div class=\"f-info qz_info_cut\">.+展开全文',elem)
        try:
            content = re.findall('[\u4e00-\u9fa5]+',content[0])
            content = ''.join(content)
        except:
            content = ['']
            print(content,elem)
        df_res.loc[index,'内容摘要'] = content
        index += 1
    return df_res

chrome = webdriver.Chrome()
chrome.get("https://qzone.qq.com/")
chrome.switch_to_frame('login_frame')
chrome.find_element_by_xpath('//*[@id="switcher_plogin"]').click()
time.sleep(1)
chrome.find_element_by_xpath('//*[@id="u"]').send_keys('2104907719')
time.sleep(2)
chrome.find_element_by_xpath('//*[@id="p"]').send_keys('19994235szj')
time.sleep(2)
chrome.find_element_by_xpath('//*[@id="login_button"]').click()
time.sleep(5)
js="var q=document.documentElement.scrollTop=100000"

res_list = []
for i in range(0,100):
	#这里的100是指下拉100次   
    chrome.execute_script(js)
    time.sleep(2)
    text = chrome.page_source
    if i%3 == 0:
    #这里的意思是每下拉三次获取一次网页源码
        qqzone(text)
        res_list.append(qqzone(text))
        
df_res = pd.concat(res_list,sort=False)#拼接
df_res = df_res.reset_index(drop = True)#重新设定索引

for i in range(0,df_res.时间.size):
    qq = df_res.loc[i,'qq']
    时间 = df_res.loc[i,'时间'] #python支持用中文作为变量名
    for j in range(0,i):
        try:
            if qq == df_res.loc[j,'qq'] and 时间 == df_res.loc[j,'时间']:
                df_res = df_res.drop(i,axis=0)
                break
        except:
            print(j,'已被删除')
df_res = df_res.reset_index(drop= True)
save_as_excel(df_res,'qq统计.xlsx')
```

### 参考资料
+ https://www.cnblogs.com/hellosecretgarden/p/9206648.html
+ https://blog.csdn.net/After__today/article/details/81153203

 （个人感觉这两篇博文写得都很不错~）
