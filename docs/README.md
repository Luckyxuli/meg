# 病人事件图谱

病人事件图谱是一种新的电子病历数据表示模型，定义了用于临床研究的医疗实体、医疗事件和事件之间的时序关系，能够解决不同医院异质电子病历的数据融合问题，为临床应用在大规模电子病历数据上开展提供了基础支撑。

![](images\schema.png)

### 介绍

不同医院的电子病历数据在结构和内容上都存在较大差别，数据融合的困难阻碍了临床研究在大规模病历数据上的开展。另外现有工作大多聚焦在局部病历文本的结构化或与知识库的链接上，忽略了电子病历数据本身潜在的价值，如医疗事件的时序关系可以反映病人的健康变化，从而推断出病因或进行疗效分析。针对多源异质电子病历数据融合难的特点，我们提出了一种新的电子病历数据表示模型，并且展示了基于该模型来构建数据集的流程。为了证明本方法的有效性，我们使用多家医院部分严格脱敏后的电子病历数据构建了数据集，同时提供了在线访问该数据集的途径和查询示例。

### 数据下载

我们的示例数据集发布在[OpenKG](http://www.openkg.cn/)上供大家使用，如需使用全部数据请发送邮件（注明单位和使用目的）至615877848@qq.com。

### 在线访问

我们使用[Apache Fuseki](http://jena.apache.org/documentation/fuseki2/index.html)工具实现了SPARQL站点来访问[病人事件图谱](https://peg.ecustnlplab.com/dataset.html)。SPARQL站点提供了一个输入框用于用户输入SPARQL查询，点击执行按钮后在界面上会返回相应的结果。

![](images\online_query.png)

### 查询示例

SPARQL前缀：

`PREFIX sem: <http://semanticweb.cs.vu.nl/2009/11/sem/>`

`PREFIX peg-o: <http://peg.ecustnlplab.com/ontology#> `

`PREFIX peg-r: <http://peg.ecustnlplab.com/resource/>`



+ 例1（**患有2型糖尿病和冠心病患者的性别比例**）

  SPARQL：

+ 例2（**被诊断位高血压性心脏病后又患有心肾阳虚证的病人**）

  SPARQL：

+ 例3（**服用安络血片期间，糖尿病病人尿素氮的变化**）

  SPARQL：

+ 例4（**患有气阴两虚证的患者使用利多卡因针后淋巴细胞的平均值**）

  SPARQL：

+ 例5（**被诊断为II型呼衰且死亡的患者**）

  SPARQL：

+ 例6（**患有中风病后服用了开博通片进行治疗，之后又被诊断为脑栓塞且死亡的患者**）

  SPARQL：

+ 例7（**患有胃癌，服用了优福定片的女性患者**）

  SPARQL：

  `SELECT (COUNT(DISTINCT ?patient) AS ?patientCount)`

  `WHERE { `

  ` ?patient rdf:type peg-o:Patient ; peg-o:gender "女" .`

  `?diagEvent rdf:type peg-o:DiagnosisEvent ; sem:hasActor ?patient; sem:hasActor ?disease .`

  `?disease rdfs:label "胃癌" .`

  `?drugEvent rdf:type peg-o:DrugEvent; sem:hasActor ?patient; sem:hasActor ?drug .`

  ` ?drug rdfs:label "优福定片" .}`

+ 例8（**有多少病人被诊断为冠心病，随后服用了卡普托利进行治疗，在服药期间球蛋白显示正常？**）

### 本体

+ 命名空间

  http://peg.ecustnlplab.com/ontology#

+ 事件类

  

+ 事件类属性

+ 术语类

+ 术语类属性

### 相关链接