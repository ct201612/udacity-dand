## P6:Tell Stories with Data Visualization

在此项目中，你将使用一个数据集创建一个数据可视化，用于表明数据的状况或突出它的趋势或模式。你将需要使用 dimple.js 或 d3.js 创建这个可视化。你需要思考数据可视化的理论和实践，如视觉编码、设计原则和有效沟通。

* 展现出选择最佳视觉元素来编码数据和批判性评估可视化效果的能力
* 使用交互式可视化向合适的受众讲述故事或发现
* 经历迭代式可视化创建流程和使用 dimple.js 或 d3.js 建立交互式可视化

## 说明
要开始数据可视化项目，首先要启动计算机上的本地服务器。要启动本地 web 服务器，计算机上必须安装了 Python 2.7.8 或更高版本；
在DOS里cd到准备做服务器根目录的路径下，输入命令：

* python -m Web服务器模块 [端口号，默认8000]

* python -m SimpleHTTPServer 8080

然后就可以在浏览器中输入

* http://localhost:端口号/路径

来访问服务器资源。 

### 概要
这个可视化项目，我选择泰坦尼克号的数据集；这数据集包括891名乘客信息，如Survived，Pclass，Name，Sex等；这个可视化的目的是，展示乘客基于Sex（male和female）和Pclass（1，2，3）的存活率和死亡率的相关统计：女性比男性的存活率高，3个级别的船舱中class1（头等舱）存活率最高，其次是class2，class3存活率最低；

### 设计
 1.因为性别（Sex）和船舱等级（Pclass）都是类别型，所以选择bar图；
 2.选择百分比堆积条形图清楚显示乘客存活率(粉红色)和死亡率(浅蓝色)；
 3.为了提高图的可读性，修改y轴名，乘客的状态改为Survived/Deceased；
 4.每个柱形图上标明人数；
 5.每个柱形图的存活率按照由左往右依次递减，突出主题；

### 反馈
#### 反馈1（对index1.html）
你觉得这个可视化主要表达了什么？
女性的存活率比男性高；

这个图形中你有什么不明白的地方吗？
y轴名，标题；

如何修改会更好？
1.修改y轴名以及其它图标；
2.修改标题，更能突出主题；

#### 反馈2 （对index1.html）
你觉得这个可视化主要表达了什么？
1.女性存活率比男性高；

这个图形中你有什么不明白的地方吗？
乘客的存活率除了跟性别相关，还跟什么特征相关；

如何修改会更好？
添加至少一个相关特征的图；

#### 反馈3 （对index2.html）
你觉得这个可视化主要表达了什么？
1.女性存活率比男性高；
2.三个船舱等级中，class1的存活率最高，其次是class2，class3存活率最低；

这个图形中你有什么不明白的地方吗？
乘客的存活人数和死者人数分别是多少

如何修改会更好？
1.在柱形图标明人数；
2.添加按钮切换；

### 参考资源
Dimple.js - Add data labels to each bar of the bar chart
https://stackoverflow.com/questions/18558045/dimple-js-add-data-labels-to-each-bar-of-the-bar-chart

Dimple Stacked Bar Chart - adding label for aggregate total
https://stackoverflow.com/questions/29177336/dimple-stacked-bar-chart-adding-label-for-aggregate-total

How can I remove or replace SVG content?
https://stackoverflow.com/questions/10784018/how-can-i-remove-or-replace-svg-content
