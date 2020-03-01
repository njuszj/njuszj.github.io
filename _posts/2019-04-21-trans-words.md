---
layout: post
title:  C++学习笔记：单词转换程序
date:   2019-04-21 01:08:00 +0800
categories: note
tag: C/C++
---

* content
{:toc}




### 单词转换程序
在*C++primer 5th*第11章(关联容器)的末尾给出了一个单词转换程序，我觉得非常有意思。里面运用了很多关联容器和IO处理的基本操作，虽然书上已经给出了这个程序的源代码，但我还是准备在这里贴出来分析一下，讲一些我自己的理解。

 功能：从一个文本文件中读取转换规则，比如：
 + u you
 + hs handsome
 + y yes
 + i I
 + k know
 
 将另一个文本中所有可以被替换的单词替换，比如：
  *原文本*
 + u are so hs
 + y i k i am hs

  *转换后的文本*
+ you are so handsome
+ yes I know I am handsome

 很有意思，不是吗？现在我们来考虑怎么去实现这个程序。
 + 我们需要一个函数建立一个转换映射--即建立一个map，键key是待转换文本，值value是转换后的文本
 + 需要一个函数靠上面创建的map进行实际转换工作
 + 最后还要一个函数处理文件的读取以及调用上面两个函数
 
---
以下程序可以在C++primer第五版第391-393页找到，但是这里的注释是我自己写的
---


 ```cpp
map<string, string> buildMap(ifstream &map_file) {
//建立转换映射，将转换规则存入一个map并返回该map
	map<string, string> trans_map;//定义一个保存转换规则的map
	string key;//map的key，待转换的单词
	string value;//map的value，转换的结果
  while(map_file>>key&&getline(map_file,value))//每次循环key获得“转换规则”文本中一行的首单词，即待转换的单词，value获得该行剩下的内容，即“空格+目标单词”																						
		if (value.size() > 1)//防止只有一个单词
			trans_map[key] = value.substr(1);//去除空格，建立映射
		else
			throw runtime_error("no rule for " + key);//抛出异常信息
	return trans_map;
}
 ```

```cpp
const string &transform(const string &s, const map<string, string> &m) {
//转换工作，输入需要转换的string和存有转换规则的map，返回转换后的string
	auto map_it = m.find(s);//查找是否存在需要转换的单词
	if (map_it != m.cend())	//如果找到了，进行转换
		return map_it->second;
	else
		return s;//否则原样反回
}
```


```cpp
void word_transform(ifstream &map_file, ifstream &input) {
//这个函数是与外部文本的接口
	auto trans_map = buildMap(map_file);//调用函数，建立映射
	string text;
	while (getline(input, text)) {	//将待转换文本的一行存入text
		istringstream stream(text);	//text与一个string输入流绑定
		string word;
		bool firstword = true;//控制是否打印空格
		while (stream >> word) {   //word将通过循环，保存text中的每个单词
			if (firstword)//开头不需要空格，其他位置单词之间需要空格
				firstword = false;
			else
				cout << " ";
			cout << transform(word, trans_map);	//调用transform函数对读取的单词进行转换
		}
		cout << endl;
	}
}
```


### 思考
 经过测试，转换完成了目的。而且即使读入的是中文字符，转换也能完成，但是中文字之间显然是没有空格的，所以程序需要进行修改。还有一个问题是，即使要转换的是纯英文字符，遇到了标点符号等也会出现无法转换的情况，解决这个问题要么把单词连着标点输进转换规则，要么就修改程序。

### 一些重要的知识点
+ **getline(istream,line)**  读入一整行，即遇到回车结束，属于string头文件
+ **istream >> strword** 读入一个单词，即遇到空格结束
+ **map**的查找函数**find**查找键值，返回一个迭代器，若没有找到，则返回尾后迭代器

### 参考资料
+ *C++ primer 5th*

