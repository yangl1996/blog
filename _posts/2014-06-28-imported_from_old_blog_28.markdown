---
layout: post
title: 用Python的minidom解析XML，以Wikipedia Miner为例
date: '2014-06-28 04:41:27'
---

<p>最近需要用Python解析XML，用minidom进行解析。</p>
<p>需要解析的是<a href="http://wikipedia-miner.cms.waikato.ac.nz" title="Wikipedia Miner" target="_blank">Wikipedia Miner</a>这个服务返回的XML文件，有几种不同结构的XML文件。这次是用的是xml.dom.mimidom这个模块，存在于Python标准库当中，所以调用非常方便。</p>
<p>首先import这个库。<code>import xml.dom.minidom</code></p>
<p>需要解析的文件应当以DOM对象形式存在，如此导入：
<code>xmlfile = xml.dom.minidom.parse(sourcefile)
rootnodes = xmlfile.documentElement</code>
其中sourcefile是XML文件，可以下载于Internet，也可以是本地文件。我需要直接操作来源于Wikipedia Miner的Web服务的XML文件，所以之前先用下面代码获取文件：
<code>from urllib2 import Request, urlopen
req = Request('http://wikipedia-miner.cms.waikato.ac.nz/services/search?query=hiking%20new%20zealand&complex=true')
try:
  sourcefile = urlopen(req)
except IOError, e:
  if hasattr(e, 'reason'):
    print 'We failed to reach a server.'
    print 'Reason: ', e.reason
  elif hasattr(e, 'code'):
    print 'The server couldn't fulfill the request.'
    print 'Error code: ', e.code
sourcefile = urlopen(req)</code>
这段就是返回<a href="http://wikipedia-miner.cms.waikato.ac.nz/services/search?query=hiking%20new%20zealand&complex=true" title="http://wikipedia-miner.cms.waikato.ac.nz/services/search?query=hiking%20new%20zealand&complex=true" target="_blank">http://wikipedia-miner.cms.waikato.ac.nz/services/search?query=hiking%20new%20zealand&complex=true</a>处（这是一个例子）的XML文件，并存于变量sourcefile中。</p>
<p>首先解析底层对象
<code>rootnodes = xmlfile.documentElement#The root layer
  labelnodes = rootnodes.getElementsByTagName('label')#Generate a list and store all 'label' nodes</code>
上面代码首先读取文档元素，然后以此为基础，找出下面一层中标签名为“label”的元素，并且储存于一个列表中。之后我们如果需要遍历所有标签名为“label”的元素，那只需遍历这个列表就可以了。</p>
<p>Wikipedia Miner接收到一个搜索请求（本例中是搜索“hiking new zealand”），然后对输入的字符串进行分词，分词后的每一个最小单位就储存于一个label元素中。也就是我们刚刚解析并存在一个列表里面的东西。下面遍历这个列表，提取出每个label的搜索结果。</p>
<p>label下是senses元素，senses元素下是sense元素。每个sense元素分别代表一个可能的相关Wikipedia条目。我们需要对每个label获得它的每个sense，再有组织地存储，这样就达到了调用Wikipedia Miner服务的目的。</p>
<p>思路就是遍历每个label，对每个label元素，读取并存储senses元素。对senses元素，读取并存储每个sense元素。之后遍历sense元素，获得我们需要的属性（id、title等等）并存储。
<code>result = []
  for labels in labelnodes:#Go over all labels
    labelinfo = {}#Init a dict
    labelinfo['labelName'] = labels.getAttribute('text')
    labelinfo['linkProbability'] = float(labels.getAttribute('linkProbability'))#Store basic info in the dict
    labelinfo['labelSenses'] = []
    labelSensesTag = labels.getElementsByTagName('senses')#Peel the first layer
    labelSenseList = labelSensesTag[0].getElementsByTagName('sense')#Finally get the sense list of a label</code>
上面的代码中，先创建一个列表用于存储最终的结果。然后开始对label遍历。对其中每个label，创建一个叫labelinfo的字典用于分门别类存储信息。用<code>.getAttribute()</code>来获得label元素的各个属性，并存入labelinfo字典。而对于labelSenses的值，则又需要提取label下的senses元素，再提取senses元素下的sense元素，对sense元素进行遍历才可得到。所以，上面代码中，labelSenseList最终是一个存储了各个sense元素的列表。</p>
<p>接下来开始遍历sense元素。
<code>for senses in labelSenseList:#Go over all possible senses of a label
      senseinfo = {}#Create a dict to store info of a sense
      senseinfo['artID'] = int(senses.getAttribute('id'))#Get every piece of info and add to the dict
      senseinfo['artTitle'] = senses.getAttribute('title')
      senseinfo['probability'] = float(senses.getAttribute('priorProbability'))
      senseinfo['weight'] = float(senses.getAttribute('weight'))
      labelinfo['labelSenses'].append(senseinfo)</code>
对每个sense元素，先建立senseinfo字典准备存储，在存入所有信息之后，再通过<code>.append()</code>将senseinfo字典作为上面labelinfo['labelSenses']列表的一个值存入其中。</p>
<p>到这里为止，我们已经提取了所有有需要的信息，并以一个合理的，方便调用的结构存在内存中。</p>
<p>观察整个处理过程，除了最初的导入模块和需要处理的文档之外，其他操作需要按照不同的XML文档的内容来改变。基本上，同类的元素用列表来存储，一个元素的不同属性（或值，本例中没有涉及到值的读取）用字典来存储。同时，使用<code>.getElementsByTagName()</code>来获得下层的元素，也可以通过<code>.childNodes</code>（本例中未涉及到）来直接获得子元素的列表。
在这里有一个易错点。无论查找到符合条件的子元素有多少个，<code>.getElementsByTagName()</code>返回的都是一个列表，所以不要忘记用<code>list[0]</code>来读取真正的XML元素。当列表中只有一项时，这个操作还是容易忘记的</p>
<p>完整例子：<a href="https://github.com/yangl1996/WikiHack/blob/master/xmlextract.py" target="_blank">https://github.com/yangl1996/WikiHack/blob/master/xmlextract.py</a></p>