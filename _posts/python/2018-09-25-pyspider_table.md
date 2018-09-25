---
layout: post
title: "pyspider爬取并解析网页中table数据的例子"
keywords: ["python"]
description: "python"
category: "python"
tags: ["python","python"]
---

主要爬取一些第三方接口参数，字段，说明信息，有很多个下面这种类似的页面，这些都是table数据
```
<table class="confluenceTable">
    <colgroup>
     <col style="width: 130.0px;" />
     <col style="width: 77.0px;" />
     <col style="width: 384.0px;" />
    </colgroup>
    <tbody>
     <tr>
      <th class="confluenceTh">参数</th>
      <th class="confluenceTh">是否必填</th>
      <th class="confluenceTh">说明</th>
     </tr>
     <tr>
      <td class="confluenceTd">appid</td>
      <td class="confluenceTd">Y</td>
      <td class="confluenceTd"><br /></td>
     </tr>
     <tr>
      <td class="confluenceTd"><span>idNo</span></td>
      <td class="confluenceTd">Y</td>
      <td class="confluenceTd"><span>证件号码</span></td>
     </tr>
     <tr>
      <td class="confluenceTd"><span>name</span></td>
      <td class="confluenceTd"><span>Y</span></td>
      <td class="confluenceTd"><span>姓名</span></td>
     </tr>
     <tr>
      <td colspan="1" class="confluenceTd"><span>photoStr</span></td>
      <td colspan="1" class="confluenceTd"><span>Y</span></td>
      <td colspan="1" class="confluenceTd"><span>照片文件</span><br /><span>注意：原始图片不能超过 2M，且必须为 JPG 或 PNG 格式</span></td>
     </tr>
     <tr>
      <td colspan="1" class="confluenceTd"><span>sourcePhotoStr</span></td>
      <td colspan="1" class="confluenceTd"><span>Y</span></td>
      <td colspan="1" class="confluenceTd"><span>合作伙伴自己提供的可信比对源照片</span><br /><span>注意：原始图片不能超过 2M，且必须为 JPG 或 PNG 格式</span></td>
     </tr>
     <tr>
      <td colspan="1" class="confluenceTd"><span>sourcePhotoType</span></td>
      <td colspan="1" class="confluenceTd"><span>Y</span></td>
      <td colspan="1" class="confluenceTd"><span>比对源照片类型</span><br /><span>1：网纹照</span><br /><span>2：高清照</span></td>
     </tr>
     <tr>
      <td colspan="1" class="confluenceTd">transationid</td>
      <td colspan="1" class="confluenceTd">Y</td>
      <td colspan="1" class="confluenceTd">每次请求唯一的标识（和第三方接口中的orderno对应）</td>
     </tr>
    </tbody>
   </table>
```

  原本用pyspider的response应该就可以了，response.doc返回的本来就是PyQuery，于是遍历所有tr,然后遍历tr内的td获取text就好了，     
  发现有的td内部有span标签的会获取不到数据，有很多td的text本来就是，又不能直接用doc获取整个页面的td计算，也想过直接用etree,   
没招了，只好试一试BeautifulSoup

详细代码

```
from bs4 import BeautifulSoup
from lxml.html import etree
import lxml.html
for item in response.doc('.confluenceTable').items():
    table_header_dict={}
    root = lxml.html.fromstring(item.html())
    soup = BeautifulSoup(item.html())
    for row in soup.findAll("tr"):
        # 获取tr内的th,得到表头信息
        th_cells = row.find_all("th")
        if len(th_cells)>0 :
            table_header_dict = self.get_table_header_dict(th_cells)
    #解析tr内的td,解析表格数据
    trs = soup.findAll("tr");
    td_dict_data = self.parse_tddata_to_dict(table_header_dict,trs)        
```
# 获取tr内的th,得到表头信息

```
#获取表头信息
def get_table_header_dict(self,th_cells):
    th_count = len(th_cells)
    #print th_count
    th_dict={}
    index =0
    for i in range(th_count):
        temp = th_cells[i].text
        if ((temp =='参数')|(temp =='参数名')|(temp =='参数名称')|(temp =='名称')|(temp =='字段')|(temp =='字段名')|(temp='字段名称')):
            th_dict[i]= 'name'
        elif ('类型' in temp ):
            th_dict[i]= 'type'   
        elif ((temp=='说明')|(temp=='备注')|(temp='参数说明')|(temp='字段说明')):
            th_dict[i]= 'desc'
        else :
            th_dict[i]= temp
    return th_dict
```

#解析tr内的td,解析表格数据
```
def parse_tddata_to_dict(self,table_header_dict,trs):
    ret_list = []
    retuslt = {}
    for row in trs:
            # 获取表格内的所有td
            cells = row.findAll("td")
            if len(cells)>0 :
                index = 0
                for i in range(len(cells)):
                       #直接通过text得到文本内容
                        print cells[index].text
                        retuslt[table_header_dict.get(index)] = cells[index].text
                        index = index+1

                ret_list.append(retuslt)
    #print len(ret_list)
    #print ret_list
    return ret_list;
```

直接用PyQuery获取不到不知道为啥，

```
for item in response.doc('.confluenceTable').items():
for i in range(len(item('td'))):
        field_name = th_dict[i%len(th_dict)]
       # print i,th_count,th_dict[i%th_count]
       #这个 item('td')[i].text很多时候获取不到数据
        if item('td')[i] is not None and item('td')[i].text is not None:

            index = i%len(th_dict);
            field_name = th_dict[index]
            if 'name' in field_name or 'type' in field_name or 'desc' in field_name:
                td_data[field_name]=item('td')[i].text.encode("utf-8")
        else:
            print th_dict[i%len(th_dict)] +' no data'
            if 'name' in field_name or 'type' in field_name or 'desc' in field_name:
                field_name = th_dict[i%len(th_dict)]
                td_data[field_name]='null'

        if i>0 and i%len(th_dict) ==0:
            print td_data
            td_data_list.append(td_data)
            td_data = {}
            td_data['url'] =url
            td_data['title'] =title



    print td_data_list
```
用lxml.html应该也可以的

```
for item in response.doc('.confluenceTable').items():
    root = lxml.html.fromstring(item.html())
    print "th  header"
    header_data = root.xpath('//tr/th//text()')
    print '======================='
    print 'len(header_data):'
    print len(header_data)
    print "td data"
    td_data = root.xpath('//tr/td//text()')
    '''
    for row in root.xpath('//tr/td//text()'):
        print([i for i in row.itertext()])
    '''


    print 'len(td_data):'
    print len(td_data)
    print  '======================='
```
