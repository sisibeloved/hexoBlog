---
title: Intellij IDEA 类模板自动添加Serializable
date: 2020-12-14 17:51:00
categories: Tools
tags:
- IDEA
- Java
---

在 `Editor->File and Code Templates->Files` 里找到 Java 的 Class，修改模板：

```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end

#set($name = ${NAME})
#if($name.toString().matches(".*[(V|P|B|DT)O|Query]"))
    #set($obj = true)
#end
#if($obj)
import java.io.Serializable;
#end
#parse("File Header.java")
public class ${NAME} #if($obj)implements Serializable #end{
}

```

<!-- more -->

其中的 `#parse("File Header.java")` 是在 `Editor->File and Code Templates->Includes` 中添加的自定义的类注释：

```
/**
 * @author xxx
 * @date ${DATE}
 */
```

