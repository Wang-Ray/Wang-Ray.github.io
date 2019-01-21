---
layout: post
title: "实践Thymeleaf"
date: 2018-11-19 11:08:00 +0800
categories: Java
tags: Java Thymeleaf template-engine
---

[Thymeleaf](https://www.thymeleaf.org/) is a modern server-side Java template engine for both web and standalone environments.



## 标签

```html
<html xmlns:th="http://www.thymeleaf.org">
```

## URL

```html
<link rel="stylesheet" type="text/css" media="all" href="../../webapp/static/css/style.css" th:href="@{/static/css/style.css}"/>
    <script type="text/javascript" src="../../webapp/static/js/thymeleaf.js" th:src="@{/static/js/thymeleaf.js}"></script>
```

## 表达式

```html
<p th:text="${name}" >name</p>
<p th:utext="${htmlContent}" ></p>
<p th:text="'Hello！ ' + ${name} + '!'" >hello world</p>
<p th:text="|Hello！ ${name}!|" >hello world</p>
<p th:text="${currentProduct.price+999}" ></p>

<p th:text="${currentProduct.name}" ></p>
<p th:text="${currentProduct.getName()}" ></p>

<div class="showing" th:object="${currentProduct}">
<h2>*{}方式显示属性</h2>
<p th:text="*{name}" ></p>
</div>

<script th:inline="javascript">
    var message = [[${message}]];
    console.log(message);
</script>
```

## 包含

include.html

```html
<html xmlns:th="http://www.thymeleaf.org">
<footer th:fragment="footer1"> 
   <p >All Rights Reserved</p>
</footer>
<footer th:fragment="footer2(start,now)"> 
   <p th:text="|${start} - ${now} All Rights Reserved|"></p>
</footer>
</html>
```

test.html

```html
<div th:replace="include::footer1" ></div>
<div th:replace="include::footer2(2015,2018)" ></div>
<div th:include="include::footer1" ></div>
<div th:insert="include::footer1" ></div>
```



## 条件

```html
<p th:if="${testBoolean}" >如果testBoolean 是 true ，本句话就会显示</p>
<p th:if="${not testBoolean}" >取反 ，所以如果testBoolean 是 true ，本句话就不会显示</p>
<p th:unless="${testBoolean}" >unless 等同于上一句，所以如果testBoolean 是 true ，本句话就不会显示</p>
<p th:text="${testBoolean}?'当testBoolean为真的时候，显示本句话，这是用三相表达式做的':''" ></p>
```

## 遍历

```html
<tr th:each="p: ${ps}">
	<td th:text="${p.id}"></td>
	<td th:text="${p.name}"></td>
	<td th:text="${p.price}"></td>
</tr>

<tr th:class="${status.even}?'even':'odd'" th:each="p,status: ${ps}">
	<td th:text="${p.id}"></td>
	<td th:text="${p.name}"></td>
	<td th:text="${p.price}"></td>
</tr>

<select size="3">
	<option th:each="p:${ps}" th:value="${p.id}"     th:selected="${p.id==currentProduct.id}"    th:text="${p.name}" ></option>
</select>

<input name="product" type="radio" th:each="p:${ps}" th:value="${p.id}"  th:checked="${p.id==currentProduct.id}"     th:text="${p.name}"  />
```

## 内置工具

### Execution Info

```
${#execInfo.templateName}
${#execInfo.templateMode}
${#execInfo.processedTemplateName}
${#execInfo.processedTemplateMode}
${#execInfo.templateNames}
${#execInfo.templateModes}
${#execInfo.templateStack}
```

### Messages

```
${#messages.msg('msgKey')}
${#messages.msg('msgKey', param1)}
${#messages.msg('msgKey', param1, param2)}
${#messages.msg('msgKey', param1, param2, param3)}
${#messages.msgWithParams('msgKey', new Object[] {param1, param2, param3, param4})}
${#messages.arrayMsg(messageKeyArray)}
${#messages.listMsg(messageKeyList)}
${#messages.setMsg(messageKeySet)}
${#messages.msgOrNull('msgKey')}
${#messages.msgOrNull('msgKey', param1)}
${#messages.msgOrNull('msgKey', param1, param2)}
${#messages.msgOrNull('msgKey', param1, param2, param3)}
${#messages.msgOrNullWithParams('msgKey', new Object[] {param1, param2, param3, param4})}
${#messages.arrayMsgOrNull(messageKeyArray)}
${#messages.listMsgOrNull(messageKeyList)}
${#messages.setMsgOrNull(messageKeySet)}
```

### URIs/URLs

```
${#uris.escapePath(uri)}
${#uris.escapePath(uri, encoding)}
${#uris.unescapePath(uri)}
${#uris.unescapePath(uri, encoding)}
${#uris.escapePathSegment(uri)}
${#uris.escapePathSegment(uri, encoding)}
${#uris.unescapePathSegment(uri)}
${#uris.unescapePathSegment(uri, encoding)}
${#uris.escapeFragmentId(uri)}
${#uris.escapeFragmentId(uri, encoding)}
${#uris.unescapeFragmentId(uri)}
${#uris.unescapeFragmentId(uri, encoding)}
${#uris.escapeQueryParam(uri)}
${#uris.escapeQueryParam(uri, encoding)}
${#uris.unescapeQueryParam(uri)}
${#uris.unescapeQueryParam(uri, encoding)}
```

### Conversions

```
${#conversions.convert(object, 'java.util.TimeZone')}
${#conversions.convert(object, targetClass)}
```

### Dates

```html
<div class="showing date">
	<h2>格式化日期</h2>
	直接输出日期 ${now}:
	<p th:text="${now}"></p>
	默认格式化 ${#dates.format(now)}:
	<p th:text="${#dates.format(now)}"></p>
	自定义格式化 ${#dates.format(now,'yyyy-MM-dd HH:mm:ss')}:
	<p th:text="${#dates.format(now,'yyyy-MM-dd HH:mm:ss')}"></p>
</div>
```



```
${#dates.format(date)}
${#dates.arrayFormat(datesArray)}
${#dates.listFormat(datesList)}
${#dates.setFormat(datesSet)}
${#dates.formatISO(date)}
${#dates.arrayFormatISO(datesArray)}
${#dates.listFormatISO(datesList)}
${#dates.setFormatISO(datesSet)}
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}
${#dates.day(date)}                    
${#dates.month(date)}                  
${#dates.monthName(date)}              
${#dates.monthNameShort(date)}         
${#dates.year(date)}                   
${#dates.dayOfWeek(date)}              
${#dates.dayOfWeekName(date)}          
${#dates.dayOfWeekNameShort(date)}     
${#dates.hour(date)}                   
${#dates.minute(date)}                 
${#dates.second(date)}                 
${#dates.millisecond(date)}            
${#dates.create(year,month,day)}
${#dates.create(year,month,day,hour,minute)}
${#dates.create(year,month,day,hour,minute,second)}
${#dates.create(year,month,day,hour,minute,second,millisecond)}
${#dates.createNow()}
${#dates.createNowForTimeZone()}
${#dates.createToday()}
${#dates.createTodayForTimeZone()}
```

### Calendars

```
${#calendars.format(cal)}
${#calendars.arrayFormat(calArray)}
${#calendars.listFormat(calList)}
${#calendars.setFormat(calSet)}
${#calendars.formatISO(cal)}
${#calendars.arrayFormatISO(calArray)}
${#calendars.listFormatISO(calList)}
${#calendars.setFormatISO(calSet)}
${#calendars.format(cal, 'dd/MMM/yyyy HH:mm')}
${#calendars.arrayFormat(calArray, 'dd/MMM/yyyy HH:mm')}
${#calendars.listFormat(calList, 'dd/MMM/yyyy HH:mm')}
${#calendars.setFormat(calSet, 'dd/MMM/yyyy HH:mm')}
${#calendars.day(date)}                
${#calendars.month(date)}              
${#calendars.monthName(date)}          
${#calendars.monthNameShort(date)}     
${#calendars.year(date)}               
${#calendars.dayOfWeek(date)}          
${#calendars.dayOfWeekName(date)}      
${#calendars.dayOfWeekNameShort(date)} 
${#calendars.hour(date)}               
${#calendars.minute(date)}             
${#calendars.second(date)}             
${#calendars.millisecond(date)}        
${#calendars.create(year,month,day)}
${#calendars.create(year,month,day,hour,minute)}
${#calendars.create(year,month,day,hour,minute,second)}
${#calendars.create(year,month,day,hour,minute,second,millisecond)}
${#calendars.createForTimeZone(year,month,day,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,millisecond,timeZone)}
${#calendars.createNow()}
${#calendars.createNowForTimeZone()}
${#calendars.createToday()}
${#calendars.createTodayForTimeZone()}
```

### Numbers

```
${#numbers.formatInteger(num,3)}
${#numbers.arrayFormatInteger(numArray,3)}
${#numbers.listFormatInteger(numList,3)}
${#numbers.setFormatInteger(numSet,3)}
${#numbers.formatInteger(num,3,'POINT')}
${#numbers.arrayFormatInteger(numArray,3,'POINT')}
${#numbers.listFormatInteger(numList,3,'POINT')}
${#numbers.setFormatInteger(numSet,3,'POINT')}
${#numbers.formatDecimal(num,3,2)}
${#numbers.arrayFormatDecimal(numArray,3,2)}
${#numbers.listFormatDecimal(numList,3,2)}
${#numbers.setFormatDecimal(numSet,3,2)}
${#numbers.formatDecimal(num,3,2,'COMMA')}
${#numbers.arrayFormatDecimal(numArray,3,2,'COMMA')}
${#numbers.listFormatDecimal(numList,3,2,'COMMA')}
${#numbers.setFormatDecimal(numSet,3,2,'COMMA')}
${#numbers.formatDecimal(num,3,'POINT',2,'COMMA')}
${#numbers.arrayFormatDecimal(numArray,3,'POINT',2,'COMMA')}
${#numbers.listFormatDecimal(numList,3,'POINT',2,'COMMA')}
${#numbers.setFormatDecimal(numSet,3,'POINT',2,'COMMA')}
${#numbers.formatCurrency(num)}
${#numbers.arrayFormatCurrency(numArray)}
${#numbers.listFormatCurrency(numList)}
${#numbers.setFormatCurrency(numSet)}
${#numbers.formatPercent(num)}
${#numbers.arrayFormatPercent(numArray)}
${#numbers.listFormatPercent(numList)}
${#numbers.setFormatPercent(numSet)}
${#numbers.formatPercent(num, 3, 2)}
${#numbers.arrayFormatPercent(numArray, 3, 2)}
${#numbers.listFormatPercent(numList, 3, 2)}
${#numbers.setFormatPercent(numSet, 3, 2)}
${#numbers.sequence(from,to)}
${#numbers.sequence(from,to,step)}
```

### Strings

```
${#strings.toString(obj)}                          
${#strings.isEmpty(name)}
${#strings.arrayIsEmpty(nameArr)}
${#strings.listIsEmpty(nameList)}
${#strings.setIsEmpty(nameSet)}
${#strings.defaultString(text,default)}
${#strings.arrayDefaultString(textArr,default)}
${#strings.listDefaultString(textList,default)}
${#strings.setDefaultString(textSet,default)}
${#strings.contains(name,'ez')}                    
${#strings.containsIgnoreCase(name,'ez')}          
${#strings.startsWith(name,'Don')}                 
${#strings.endsWith(name,endingFragment)}          
${#strings.indexOf(name,frag)}                     
${#strings.substring(name,3,5)}                    
${#strings.substringAfter(name,prefix)}            
${#strings.substringBefore(name,suffix)}           
${#strings.replace(name,'las','ler')}              
${#strings.prepend(str,prefix)}                    
${#strings.append(str,suffix)}                     
${#strings.toUpperCase(name)}                      
${#strings.toLowerCase(name)}                      
${#strings.arrayJoin(namesArray,',')}
${#strings.listJoin(namesList,',')}
${#strings.setJoin(namesSet,',')}
${#strings.arraySplit(namesStr,',')}               
${#strings.listSplit(namesStr,',')}                
${#strings.setSplit(namesStr,',')}                 
${#strings.trim(str)}                              
${#strings.length(str)}                            
${#strings.abbreviate(str,10)}                     
${#strings.capitalize(str)}                        
${#strings.unCapitalize(str)}                      
${#strings.capitalizeWords(str)}                   
${#strings.capitalizeWords(str,delimiters)}        
${#strings.escapeXml(str)}                         
${#strings.escapeJava(str)}                        
${#strings.escapeJavaScript(str)}                  
${#strings.unescapeJava(str)}                      
${#strings.unescapeJavaScript(str)}                
${#strings.equals(first, second)}
${#strings.equalsIgnoreCase(first, second)}
${#strings.concat(values...)}
${#strings.concatReplaceNulls(nullValue, values...)}
${#strings.randomAlphanumeric(count)}
```

### Objects

```
${#objects.nullSafe(obj,default)}
${#objects.arrayNullSafe(objArray,default)}
${#objects.listNullSafe(objList,default)}
${#objects.setNullSafe(objSet,default)}
```

### Booleans

```
${#bools.isTrue(obj)}
${#bools.arrayIsTrue(objArray)}
${#bools.listIsTrue(objList)}
${#bools.setIsTrue(objSet)}
${#bools.isFalse(cond)}
${#bools.arrayIsFalse(condArray)}
${#bools.listIsFalse(condList)}
${#bools.setIsFalse(condSet)}
${#bools.arrayAnd(condArray)}
${#bools.listAnd(condList)}
${#bools.setAnd(condSet)}
${#bools.arrayOr(condArray)}
${#bools.listOr(condList)}
${#bools.setOr(condSet)}
```

### Arrays

```
${#arrays.toArray(object)}
${#arrays.toStringArray(object)}
${#arrays.toIntegerArray(object)}
${#arrays.toLongArray(object)}
${#arrays.toDoubleArray(object)}
${#arrays.toFloatArray(object)}
${#arrays.toBooleanArray(object)}
${#arrays.length(array)}
${#arrays.isEmpty(array)}
${#arrays.contains(array, element)}
${#arrays.containsAll(array, elements)}
```

### Lists

```
${#lists.toList(object)}
${#lists.size(list)}
${#lists.isEmpty(list)}
${#lists.contains(list, element)}
${#lists.containsAll(list, elements)}
${#lists.sort(list)}
${#lists.sort(list, comparator)}
${#sets.toSet(object)}
```

### Sets

```
${#sets.size(set)}
${#sets.isEmpty(set)}
${#sets.contains(set, element)}
${#sets.containsAll(set, elements)}
```

### Maps

```
${#maps.size(map)}
${#maps.isEmpty(map)}
${#maps.containsKey(map, key)}
${#maps.containsAllKeys(map, keys)}
${#maps.containsValue(map, value)}
${#maps.containsAllValues(map, value)}
```

### Aggregates

```
${#aggregates.sum(array)}
${#aggregates.sum(collection)}
${#aggregates.avg(array)}
${#aggregates.avg(collection)}
```

### IDs

```
${#ids.seq('someId')}
${#ids.next('someId')}
${#ids.prev('someId')}
```

