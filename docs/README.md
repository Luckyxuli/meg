# 病人事件图谱

病人事件图谱是一种新的基于RDF的医疗观察性数据表示模型，可以清晰地表示临床检查、诊断、治疗等多种事件类型以及事件的时序关系。

------

![](images\schema.png)

------

### 介绍

基于电子病历的观察性数据真实世界研究成为目前临床科研的热点。然而关系数据模型无法直接支撑起科研应用中医疗事件的时序关系表示以及知识融合的查询需求。针对上述问题，本文提出了一种新的基于RDF的医疗观察性数据表示模型，可以清晰地表示临床检查、诊断、治疗等多种事件类型以及事件的时序关系。对来源于医院的电子病历数据，使用**数据预处理**、**数据模式转换**、**时序关系构建**以及**知识融合**4个步骤建立事件图谱。具体地，使用三家上海三甲医院的电子病历数据，构建了包括**3个专科**、**173395个医疗事件**、**501335个事件时序关系**以及与**5313个知识库概念**链接的医疗数据集。

------

### 数据下载

我们的示例数据集发布在[OpenKG](http://www.openkg.cn/)上供大家使用，如需使用全部数据请发送邮件（注明单位和使用目的）至ecust_lxl@126.com。

------

### 在线访问

我们提供了SPARQL站点来访问[病人事件图谱](https://peg.ecustnlplab.com/dataset.html)。SPARQL站点提供了一个输入框用于用户输入SPARQL查询，点击执行按钮后在界面上会返回相应的结果。

![](images\online_query.png)

------

### 查询语句

~~~~sparql前缀
PREFIX sem: <http://semanticweb.cs.vu.nl/2009/11/sem/>
PREFIX peg-o: <http://peg.ecustnlplab.com/ontology#>
PREFIX peg-r: <http://peg.ecustnlplab.com/resource/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX sem: <http://semanticweb.cs.vu.nl/2009/11/sem/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX dsc: <http://dsc.nlp-bigdatalab.org:8086/>
~~~~

+ 例句一（**2012年和2013年高血压患者的年龄差异**）
~~~~sparql
  select  ?e1 ?eca1 ?e2 ?eca2
  where {
      ?e1 sem:hasActor ?p1;
          sem:hasObject ?disease ;
          peg-o:condition_age ?eca1 ;
          sem:hasTimeStamp ?et1.
      filter(?et1 > "2012-01-01"^^xsd:date || ?et1 < "2012-12-31"^^xsd:date)
      ?e2 sem:hasActor ?p2 ;
          sem:hasObject ?disease ;
          peg-o:condition_age ?eca2 ;
          sem:hasTimeStamp ?et2.
      filter(?et2 > "2013-01-01"^^xsd:date || ?et2 < "2013-12-31"^^xsd:date )
      ?disease rdfs:label "高血压病"
  }

																execute time: 0.045s
~~~~

+ 例句二（**患有双侧膝关节炎后血脂紊乱的患者的平均年龄**）
~~~~sparql
select ?age
where {
      ?p peg-o:birthdate ?p_birthdate .
      ?e1 sem:hasActor ?p;
          sem:hasObject ?disease .
      ?disease rdfs:label "双侧膝关节炎" .
  
      ?e2 sem:hasActor ?p ;
          sem:hasObject ?assay ;
          peg-o:measurement_prompt "紊乱" ;
          sem:hasTimeStamp ?et2.
      ?assay rdfs:label "血脂" .
      ?e2 peg-o:after ?e1 .
      bind(year(?et2)-year(?p_birthdate) as ?age)
}

																execute time: 0.022s
~~~~

+ 例句三（**同时被诊断为咳嗽和风寒外袭的患者住了多少次院**）
~~~~sparql
SELECT  (count(?e3)as ?e3_count)
WHERE{
    ?disease1 rdfs:label "咳嗽" .
    ?disease2 rdfs:label "风寒外袭" .
	?e1 sem:hasActor ?patient ;
        sem:hasObject ?disease1 .
	?e2 sem:hasActor ?patient ;
        sem:hasObject ?disease2 .
  	?e1 peg-o:concurrent ?e2 .
    ?e3 sem:eventType peg-o:VISIT_EVENT ;
  		sem:hasActor ?patient  .
}

																execute time: 11.302s
~~~~

+ 例句四（**患有冠心病且患有不稳定性心绞痛的人数**）
~~~~sparql
SELECT  (count(?patient)as ?patient_count)
WHERE{
	?e1 sem:hasActor ?patient ;
        sem:hasObject ?disease1 .
	?e2 sem:hasActor ?patient ;
        sem:hasObject ?disease2 .
	?disease1 rdfs:label "冠心病" .
	?disease2 rdfs:label "不稳定性心绞痛" .
  	?e1 peg-o:concurrent ?e2 .
}

																execute time: 0.030s
~~~~

+ 例句五（**首次患有冠心病，1年内又首次被诊断为气阴两虚证的患者的红细胞压积检验结果**）
~~~~sparql
SELECT ?patient ?measurement_result
WHERE{
	?e1 sem:hasActor ?patient ;
        sem:hasTimeStamp ?t1 ;
        peg-o:first_condition true ;
        sem:hasObject ?disease1 .
	?e2 sem:hasActor ?patient ;
        sem:hasTimeStamp ?t2 ;
        peg-o:first_condition true ;
        sem:hasObject ?disease2 .
	?e3 sem:hasActor ?patient ;
        sem:hasObject ?assay ;
        peg-o:measurement_result ?measurement_result ;
        sem:hasTimeStamp ?t3 .
	?disease1 rdfs:label "冠心病" .
  	?assay rdfs:label "红细胞压积" .
	?disease2 rdfs:label "气阴两虚证" .
	filter(((year(?t2)-year(?t1))*12+(month(?t2)-month(?t1))<12) && ((year(?t2)-year(?t1))*12+(month(?t2)-month(?t1))>0))
}
           
																execute time: 0.170s
~~~~

+ 例句六（**患有2型糖尿病和慢性心衰的患者的血小板分布宽度的结果**）
~~~~sparql
SELECT ?patient ?measurement_result
WHERE{
	?e1 sem:hasActor ?patient ;
        sem:hasObject ?disease1 .
	?e2 sem:hasActor ?patient ;
        sem:hasObject ?disease2 .
	?disease1 rdfs:label "2型糖尿病" .	
	?disease2 rdfs:label "慢性心衰" .
	?e3 sem:hasActor ?patient ;
        sem:hasObject ?assay ;
        peg-o:measurement_result ?measurement_result .
  	?assay rdfs:label "血小板分布宽度" .
}

																execute time: 16.489s
~~~~

+ 例句七（**在服用格华止片的六个月内，2型糖尿病患者的糖类抗原125变化趋势**）
~~~~sparql
SELECT ?patient  ?measurement_result
WHERE{
	  ?disease1 rdfs:label "2型糖尿病" .
      ?drug rdfs:label "格华止片" .
      ?assay rdfs:label "糖类抗原125" .
      ?e1 sem:hasActor ?patient ;
          sem:hasTimeStamp ?t1 ;
          sem:hasObject ?drug .
      ?e2 sem:hasActor ?patient ;
          sem:hasTimeStamp ?t2 ;
          sem:hasObject ?disease1 .
      ?e3 sem:hasActor ?patient ;
          sem:hasObject ?assay ;
          peg-o:measurement_result ?measurement_result ;
          sem:hasTimeStamp ?t3  .
      filter(((year(?t2)-year(?t1))*12+(month(?t2)-month(?t1))<6) && ((year(?t2)-year(?t1))*12+(month(?t2)-month(?t1))>0))
}
          
																execute time: 0.026s
~~~~
+ 例句八（**肝肾不足证的患者服用哪些ACEI类药期间血红蛋白水平正常**）
~~~~sparql
SELECT ?drugname
WHERE{
	?e1 sem:hasActor ?patient ;
        sem:hasObject ?disease1 .
  	?disease1 rdfs:label "肝肾不足证" .
  	?e2 sem:hasActor ?patient ;
        sem:hasObject ?drug ;
        sem:hasBeginTimeStamp ?t1 ;
        sem:hasEndTimeStamp ?t2 .
  	?drug rdfs:label ?drugname ;
          peg-o:isHyponymOf dsc:ACEI类药 .
	?e3 sem:hasActor ?patient ;
        sem:hasObject ?assay ;
        sem:hasTimeStamp ?t3 ;
        peg-o:measurement_prompt "正常" .
   	?assay rdfs:label "血红蛋白" .
  	 ?e3 peg-o:during ?e2 .
}

																execute time: 0.218s
~~~~
+ 例句九（**被诊断为肥厚性梗阻性心肌病经过药物治疗并且好转的患者**）
~~~~sparql
SELECT distinct ?patient
WHERE{
    ?disease1 rdfs:label "二尖瓣重度关闭不全" .
    ?e1 sem:hasActor ?patient ;
        sem:hasTimeStamp ?t2 ;
        sem:hasObject ?disease1 .
    ?e2 sem:hasObject ?disease1 ;
        peg-o:situation "好转" ;
        sem:hasTimeStamp ?t3 .
  	?e3 sem:eventType peg-o:TREATMENT_EVENT ;
        sem:hasActor ?patient ;
        sem:hasBeginTimeStamp ?t1 .
  	?e1 peg-o:before ?e3 .
  	?e3 peg-o:before ?e2 .
}

																execute time: 0.007s
~~~~
+ 例句十（**肝肾不足证的患者服用哪些ACEI类药期间血红蛋白水平正常**）
~~~~sparql
SELECT (count(?patient)as ?patient_count)
WHERE{
    ?e3 sem:hasActor ?patient ;
        sem:hasObject ?procedure ;
        sem:hasTimeStamp ?t3  .
  	?procedure rdfs:label "心脏临时性起博器植入术".
    ?disease1 rdfs:label "心衰病" .
	?e1 sem:hasActor ?patient ;
        sem:hasTimeStamp ?t1 ;
        sem:hasObject ?disease1 .
    ?disease2 rdfs:label "慢性心力衰竭急性加重" .
	?e2 sem:hasActor ?patient ;
        sem:hasTimeStamp ?t2 ;
        sem:hasObject ?disease2 .
 	filter(((year(?t1)-year(?t3))*12+(month(?t1)-month(?t3))>0)&&((year(?t2)-year(?t1))*12+(month(?t2)-month(?t1))>0))
}

																execute time: 0.350s
~~~~
<a href="examples.txt" target="_blank">全部问题</a>

------

### 本体

+ 

------

### 相关链接

