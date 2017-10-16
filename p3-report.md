
# 清洗OpenStreetMap数据

+ 日期：2017，6，12
+ 作者：谭志成
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
     '1100': set(['First Street, Suite 1100']),
     '12': set(['Harvard St #12']),
     '1702': set(['Franklin Street, Suite 1702']),
     '3': set(['Kendall Square - 3']),
     '303': set(['First Street, Suite 303']),
     '501': set(['Bromfield Street #501']),
     '6': set(['South Station, near Track 6']),
     '846028': set(['PO Box 846028']),
     'Albany': set(['Albany']),
     'Artery': set(['Southern Artery']),
     'Ave': set(['738 Commonwealth Ave',
                 'Blue Hill Ave',
                 'Boston Ave',
                 'College Ave',
                 'Commonwealth Ave',
                 'Concord Ave',
                 'Everett Ave',
                 'Francesca Ave',
                 'Gibson Ave',
                 'Harrison Ave',
                 'Highland Ave',
                 'Josephine Ave',
                 'Lexington Ave',
                 'Massachusetts Ave',
                 'Massachusetts Ave; Mass Ave',
                 'Morrison Ave',
                 'Mystic Ave',
                 'Sagamore Ave',
                 'Somerville Ave',
                 'Washington Ave',
                 'Western Ave',
                 'Willow Ave']),
     'Ave.': set(['Brighton Ave.',
                  'Massachusetts Ave.',
                  'Somerville Ave.',
                  'Spaulding Ave.']),
     'Boylston': set(['Boylston']),
     'Broadway': set(['Broadway', 'East Broadway', 'West Broadway']),
     'Brook': set(['Furnace Brook']),
     'Building': set(['South Market Building']),
     'Cambrdige': set(['Cambrdige']),
     'Center': set(['Cambridge Center', 'Channel Center', 'Financial Center']),
     'Circle': set(['Achorn Circle',
                    'Edgewood Circle',
                    'Norcross Circle',
                    'Stein Circle']),
     'Corner': set(['Webster Street, Coolidge Corner']),
     'Ct': set(['Kelley Ct']),
     'Dartmouth': set(['Dartmouth']),
     'Dr': set(['Harborside Dr']),
     'Driveway': set(['Museum of Science Driveway']),
     'Elm': set(['Elm']),
     'Ext': set(['B Street Ext']),
     'Fellsway': set(['Fellsway']),
     'Fenway': set(['Fenway']),
     'Floor': set(['Boylston Street, 5th Floor']),
     'Garage': set(['Stillings Street Garage']),
     'Greenway': set(['East Boston Greenway']),
     'H': set(['H']),
     'HIghway': set(['American Legion HIghway']),
     'Hall': set(['Faneuil Hall']),
     'Hampshire': set(['Hampshire']),
     'Highway': set(['American Legion Highway',
                     'Cummins Highway',
                     "Monsignor O'Brien Highway",
                     'Providence Highway',
                     'Santilli Highway']),
     'Holland': set(['Holland']),
     'Hwy': set(["Monsignor O'Brien Hwy"]),
     'Jamaicaway': set(['Jamaicaway']),
     'LEVEL': set(['LOMASNEY WAY, ROOF LEVEL']),
     'Lafayette': set(['Avenue De Lafayette']),
     'Longwood': set(['Longwood']),
     'Mall': set(['Cummington Mall']),
     'Market': set(['Faneuil Hall Market']),
     'Newbury': set(['Newbury']),
     'Park': set(['Acorn Park',
                  'Agassiz Park',
                  'Austin Park',
                  'Batterymarch Park',
                  'Canal Park',
                  'Exeter Park',
                  'Giles Park',
                  'Granton Park',
                  'Greenough Park',
                  'Malden Street Park',
                  'Monument Park',
                  'Newsome Park']),
     'Pasteur': set(['Avenue Louis Pasteur']),
     'Pkwy': set(['Birmingham Pkwy']),
     'Pl': set(['Longfellow Pl']),
     'Plaza': set(['Park Plaza', 'Two Center Plaza']),
     'Rd': set(['Abby Rd',
                'Aberdeen Rd',
                'Bristol Rd',
                'Goodnough Rd',
                'Oakland Rd',
                'Rawson Rd',
                'Soldiers Field Rd',
                'Squanto Rd']),
     'Row': set(['Assembly Row', 'East India Row']),
     'ST': set(['Newton ST']),
     'South': set(['Charles Street South']),
     'Sq.': set(['1 Kendall Sq.']),
     'St': set(['Adams St',
                'Antwerp St',
                'Arsenal St',
                'Athol St',
                'Bagnal St',
                'Bowdoin St',
                'Brentwood St',
                'Broad St',
                'Cambridge St',
                'Centre St',
                'Charles St',
                'Congress St',
                'Court St',
                'Cummington St',
                'Dane St',
                'Duval St',
                'Elm St',
                'Everett St',
                'George St',
                'Grove St',
                'Hammond St',
                'Hampshire St',
                'Holton St',
                'Kirkland St',
                'Leighton St',
                'Litchfield St',
                'Lothrop St',
                'Mackin St',
                'Main St',
                'Maverick St',
                'Medford St',
                'Merrill St',
                'Mt Auburn St',
                'N Beacon St',
                'Norfolk St',
                'Portsmouth St',
                'Richardson St',
                'Sea St',
                'South Waverly St',
                'Stewart St',
                'Ware St',
                'Waverly St',
                'Winter St',
                'Winthrop St']),
     'St,': set(['Walnut St,']),
     'St.': set(['Albion St.',
                 'Banks St.',
                 'Boylston St.',
                 'Centre St.',
                 'Elm St.',
                 'Main St.',
                 'Marshall St.',
                 'Maverick St.',
                 'Pearl St.',
                 'Prospect St.',
                 "Saint Mary's St.",
                 'Stuart St.',
                 'Tremont St.']),
     'Street.': set(['Hancock Street.']),
     'Terrace': set(['Alberta Terrace',
                     'Alveston Terrace',
                     'Arborway Terrace',
                     'Beecher Terrace',
                     'Norfolk Terrace',
                     'Westbourne Terrace']),
     'Turnpike': set(['Boston Providence Turnpike']),
     'Way': set(['Artisan Way',
                 'Ballard Way',
                 'Cedar Lane Way',
                 'Courthouse Way',
                 'David G Mugar Way',
                 'Davidson Way',
                 'Evans Way',
                 'Harry Agganis Way',
                 'Legends Way',
                 'Memorial Way',
                 'Riley Way',
                 'Ross Way',
                 'Transportation Way',
                 'Yawkey Way']),
     'Wharf': set(['Central Wharf', 'Lewis Wharf', 'Long Wharf', 'Rowes Wharf']),
     'Windsor': set(['Windsor']),
     'Winsor': set(['Winsor']),
     'Yard': set(['Charleston Navy Yard']),
     'floor': set(['First Street, 18th floor']),
     'place': set(['argus place']),
     'rd.': set(['Corey rd.']),
     'st': set(['Main st']),
     'street': set(['Boston street'])}
    

### 修正街道名

基于审查街道名的结果，我创建一个字典，包含缩写和不正确的街道名，如下；


```python
mapping = {"Ct": "Court",
            "St": "Street",
            "st": "Street",
            "St.": "Street",
            "St,": "Street",
            "ST": "Street",
            "street": "Street",
            "Street.": "Street",
            "Ave": "Avenue",
            "Ave.": "Avenue",
            "ave": "Avenue",
            "Rd.": "Road",   
            "rd.": "Road",
            "Rd": "Road",    
            "Hwy": "Highway",
            "HIghway": "Highway",
            "Pkwy": "Parkway",
            "Pl": "Place",      
            "place": "Place",
            "Sedgwick": "Sedgwick Street",
            "Sq.": "Square",
            "Albany":"Albany Street",
            "Newbury": "Newbury Street",
            "Boylston": "Boylston Street",
            "Brook": "Brook Parkway",
            "South Market Building":"South Market Street",
            "Dartmouth":"Dartmouth Street",
            "Harborside Dr":"Harborside Driveway",
            "Elm": "Elm Street",
            "B Street Ext":"B Street",
            "Fellsway":"Fellsway Street",
            "Stillings Street Garage":"Stillings Street",
            "Webster Street, Coolidge Corner": "Webster Street",
            "Faneuil Hall": "Faneuil Hall Market Street",
            "Furnace Brook": "Furnace Brook Parkway",
            "Federal": "Federal Street",
            "South Station, near Track 6": "South Station, Summer Street",
            "Mill Street, Suite 104":"Mill Street",
            "First Street, Suite 303": "First Street",
            "Bromfield Street #501":"Bromfield Street",
            "Kendall Square - 3": "Kendall Square",
            "Franklin Street, Suite 1702": "Franklin Street",
            "First Street, Suite 1100": "First Street",
            "Harvard St #12":"Harvard Street",
            "Windsor": "Windsor Stearns Hill Road",
            "Winsor": "Winsor Village Pilgrim Road",
            "First Street, 18th floor": "First Street",
            "Sidney Street, 2nd floor": "Sidney Street",
            "Boston Providence Turnpike": "Boston Providence Highway",
            "Holland": "Holland Albany Street",
            "Hampshire": "Hampshire Street",
            "Boylston Street, 5th Floor": "Boylston Street",
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
ph_num = audit_phone_num("boston_massachusetts.osm") #跑了15分钟出结果 含"-",".","(",")",连在一起（10，11，+11），缺“+”，含字母，非数字
print ph_num
```

    ['+1 617 376 5533', '+1 617 523 7900', '+1 617 636 8100', '617-635-8532', '617-635-8710', '617-635-8930', '617-635-1534', '617-635-8507', '617-984-8762', '617-552-7800', '(617) 464-2485', '781-326-5350', '617-361-0050', '617-635-1660', '617-442-5419', '617-635-8264', '617-474-7950', '617-254-8383', '617-349-6555', '617-266-8427', '781-393-2333', '617-635-6824', '617-626-8857', '617-846-5509', '617-635-8545', '617-635-6456', '617-635-8940', '617-284-7800', '617-265-1172', '617-635-8608', '617-635-8497', '617-559-6700', '617-635-8601', '617-731-3196', '617-522-1841', '617-635-9976', '781-848-6250', '617-635-9730', '617-354-0047', '617-666-3311', '617-926-7760', '617-696-4568', '617-635-9832', '617-268-2326', '617-442-1993', '617-635-8466', '617-267-4530', '617-635-8979', '781-286-8270', '617-635-6429', '617-469-0300', '617-635-6789', '617-635-9989', '781-326-5350', '617-635-8234', '617-696-4285', '617-427-3881', '617-635-1615', '617-394-2480', '617-825-6262', '617-635-9914', '617-635-9860', '781-646-9307', '617-635-8109', '617-394-2410', '617-635-8069', '617-442-2660', '617-898-1798', '617-333-9610', '617-349-6632', '617-635-8773', '781-284-4970', '781-286-8278', '617-333-6688', '781-393-5600', '617-635-8275', '617-696-4282', '781-461-5978', '617-389-2448', '617-326-0425', '617-389-7277', '781-326-5351', '617-635-8372', '617-635-8412', '617-394-5013', '617-696-4470', '617-635-7734', '617-264-5349', '781-393-2177', '617-635-8722', '617-522-6980', '617-625-6365', '617-635-8388', '617-547-0101', '781-641-5987', '617-924-5219', '617-635-8865', '617-635-1650', '617-282-2577', '617-625-6600', '617-254-3110', '617-625-6600', '617-635-8529', '617-635-8149', '617-349-6577', '617-635-8060', '617-566-2361', '617-696-4288', '617-889-3100', '617-635-9957', '781-286-8296', '781-316-3730', '617-542-2325', '617-889-7540', '617-635-6437', '617-635-8631', '617-846-5543', '617-567-7456', '617-394-5040', '617-522-5544', '617-731-5330', '617-730-2580', '617-635-7945', '617-427-1177', '617-232-0300', '617-635-8910', '617-282-3388', '617-635-9870', '617-846-5507', '617-296-1161', '617-696-3241', '617-625-3459', '617-635-8157', '617-635-8523', '617-635-8840', '617-296-1210', '617-265-7110', '617-889-8422', '617-232-8390', '781-286-8298', '617-635-8935', '617-742-0520', '617-635-8379', '781-646-7770', '+1 617 855 2847', '617-635-6446', '617-635-8725', '617-635-8534', '617-523-7577', '617-739-1794', '617-422-0042', '617-975-0080', '617-354-5410', '617-825-3883', '617-924-0187', '781-393-2343', '781-326-3700', '617-635-8781', '617-524-6136', '617-635-8827', '617-635-6437', '617-499-1451', '617-635-8731', '617-361-3563', '617-227-3143', '617-349-6588', '617-635-9911', '617-323-1050', '617-879-4250', '617-635-8618', '617-536-5984', '617-445-1515', '617-566-4394', '617-635-8803', '617-625-6600', '781-284-0519', '617-282-8573', '617-497-8454', '617-825-0703', '617-268-8431', '781-286-8222', '617-713-5124', '617-876-6068', '617-635-6470', '617-625-6600', '617-635-8187', '617-635-8393', '+1 617 6359373', '617-635-8672', '617-984-8791', '617-349-6841', '781-326-6900', '617-825-7955', '617.253.1000', '+1 617 787 1080', '+1 617 426.2000', '+1 800 662 9937', '(617) 868-8658', '617-536-5260', '+1 617 636 5000', '+1 617 296 0061', '(617) 984-8762', '983-890-4140', '617-527-0045', '781-316-3593', '617-635-9865', '617-522-0880', '781-485-2715', '+1 617 325 4439', '+1 617 876-6555', '+1-617-253-4680', '(617) 349-4015', '617-278-4000', '+1-617-496-1027', '1-617-253-5927', '+1 (617) 868-3392', '+1 (617) 264-4400', '+16175779100', '617 491 2999', '+1 617 7289101', '617-354-7766', '+1 617 6664200', '1-617-497-0965', '617-491-2220', '+1 617 354 8881', '+16174922541', '617-491-6616', '6175760280', '6179455001', '617-679-7000', '+1 (617) 482-3473', '+ 1 617 242 9000', '617-773-0120', '+1 617 569 5800', '+1-617-437-8884', '6177428565', '6173671866', '6177205740', '+1 617 494-1994', '781-337-8770', '781-340-7866', '617-472-9970', '781-340-7888', '781-335-1836', '781-340-4922', '781-340-1455', '781-340-0740', '617-776-3596', '781-337-4869', '781-803-6675', '781-789-8063', '+1 617 354 5471', '+1 617 625-1793', '+1 617 868-7711', '617-666-3900', '617-876-2200', '617-500-9373', '+1-617-627-3030', '617-698-8139', '617-666-3900', '+1 617 439 7000', '+1 617 783-5804', '617 262 6505', '617-742-2100', '+1 617 4412116', '(617) 714-3974', '+1 617 4971513', '1 617 252 4444', '1-617-367-0055', '+1 617 227-2750', '+1 617 623 0867', '+1 617 628-3618', '617-625-0600', '+1 617 354-4110', '+16178761655', '+1 617 2682435', '+16175766260', '6174247000', '+1 617 532 4600', '+1 617 443 0800', '+1 617 720 1050', '+1 617 443 0800', '+1 617 262 2424', '+1 617 437 0300', '+1 617 425 4444', '+1 617 357 LUCK', '+1 617 532 4600', '+1 781 3226057', '+1 781 3389698', '+1 781 3880290', '+1 781 3221400', '+1 781 3978885', '+1 781 3243324', '+1 781 6053120', '+1 781 3244747', '+1 781 3880240', '+1 781 3244466', '+1 781 3220303', '+1 781 3217434', '+1 781 3223400', '+1 781 3240600', '+1 781 3880078', '+1 781 3210859', '+1 781 3212737', '+1 781 3243344', '+1 781 3242900', '+1 781 3880099', '+1 617 282-5585', '+1-617-492-7021', '617-294-6063', '+1 617 876 3076', '+1 617 253 4481', '+1 617 864 3400', '617.354.4200', '617-576-3000', '617-269-0110', '(617) 436-2786', '(617) 825-6180', '+1 781 3951600', '+1 781 3969845', '+1 781 3220400', '6173752524', '+1 617-734-7708', '617-566-7730', '(617) 349-4023', '+1 617 541 6600', '617-261-0158', '617 330-1002', '+1 617 2690900', '(617) 524-2836', '+1 617 524-1110', '+1 617 522-2255', '+1 617 666 0000', '(617) 734-7700', '+1 617 666-6072', '+1-617-868-4200', '617-625-0700', '+1-617-623-4378', '617-442-1162', '617-427-8068', '617-424-5500', '617-536-4540', '617-247-3399', '617-236-8324', '617-267-1490', '617-266-7171', '617-547-2727', '1-857-233-5943', '+1 617.933.5000', '617-372-8156', '+1 617 441-2500', '+1 781 777-1451', '6179121234', '1-617-494-5300', '+1 617 742 3050', '(617) 247-4141', '+1 781-646-0110', '617 625 0001', '617-225-2525', '+1-617-936-2285', '616-225-0409', '+1 617 424-6400', '+16172772880', '+1 (781) 267-4539', '+1 (617) 284-6212', '+1 ( 617 ) 764 - 5693', '+1 (617) 629-0264', '+1 (617) 284-6228', '+1 (617) 623-9068', '+1 617 375 8550', '+1-617-542-5942', '+1 617 561 0011', '+1 617 567-4627', '+1 617 561-3737', '+1 617 349-1650', '+1 617 567-8787', '+1 617 561-1112', '1-617-209-2257', '+18572596888', '6172546030', '6179457030', '+1 617 9263300', '617 923 6060', '(617) 924-9600', '+1 (617) 497-5555', '+1 617 661 0077', '781-646-5598', '+16174076271', '+16172472818', '617-500-6778', '617-500-6778', '+1 617 421 8692', '(617) 963-0889', '(617) 855-8593', '(617) 494-8700', '617.954.2098', '+1 617 229 7900', '+1 617 868-6330', '+16178688800', '+16173548808', '+16178689400', '617-522-4850', '617-576-6163', '+1-617-764-3152', '+1-617-666-4444', '(781) 395-8885', '781.396.8999', '781-391-9969', '(781) 395-9097', '781-783-6996', '+1 617 612 8253', '617-549-8255', '+1 617 576 1253', '+1 617 567-9871', '+1 (617) 772-5800', '+1 (617) 574-7100', '+1 617 524-7663', '+16175697272', '+1 781 9171298', '+1 781 9171299', '+1 781 9171300', '+1 617 227-8600', '+1 617-723-7500', '+1  617 422 0045', '617-354-5287', '+1 857 2596552', '+1 617 4927540', '+1-617-654-5302', '+1 617 623 0803', '+1 617 3309790', '+1 617 3384600', '1-617-742-2286', '+1 6175361775', '617-576-1900', '+1 617 7386575', '+1 617 7348100', '+90 617 227-3193', '8776370450', '+1 617-208-6922;+1 617-208-6928', '+1 617 627-9801', '+1 617 864-1300', '+1 617 661-0969', '+1 617 496-5955', '+1 617 524-2453', '+1 617 542-8623', '+1 617 325-2453', '+1 617 232-0446', '+1 617 440 4192', '(866) 995-2479', '+1 866 228 6439', '+1 617 345 5495', '(617) 863-3650', 'AstraZeneca Neuroscience: T: (617) 679-1680', '617-494-9330,  Forest City Management', '617-225-2777', '781-646-6835', '(617) 492-7289', '617.588.2628', '(617) 871-9911', 'Phone 617.714.0555', '617-418-2200', '617-575-2824', '617.498.0020', '617-250-8433', '617-492-7083', '1-855-305-2347', '617.431.7250', '617-871-2098', '(617) 401-3390', '(617) 491-3400', '617-284-6878', '+1 617 254-0112', '+1 617 541-1270', '+1 617 307-6367', '+1 617 628-0328', '+1 617 497-1136', '+1 617 491-3405', '+1 617 776-1234', '+1 617 492-5588', '(857) 350-4035', '+16179451548', '+1-617-519-9296', '+1-857-999-2003', '+1-617-431-1433', '+1-617-764-4143', '+1-857-242-3605', '+1-617-945-1768', '+16174920711', '+1 617 666 3830', '+1-617-254-9600', '+1-617-783-5100', '+1-617-539-8570', '+1 617 623 1159', '+1 617 764 4960', '+1 617 616 5561', '+1-617-661-8049', '+1 617 337 4088', '+1 617 996 6100', '+1 617 666 1790', '+1 617 666 8282', '+1 857 263 8491', '(617) 868-9560', '6173389788', '1-617-379-4017', '617-426-2121', '(617) 535-7763', '617-993-0750', '+1 617 868 6740', '+1 617 354 9898', '+1 617 354 8912', '+1 617 354 5279', '+1 617 868 4140', '+1 617 547 6000', '+1 617 354 3388', '+1 617 945 0878', '+1 617 876 5550', '+1 617 349 3937', '+1 617 576 0108', '+1 617 868 8838', '+1 617 945 9656', '+1 617 492 0870', '+1 617 576 5300', '+617 242 2229', '6175369000', '+1-617-783-9800', '+1-617-782-7625', '+1-617-202-5817', '617-482-7467', '+16175470836', '(617) 864-3191', '+1-617-294-4233', '+1 (857) 417-2396', '+1 617 6815000', '(857) 706-1125', '+1-617-887-9560', '+1 (617) 887-0905', '+1 617 635-8198', '+1 617 635-6384', '+1 617 524-2453', '+1 617 522 6241', '+1 617 524-5120', '+1 617 524-0130', '+1 617 522-7272', '+1 617 983-0010', '+1 617 524-9200', '+1 617 524-4714', '+1 617 522-1610', '+1 617 524-9852', '+1 617 524-9677', '+1 617 522-3404', '+1 617 524-9893', '+1 617 524-4572', '+1 617 524-1155', '+1 617 522-7997', '+1 617 477-4519', '+1 617 522-7414', '+1 617 522-8242', '+1 617 522-2225', '+1 617 522-3900', '+1 617 942-2966', '+1 617 522-7264', '+1 617 983-4294', '+1 617 524-7300', '+1 617 522-5061', '+1 617 524-1570', '+1 617 983-0154', '+1 617 522-4600', '+1 617 522-9263', '+1 617 522-4154', '+1 617 522-0277', '+1 617 522-2299', '+1 617 522-3443', '+1 617 553-4884', '+1 617 522-6497', '+1 617 477-9442', '+1 857 203-9414', '+1 617 522-7783', '+1 617 971-0733', '+1 857 312-6672', '+1 617 522-0184', '+1 617 524-4041', '+1 617 942-0294', '617 602 1921', '+1 617 983-5466', '+1 617 983-9854', '+1 617 524-9461', '+1 617 522-4744', '+1 617 522-4948', '+1 617 522-0226', '+1 617 942-8181', '+1 617 262-8222', '+1 617 269-6288', '+1 617 269-1309', '+1 617 524-5224', '+1 617 524-8833', '+1 617 522-6700', '+1 617 553-5480', '+1 617 522-5511', '+1 617 383-6522', '+1 617 442-4151', '+1 617 636 3737', '617442299', '+1 - 617-466-207', '+1-617-466-2290', '+1-617-889-9800', '+1-617-819-8175', '+1 617 522-2195', '+1 617 524-0262', '+1 617 428-0155', '+1 617 357-3000', '+1 617 523-0990', '+1 617 738-4706', '+1 617 327-6688', '+16177872430', '(617) 307-7608', '617-876-1262', '+16172628900', '+16175366300', '+16178679300', '+16177786841', '+16174211200', '+26777722147', '+1 617 764-5365', '+1 617 670-0637', '+1 617 670-0637', '+1 617 776-2011', '+1 617 876-3988', '+16178646100', '+16174911160', '857-250-2484', '(617) 451-2395', '617-422-5898', '+1 617 2770324', '(617) 577-7427', '+16179451008', '+1617958DELI', '6178460955', '+16176617411', '+16172997405', '617-227-0150', '+16173494758', '+16178764853', '+1 617 236 0990', '+16176282379', '6176661079', '18006350489', '6173811647', '+16179452708', '857 317 5220', '+1 617-471-8530', '617-472-3232', '617-723-9300', '1-617-227-3962', '617-227-0236', '617-227-6005', '6172414999', '1-617-437-7700', '+16173543600', '+16175722000', '1-617-421-9595', '+1-617-491-1200', '+1-617-424-1010', '+1-617-439-9020', '+16174412101', '+16174971221', '+1 617-547-1595', '+16176613040', '617-889-5800', '+16177588495', '+1-617-787-0767', '617-738-5289', '+1-617-522-0045', '617-277-3339', '617-242-2080', '+1-617-734-3016', '7812849693', '617 236 0571', '+1-617-387-7101', '(617) 277-3737', '781-682-4202', '617-451-2289', '6172270786', '+1617-268-4686', '(617) 354-1390', '(617) 547-3110', '888-476-2821', '617-782-5782', '+1-617-268-1119', '617-783-8800', '6174235501', '7812194260', '6174220082', '6177658636', '8572637169', '6174233447', '6173576861', '6172478100', '+1 617 489 1371', '781-648-0279', '+1 617 720 0052', '1-617-426-1812', '1-617-624-1234', '+1 (617) 714-5152', '+1 617 946 4653', '+1 857 337 5110', '1-617-437-9999', '1 877 632-6654', '1 617 267 0060', '1 617 247 8656', '1-617-358-2679', '1-617-247-2789', '1-617-731-4556', '1-617-358-2881', '1-617-236-8371', '1-617-227-4059', '1-617-262-0911', '1-617-742-2090', '1-617-695-2529', '+1 888 815 5510', '+1 617 779 1500', '+1 857 239-9276', '+1 617 742-9991', '1-617-437-0300', '718-322-3708', '+16177235612', '+16177232786', '+16172674774', '+1-617-789-4100', '6172088663', '+1 617 236 0990', '781-391-5606', '781-395-2221', '+1 617 945 0733', '(617) 264-4800', '(617) 462-4783', '(617) 277-4992', '+1-617-731-8310', '+1-617-731-0018', '6177445290', '+16172481520', '+16177424724', '+16177428770', '+16177423115', '+1-774-218-4054', '+1-617-770-2828', '+1-617-786-8805', '+1-617-845-5990', '+1-617-773-4458', '+1-617-479-7100', '6174821800', '+1 617 492 1234', '617-238-5110', '+1-617-491-9638', '+1-617-868-0772', '+1 781 3910404', '+1.781.648.0360', '617-251-9646 or 617-290-6040', '617-268-0750', '617-884-2626', '617-635-8358', '+1-617-242-7275', '617-635-8789', '617-561-1371', '617-625-6600', '+1 617 436 8301', '+1-617-889-3100', '+1-617-884-4188', '(617) 373-2350', '617-364-6767', '+1-617-782-7870', '+1 617-332-9255', '+1 617 524 6822', '1-617-469-4440', '+1 617 547 1234', '+1 617 547 7400', '+1 617 492 9030', '+1 617 524-4881', '781-340-9500', '(617)327-5000', '617-542-5682', '+1 617 722 3000', '+1 617 265-9463', '+1-617-884-0070', '(617) 423-4630', '617 523 0453', '(617) 577-8856', '617-266-5999', '6172244000', '+16179950900', '+16173545065', '+1 617 776-2100', '+1 617 252 0005', '1 617 4842228', '+1 617 876 6990', '+1 617 666-2770', '+1 781 3932869', '+1 866 7740879', '617-625-6600', '617 776 4036', '+1 617 242 9728', '617-635-9837', '+1 617 349 0700', '617-262-4567', '+1-617-566-1401', '(617) 288-1202', '617-635-8383', '+1 714 488 2040', '617-635-8436', '617-635-8072', '+1 617 524-0740', '617-625-6600', '(617) 349-4021', '+1 617 354 8582', '781-335-0191', '781-380-0240', '781-335-2210', '617-799-1455', '+1 617 825 9660', '+1 617 354 9523', '617-786-7900', '(617) 696-5274', '+1-617-547-3144', '617-623-3447', '+1 617 523 9327', '617-984-8710', '+1 617 484 3078', '617-993-5800', '781-337-3942', '617-635-8454', '+1-617-394-2120', '+1 617 928 1400', '+16174898767', '+1 617 524-3313', '+1-617-627-2000', '+16176255700', '+1 617 354 7644', '+16174977626', '617-323-2500', '+1 617 524-0399', '617-323-8202', '+1 617 5243158', '617-635-8102', '+1 617 7340608', '+16178763601', '+1 617 547 2455', '+1 617 623-5000', '617-635-8757', '617 868-3000', '+1 617 547 3272', '617-536-3355', '617 536-1950', '+1 617 484 3519', '617-984-8745', '+1 617 277 5888', '+1 617 492 0070', '+1 617 983-9334', '1-781-643-1377', '+1 617 354 3062', '+18006302521', '+16174927777', '+1 617 381 4469', '+16176610433', '+1 617 491 0843', '617-993-5600', '617-364-2300', '617-484-2400', '+1 617-328-9600', '+1 617466 4855', '617-635-8510', '617-484-4410', '617-993-5500', '617-389-0240', '617-984-8729', '+1 617 696 4600', '617-387-7443', '617-698-0909', '(617) 472-3489', '+1 617 924 6143', '617-984-8715', '781-643-9031', '617-984-8708', '617-879-4650', '617-472-8466', '617-635-8137', '617-928-4500', '+1-617-248-8971', '617-332-1320', '+16178646885', '617-559-9200', '+1 617 441 0537', '617-566-0240', '617-630-1971', '617-964-7765', '617-559-9330', '+16174924922', '+1 617 524 1634', '+1 617 575 8700', '617-489-6600', '617.884.0134', '+1-781-853-0005', '+1 617 522-7282', '(617) 621-9902', '+1 617 524-3992', '617-254-7163', '6174894408', '+1 617 576 3229', '+16175766779', '+1 617 876 4047', '6179243068', '781-331-SUBS', '+1 617 524-1760', '+1 617 547 0120', '+1 617 623-5000', '+1 617 491 6969', '+1 617 547 7788', '617-879-4600', '+1 617 497 3926', '617-547-1950', '+1 617 547-0680', '617-696-4291', '+1 617 536 8851', '617-370-6227', '+1 617 385-4000', '617-635-8641', '+1 781 284 9491', '+1 617-466-4600', '+1 617 466 4662', '+1 857 209 2747', '+16173494758', '+1 781 648 3799', '781-316-3782', '781-641-5992', '+16173541441', '617-244-4246', '781-335-2166', '781-646-0510', '+16173496575', '(781) 829-3100', '+1-617-884-0646', '617-389-4500', '6177471000', '617-972-7200', '617-969-2260', '+1 617 726 2000', '+1-617-478-3100', '617 973 0421', '+1 781 3954899', '781-316-3593', '617-984-8713', '617 984 8754', '781-316-3702', '+1 (617) 884-9579', '617-474-3000', '617-474-3000', '617-522-2261', '+1 617 635 8937', '781-337-8221', '(617) 328-7772', '+1 617 667 7000', '+1 617 667 7000', '+1 617 309 2400', '+1 866 408 3324', '617-635-8895', '+1 617 355 6000', '617-735-9500', '617-635-8450', '1-617-523-2338', '617-742-4715', '+1 (617) 536-9788', '617-327-2160', '(617) 987-0086', '+16174416999', '+16175763634', '+16175473721', '617-868-1260', '617-547-6100', '+1 611 542 7696', '617-338-3030', '617-265-4084', '617-846-5506', '617-738-2700', '617-879-4500', '617-635-8169', '617-635-8665', '617-635-8099', '617-635-8652', '617-635-8792', '617-635-8064', '617-635-6804', '617-328-3444', '617-696-3548', '1-617-323-2181', '+1 617 2324452', '+1 617 2324452', '617-325-7977', '+1 617 665 1000', '+1 617 591 4500', '617-926-7740', '617-635-8810', '617-984-6600', '617-349-6600', '617-698-2464', '617-325-9338', '617-635-6804', '617-635-8778', '617-567-6609', '617-635-8131', '617-635-8681', '617-879-4400', '617-984-8763', '617-635-8405', '617-266-7480', '+1 781 316 3768', '(617) 576-2220', '617-679-5500', '617-635-8082', '617-277-2456', '781-740-2038', '617-635-8247', '617-254-7163', '617-635-7680', '+1 617 3942400', '+1 781 3963636', '617-635-9896', '+1 617 254 3800', '+1 617 9525000', '+1 617 3496314', '617-332-7770', '617-635-9873', '781-843-3636', '+1 617 522-2635', '617-787-5507', '617-635-8399', '+1 617 207-5999', '+1 617 912 0400', '617-536-3355', '617-536-3355', '6177205544', '617-635-8239', '+1 617 523 0100', '+1 617 752 4206', '617-713-5003', '617-635-8205', '617-635-8422', '617-635-8212', '617-635-8253', '617-635-8676', '+1 617 876 7772', '+16173496567', '(617) 206-2994', '+1 (781) 464-2000', '+1 617 267 6730', '+1 617 207 3009', '+1 617 884 5660', '+1 617 389 6270', '+1 781 286 3100', '+1 800 333 0338', '+1 617 323 7700', '+1 617 469 0300', '+1 877 822 4722', '+1 617 522 8110', '+1 617 635 1645', '+1 617 254 3800', '+1 617 789 3000', '+1 617 254 1100', '+1 617 731 3200', '+1 617 591 6030', '+1 617 876 4344', '+1 617 522 4400', '+1 617 754 5000', '+1 617 232 9500']
    

### 修正电话号码格式 

统一电话号码格式为：“+1 ### ### ####”


```python

phone_num_re = re.compile(r'\+1\s\d{3}\s\d{3}\s\d{4}')

def update_phone_num(phone_num):
    m = phone_num_re.match(phone_num)
    
    if  m is None:
        
        if "-" in phone_num:  #+1 617 868-6330 +16176282379
            phone_num = re.sub("-", " ", phone_num)      
        if "." in phone_num:
            phone_num = re.sub(".", "", phone_num)
        if "(" in phone_num or ")" in phone_num:
            phone_num = re.sub("[()]","", phone_num)
        
        if re.match(r'\d{10}',phone_num) is not None:  
            phone_num = phone_num[:3] + " " + phone_num[3:6] + " " + phone_num[6:]
        if re.match(r'\d{11}',phone_num) is not None:
            phone_num = phone_num[:1] + " " + phone_num[1:4] + " " + phone_num[4:7] + " " + phone_num[7:]
        if re.match(r'\+\d{11}',phone_num) is not None:
            phone_num = phone_num[:2] + " " + phone_num[2:5] + " " + phone_num[5:8] + " " + phone_num[8:]
            
        if re.match(r'\d{3}\s\d{3}\s\d{4}',phone_num) is not None:
            phone_num = "+1 " + phone_num
        if re.match(r'1\s\d{3}\s\d{3}\s\d{4}',phone_num) is not None:
            phone_num = "+" + phone_num
            
        if re.match(r'\+1\s\d{3}\s\d{7}',phone_num) is not None:
            phone_num = phone_num[:8] + phone_num[8:10] + " " + phone_num[10:]
        
        elif sum(s.isdigit() for s in phone_num) < 10:
            return None
    
    
    return phone_num
        
```

### 处理OpenStreetMap XML文档

这函数是将OpenStreetMap XML转换为csv，为sqlite导入数据做准备


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
After auditing is complete the next step is to prepare the data to be inserted into a SQL database.
To do so you will parse the elements in the OSM XML file, transforming them from document format to
tabular format, thus making it possible to write to .csv files.  These csv files can then easily be
imported to a SQL database as tables.

The process for this transformation is as follows:
- Use iterparse to iteratively step through each top level element in the XML
- Shape each element into several data structures using a custom function
- Utilize a schema and validation library to ensure the transformed data is in the correct format
- Write each data structure to the appropriate .csv files

We've already provided the code needed to load the data, perform iterative parsing and write the
output to csv files. Your task is to complete the shape_element function that will transform each
element into the correct format. To make this process easier we've already defined a schema (see
the schema.py file in the last code tab) for the .csv files and the eventual tables. Using the 
cerberus library we can validate the output against this schema to ensure it is correct.

## Shape Element Function
The function should take as input an iterparse Element object and return a dictionary.

### If the element top level tag is "node":
The dictionary returned should have the format {"node": .., "node_tags": ...}

The "node" field should hold a dictionary of the following top level node attributes:
- id
- user
- uid
- version
- lat
- lon
- timestamp
- changeset
All other attributes can be ignored

The "node_tags" field should hold a list of dictionaries, one per secondary tag. Secondary tags are
child tags of node which have the tag name/type: "tag". Each dictionary should have the following
fields from the secondary tag attributes:
- id: the top level node id attribute value
- key: the full tag "k" attribute value if no colon is present or the characters after the colon if one is.
- value: the tag "v" attribute value
- type: either the characters before the colon in the tag "k" value or "regular" if a colon
        is not present.

Additionally,

- if the tag "k" value contains problematic characters, the tag should be ignored
- if the tag "k" value contains a ":" the characters before the ":" should be set as the tag type
  and characters after the ":" should be set as the tag key
- if there are additional ":" in the "k" value they and they should be ignored and kept as part of
  the tag key. For example:

  <tag k="addr:street:name" v="Lincoln"/>
  should be turned into
  {'id': 12345, 'key': 'street:name', 'value': 'Lincoln', 'type': 'addr'}

- If a node has no secondary tags then the "node_tags" field should just contain an empty list.

The final return value for a "node" element should look something like:

{'node': {'id': 757860928,
          'user': 'uboot',
          'uid': 26299,
       'version': '2',
          'lat': 41.9747374,
          'lon': -87.6920102,
          'timestamp': '2010-07-22T16:16:51Z',
      'changeset': 5288876},
 'node_tags': [{'id': 757860928,
                'key': 'amenity',
                'value': 'fast_food',
                'type': 'regular'},
               {'id': 757860928,
                'key': 'cuisine',
                'value': 'sausage',
                'type': 'regular'},
               {'id': 757860928,
                'key': 'name',
                'value': "Shelly's Tasty Freeze",
                'type': 'regular'}]}

### If the element top level tag is "way":
The dictionary should have the format {"way": ..., "way_tags": ..., "way_nodes": ...}

The "way" field should hold a dictionary of the following top level way attributes:
- id
-  user
- uid
- version
- timestamp
- changeset

All other attributes can be ignored

The "way_tags" field should again hold a list of dictionaries, following the exact same rules as
for "node_tags".

Additionally, the dictionary should have a field "way_nodes". "way_nodes" should hold a list of
dictionaries, one for each nd child tag.  Each dictionary should have the fields:
- id: the top level element (way) id
- node_id: the ref attribute value of the nd tag
- position: the index starting at 0 of the nd tag i.e. what order the nd tag appears within
            the way element

The final return value for a "way" element should look something like:

{'way': {'id': 209809850,
         'user': 'chicago-buildings',
         'uid': 674454,
         'version': '1',
         'timestamp': '2013-03-13T15:58:04Z',
         'changeset': 15353317},
 'way_nodes': [{'id': 209809850, 'node_id': 2199822281, 'position': 0},
               {'id': 209809850, 'node_id': 2199822390, 'position': 1},
               {'id': 209809850, 'node_id': 2199822392, 'position': 2},
               {'id': 209809850, 'node_id': 2199822369, 'position': 3},
               {'id': 209809850, 'node_id': 2199822370, 'position': 4},
               {'id': 209809850, 'node_id': 2199822284, 'position': 5},
               {'id': 209809850, 'node_id': 2199822281, 'position': 6}],
 'way_tags': [{'id': 209809850,
               'key': 'housenumber',
               'type': 'addr',
               'value': '1412'},
              {'id': 209809850,
               'key': 'street',
               'type': 'addr',
               'value': 'West Lexington St.'},
              {'id': 209809850,
               'key': 'street:name',
               'type': 'addr',
               'value': 'Lexington'},
              {'id': '209809850',
               'key': 'street:prefix',
               'type': 'addr',
               'value': 'West'},
              {'id': 209809850,
               'key': 'street:type',
               'type': 'addr',
               'value': 'Street'},
              {'id': 209809850,
               'key': 'building',
               'type': 'regular',
               'value': 'yes'},
              {'id': 209809850,
               'key': 'levels',
               'type': 'building',
               'value': '1'},
              {'id': 209809850,
               'key': 'building_id',
               'type': 'chicago',
               'value': '366409'}]}
"""

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
    # YOUR CODE HERE
    
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
cur.execute('SELECT COUNT(*) FROM ways') #草稿里有做这个一步，检查发现报告没有，现在补上
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
    

## 关于数据集其它探索

### 菜品排名前十


```python
cur.execute('SELECT nodes_tags.value,COUNT(*) as num\
            FROM nodes_tags\
                JOIN (SELECT DISTINCT(id) FROM nodes_tags WHERE value="restaurant") i\
                ON nodes_tags.id = i.id\
            WHERE nodes_tags.key = "cuisine"\
            GROUP BY nodes_tags.value\
            ORDER BY num DESC\
            LIMIT 10' )
pprint.pprint (cur.fetchall())
```

    [(u'pizza', 42),
     (u'american', 36),
     (u'chinese', 31),
     (u'italian', 31),
     (u'mexican', 30),
     (u'indian', 22),
     (u'thai', 19),
     (u'asian', 15),
     (u'japanese', 14),
     (u'regional', 12)]
    

### nodes中可通行轮椅占比


```python
cur.execute('SELECT COUNT(*) \
            FROM nodes_tags \
            WHERE key="wheelchair" AND value="yes"')
print cur.fetchall()
```

    [(191,)]
    


```python
cur.execute('SELECT COUNT(*) \
            FROM nodes_tags \
            WHERE key="wheelchair"')
print cur.fetchall()
```

    [(308,)]
    


```python
r = 191.0 / 308
print "%.3f%%" % (r * 100)
```

    62.013%
    

### nodes中公共场所中设有轮椅道占比


```python
cur.execute('SELECT COUNT(*) \
            FROM nodes_tags \
            WHERE key="amenity" ')
print cur.fetchall()
```

    [(5076,)]
    


```python
cur.execute('SELECT COUNT(*) \
            FROM nodes_tags \
            WHERE key="wheelchair"')
print cur.fetchall()
```

    [(308,)]
    


```python
r = 308.0/5076
print "%.3f%%" % (r % 100)
```

    0.061%
    

### 引用：课程中习题的代码；
