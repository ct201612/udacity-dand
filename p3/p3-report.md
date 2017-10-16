
# 清洗OpenStreetMap数据

+ 日期：2017，6，12
+ 作者：Chris Tan
+ 邮箱：1078136147@qq.com

## 参考资料

+ 我用的是OpenStreetMap的波士顿数据，下载地址是 [mapzen](https://s3.amazonaws.com/metro-extracts.mapzen.com/boston_massachusetts.osm.bz2) ，我下载的日期是：2017，5，29。
+ 数据格式是XML,[这里](http://wiki.openstreetmap.org/wiki/OSM_XML) 是OpenStreetMap XML说明。
+ OpenStreetMap的 [Map_Features](http://wiki.openstreetmap.org/wiki/Zh-hans:Map_Features)
+ 关于正则表达式，[看完你就会正则表达式](http://mp.weixin.qq.com/s?__biz=MjM5ODg1NDI4OA==&mid=2651339860&idx=1&sn=fdc8ed53fb00868f2f411a18dba1c885&chksm=bd38f7ab8a4f7ebdbc57ddf752bec987d26fa4d3e76e10c3aaa5a6eaabbdf97841e66939b2f6&scene=4#wechat_redirect)
+ Python [`if x is not None` or `if not x is None`?](https://stackoverflow.com/questions/2710940/python-if-x-is-not-none-or-if-not-x-is-none)
+ 关于[re.match](https://stackoverflow.com/questions/1152385/alternative-to-the-match-re-match-if-match-idiom) 的判断
+ [Sum() in python](https://stackoverflow.com/questions/6007808/sum-in-python)

## 地图中遇到的问题

1. 这里有不正确的街道名和缩写的街道名。
2. 这里有不正确的电话号码格式。

### 街道名

有些街道名是不正确的如“Mill Street, Suite 104”，我修正这些街道名；还有些是缩写的街道名，如“738 Commonwealth Ave”，我补全这些缩写；在“代码和结果”这部分查看细节；

### 电话号码

有些电话号码的格式是不正确的，我修改为统一格式“+1 ### ### ####”，在“代码和结果”这部分查看细节；

## 数据概览

这里提供数据概览

### 数据文档大小

+ boston_massachusetts.osm：415MB
+ dandBoston.db:243MB
+ nodes.csv:152MB
+ nodes_tags.csv：16.8MB
+ ways.csv:20.0MB
+ ways_nodes.csv:52.5MB
+ ways_tags.csv:21.7MB

## 数据集的统计概要

+ 唯一用户的数量：1363
+ nodes的数量：1939039
+ ways的数量：310122
+ 第一次贡献的日期：2007-06-21T09:47:59

## 关于数据集其它的想法

我看用户对OpenStreetMap中波士顿贡献的占比，前三的用户是：

+ crschmidt，贡献占比53.41%，排名第一
+ jremillard-massgis，贡献占比19.08%，第二
+ OceanVortex，贡献占比4.07%，第三

建议：前三贡献占比达76.56%，因为接触数据越多，越熟悉数据，正确性越高，反之，占比少的数据，正确性不高；所以可以去掉其它占比很小的数据，然后进行分析； <br />
改进的益处：提高数据质量，降低对结果影响；<br />
预期的问题：占比少的数据包含正确数据，若去掉，会破坏数据完整性；因此需要少量多次进行“清洗—分析”这个进程，不断循环；

我也看了菜品排名，前十是：

1. pizza
2. american
3. chinese
4. italian
5. mexican
6. indian
7. thai
8. asian 
9. japanese
10. regional

我还看了nodes中可通行轮椅占比：62.01%，还有nodes中公共场所中设有轮椅道占比：0.06%；

## 总结

经过清洗，提升数据一致性和完整性；数据现在还不是100%干净，我相信这次清理符合这项目的要求。通过sql查询，我学到一些关于波士顿的知识。因为OpenStreetMap是开源的，可以提高数据有效性，准确率，完整性，非常有用；但使用此类数据前，需要进行数据清洗，提升一致性和均匀性。


## 代码和结果

### 审查街道名 


```python
import os
import xml.etree.cElementTree as ET
import re
from collections import defaultdict 
import pprint

street_type_re = re.compile(r'\b\S+\.?$',re.IGNORECASE)

expected = ["Street", "Avenue", "Boulevard", "Drive", "Court", "Place",
            "Square", "Lane", "Road", "Trail", "Parkway", "Commons"]


def audit_street_type(street_types,street_name):
    m = street_type_re.search(street_name)
    if m:
        street_type = m.group()
        if street_type not in expected:
            street_types[street_type].add(street_name)
            
def is_street_name(elem):
    return (elem.attrib['k'] == "addr:street")

def audit_street(osmfile):
    osm_file = open(osmfile,"r")
    street_types = defaultdict(set)
    
    for event, elem in ET.iterparse(osm_file, events = ("start",)):
        if elem.tag ==  "node" or elem.tag == "way":
            for tag in elem.iter("tag"):
                if is_street_name(tag):
                    audit_street_type(street_types, tag.attrib['v'])
    osm_file.close()
    return street_types
```


```python
st_types = audit_street("boston_massachusetts.osm") #跑了15分钟出结果
pprint.pprint(dict(st_types))
```

    {'104': set(['Mill Street, Suite 104']),
     ...
     'st': set(['Main st']),
     'street': set(['Boston street'])}
    

### 修正街道名

基于审查街道名的结果，我创建一个字典，包含缩写和不正确的街道名，如下；


```python
mapping = {"Ct": "Court",
            "St": "Street",
            ...
            "Charles Street South": "Charles Street"}

def update_name(name,mapping):
    m = street_type_re.search(name)
    if m:
        street_type = m.group()
        if street_type in mapping.keys():
            name = name.replace(street_type,mapping[street_type])
    
    return name 
```

### 审查电话号码


```python
def audit_phone_num(osmfile):
    osm_file = open(osmfile,'r')
    phone_num = []
    
    for event,elem in ET.iterparse(osm_file, events = ("start",)):
        
        if elem.tag == "node" or elem.tag == "way":
            for tag in elem.iter("tag"):
                if tag.attrib['k'] == "phone":
                    phone_num.append(tag.attrib['v'])
    return phone_num
```


```python
ph_num = audit_phone_num("boston_massachusetts.osm") #跑了15分钟出结果 
print ph_num
```

    ['+1 617 376 5533', '+1 617 523 7900', '+1 617 636 8100', '617-635-8532', '617-635-8710', '617-635-8930', '617-635-1534', ...'+1 617 591 6030', '+1 617 876 4344', '+1 617 522 4400', '+1 617 754 5000', '+1 617 232 9500']
    

### 修正电话号码格式 

统一电话号码格式为：“+1 ### ### ####”


```python

phone_num_re = re.compile(r'\+1\s\d{3}\s\d{3}\s\d{4}')

def update_phone_num(phone_num):
    m = phone_num_re.match(phone_num)
    
    if  m is None:
        
        if "-" in phone_num:  #+1 617 868-6330 +16176282379
            phone_num = re.sub("-", " ", phone_num)      
        ...
        elif sum(s.isdigit() for s in phone_num) < 10:
            return None
    
    
    return phone_num
        
```

### 处理OpenStreetMap XML文档

这函数是将OpenStreetMap XML转换为csv，为sqlite导入数据做准备


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import csv
import codecs
import pprint
import re
import xml.etree.cElementTree as ET

# import cerberus  #Skip verification1

# import schema #Skip verification2

OSM_PATH = "boston_massachusetts.osm"   

NODES_PATH = "nodes.csv"
NODE_TAGS_PATH = "nodes_tags.csv"
WAYS_PATH = "ways.csv"
WAY_NODES_PATH = "ways_nodes.csv"
WAY_TAGS_PATH = "ways_tags.csv"

LOWER_COLON = re.compile(r'^([a-z]|_)+:([a-z]|_)+')
PROBLEMCHARS = re.compile(r'[=\+/&<>;\'"\?%#$@\,\. \t\r\n]')

# SCHEMA = schema.schema  #Skip verification3

# Make sure the fields order in the csvs matches the column order in the sql table schema
NODE_FIELDS = ['id', 'lat', 'lon', 'user', 'uid', 'version', 'changeset', 'timestamp']
NODE_TAGS_FIELDS = ['id', 'key', 'value', 'type']
WAY_FIELDS = ['id', 'user', 'uid', 'version', 'changeset', 'timestamp']
WAY_TAGS_FIELDS = ['id', 'key', 'value', 'type']
WAY_NODES_FIELDS = ['id', 'node_id', 'position']

def shape_tag(el,tag):
    try:
        PROBLEMCHARS.search(tag.attrib['k'])
    except:
        pass
   

    tag = {'id': el.attrib['id'],
            'key' : tag.attrib['k'],
            'value':tag.attrib['v'],
            'type': 'regular'
            }
            
    if LOWER_COLON.match(tag['key']):
        tag['type'], _, tag['key'] = tag['key'].partition(':')
        


    
    return tag
        
def shape_way_node(el,i,nd):
    return  {'id' : el.attrib['id'],
                'node_id': nd.attrib['ref'],
                'position': i
                }
    

def shape_element(el, node_attr_fields=NODE_FIELDS, way_attr_fields=WAY_FIELDS
                  ):
    """Clean and shape node or way XML element to Python dict"""

    # node_attribs = {}
    # way_attribs = {}
    # way_nodes = []
    
    tags = []  #拆开写 判断
    for tag in el.iter('tag'):
        if is_street_name(tag):
            street_name = update_name(tag.attrib['v'],mapping)
            tag.attrib['v'] =street_name
        elif tag.attrib['k'] == "phone":
            phone_num = update_phone_num(tag.attrib['v'])
            tag.attrib['v'] = phone_num
        
        tags.append(shape_tag(el,tag))
   
    
    if el.tag == 'node':
        node_attribs = {f:el.attrib[f] for f in node_attr_fields} 
        return {'node': node_attribs, 'node_tags': tags}
        
    elif el.tag == 'way':
        way_attribs = {f:el.attrib[f] for f in way_attr_fields}
        way_nodes = [shape_way_node(el,i,nd)
                    for i, nd 
                    in enumerate(el.iter('nd'))]
        return {'way': way_attribs, 'way_nodes': way_nodes, 'way_tags': tags}


# ================================================== #
#               Helper Functions                     #
# ================================================== #
def get_element(osm_file, tags=('node', 'way', 'relation')):
    """Yield element if it is the right type of tag"""

    context = ET.iterparse(osm_file, events=('start', 'end'))
    _, root = next(context)
    for event, elem in context:
        if event == 'end' and elem.tag in tags:
            yield elem
            root.clear()


# def validate_element(element, validator, schema=SCHEMA):   #Skip verification4
#     """Raise ValidationError if element does not match schema"""
#     if validator.validate(element, schema) is not True:
#         field, errors = next(validator.errors.iteritems())
#         message_string = "\nElement of type '{0}' has the following errors:\n{1}"
#         error_string = pprint.pformat(errors)
        
#         raise Exception(message_string.format(field, error_string))


class UnicodeDictWriter(csv.DictWriter, object):
    """Extend csv.DictWriter to handle Unicode input"""

    def writerow(self, row):
        super(UnicodeDictWriter, self).writerow({
            k: (v.encode('utf-8') if isinstance(v, unicode) else v) for k, v in row.iteritems()
        })

    def writerows(self, rows):
        for row in rows:
            self.writerow(row)


# ================================================== #
#               Main Function                        #
# ================================================== #
def process_map(file_in): #Skip verification8 delete “validate”
    """Iteratively process each XML element and write to csv(s)"""

    with codecs.open(NODES_PATH, 'w') as nodes_file, \
         codecs.open(NODE_TAGS_PATH, 'w') as nodes_tags_file, \
         codecs.open(WAYS_PATH, 'w') as ways_file, \
         codecs.open(WAY_NODES_PATH, 'w') as way_nodes_file, \
         codecs.open(WAY_TAGS_PATH, 'w') as way_tags_file:

        nodes_writer = UnicodeDictWriter(nodes_file, NODE_FIELDS)
        node_tags_writer = UnicodeDictWriter(nodes_tags_file, NODE_TAGS_FIELDS)
        ways_writer = UnicodeDictWriter(ways_file, WAY_FIELDS)
        way_nodes_writer = UnicodeDictWriter(way_nodes_file, WAY_NODES_FIELDS)
        way_tags_writer = UnicodeDictWriter(way_tags_file, WAY_TAGS_FIELDS)

        nodes_writer.writeheader()
        node_tags_writer.writeheader()
        ways_writer.writeheader()
        way_nodes_writer.writeheader()
        way_tags_writer.writeheader()

        # validator = cerberus.Validator()  #Skip verification5

        for element in get_element(file_in, tags=('node', 'way')):
            el = shape_element(element)
            if el:
                # if validate is True:     #Skip verification6
                #     validate_element(el, validator)

                if element.tag == 'node':
                    nodes_writer.writerow(el['node'])
                    node_tags_writer.writerows(el['node_tags'])
                elif element.tag == 'way':
                    ways_writer.writerow(el['way'])
                    way_nodes_writer.writerows(el['way_nodes'])
                    way_tags_writer.writerows(el['way_tags'])


if __name__ == '__main__':
    # Note: Validation is ~ 10X slower. For the project consider using a small
    # sample of the map when validating.
    process_map(OSM_PATH)   #Skip verification7 delete  “validate=False”

```

## 数据概要

这里，使用波士顿openstreetmap地图数据集，清洗后转换为csv，导入sqlite3创建的dandBoston.db中；然后在python的sqlite3模块中，运用基本统计和sql查询语句，进行初步分析；

### 连接数据库 


```python
import sqlite3

conn = sqlite3.connect('dandBoston.db')
cur = conn.cursor()
```

### ways的数量


```python
cur.execute('SELECT COUNT(*) FROM ways') 
print cur.fetchall()
```

    [(310122,)]
    

### nodes的数量


```python
cur.execute('SELECT COUNT(*) FROM nodes')
print cur.fetchall()
```

    [(1939039,)]
    

### 用户的数量 


```python
cur.execute('SELECT COUNT(DISTINCT(e.uid)) \
            FROM (SELECT uid FROM nodes UNION ALL SELECT uid FROM ways) e')
print cur.fetchall()
```

    [(1363,)]
    

### 第一次贡献的日期 


```python
cur.execute('SELECT timestamp FROM nodes UNION ALL SELECT timestamp FROM ways\
            ORDER BY timestamp\
            LIMIT 1')
print cur.fetchall()
```

    [(u'2007-06-21T09:47:59Z\r',)]
    

### 贡献前三的用户


```python
cur.execute('SELECT e.user,COUNT(*) as num\
            FROM (SELECT user FROM nodes UNION ALL SELECT user FROM ways) e\
            GROUP BY e.user\
            ORDER BY num DESC\
            LIMIT 3')

pprint.pprint (cur.fetchall())
```

    [(u'crschmidt', 1201338),
     (u'jremillard-massgis', 429216),
     (u'OceanVortex', 91618)]
    

### 前三用户的贡献占比


```python
sum = 1939039 + 310122
c_crschmidt = 1201338.0 / sum
c_jm = 429216.0 / sum
c_ov = 91618.0 / sum

print "%.3f%%" % (c_crschmidt * 100)
print "%.3f%%" % (c_jm * 100)
print "%.3f%%" % (c_ov * 100)
```

    53.413%
    19.083%
    4.073%
    
### 引用：课程中习题的代码；
