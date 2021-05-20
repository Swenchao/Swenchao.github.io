---
title: Hive自定义函数
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-29 19:09:55
password:
summary: Hive自定义函数（UDF UDTF）
tags:
- Hive
categories:
- 大数据
---

# Hive

## 函数

### 系统内置函数

1. 查看系统自带的函数

```sql
hive> show functions;
```

2. 显示自带的函数的用法

```sql
hive> desc function upper;
```

3. 详细显示自带的函数的用法

```sql
hive> desc function extended upper;
```

### 自定义函数

1. Hive 自带了一些函数，比如：max/min 等，但是数量有限，自己可以通过自定义 UDF 来方便的扩展。

2. 当 Hive 提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数(UDF：user-defined function)。

3. 根据用户自定义函数类别分为以下三种：

(1) UDF(User-Defined-Function)

一进一出（给一条数据返回一条数据）

(2) UDAF(User-Defined Aggregation Function)

聚集函数，多进一出

类似于：count/max/min

(3) UDTF(User-Defined Table-Generating Functions)

一进多出

类似于：lateral view explore()

4. [官方文档地址](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)

5. 编程步骤：

(1) 继承 Hive 提供的类 org.apache.hadoop.hive.ql.UDF

(2) 需要实现 evaluate 函数；evaluate 函数支持重载；

(3) 在 hive 的命令行窗口创建函数

a) 添加 jar

add jar linux_jar_path

b) 创建 function，

create [temporary] function [dbname.]function_name AS class_name;

(4) 在 hive 的命令行窗口删除函数

Drop [temporary] function [if exists] [dbname.]function_name;

6. 注意事项

(1) UDF 必须要有返回类型，可以返回 null，但是返回类型不能为 void；

### 自定义 UDF 函数

需求

自定义一个 UDF 实现计算给定字符串的长度，例如：

```sql
hive(default)> select my_len("abcd")
——>
4
```

1. 创建一个 Maven 工程 Hive

2. 导入依赖

```xml
<dependencies>
		<!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>3.1.2</version>
		</dependency>
</dependencies>
```

3. 创建一个类

```java
package com.swenchao.udf;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

/**
 * @Author: Swenchao
 * @Date: 2020/11/29 12:13
 * @Description:
 * @Modified: NULL
 * @Version: 1.0
 */
public class MyUDF extends GenericUDF {

    // 校验数据参数个数
    @Override
    public ObjectInspector initialize(ObjectInspector[] objectInspectors) throws UDFArgumentException {
        if (objectInspectors.length != 1) {
            throw new UDFArgumentException("参数个数不对");
        }
        return PrimitiveObjectInspectorFactory.javaIntObjectInspector;
    }

    // 处理数据
    @Override
    public Object evaluate(DeferredObject[] deferredObjects) throws HiveException {

        // 取出输入数据
        String input = deferredObjects[0].get().toString();

        // 判断输入数据是否为null
        if (input == null) {
            return 0;
        }

        // 返回数据长度
        return input.length();
    }

    @Override
    public String getDisplayString(String[] strings) {
        return "";
    }
}
```

4. 打成 jar 包上传到服务器/opt/module/hive/lib/udf.jar

5. 将 jar 包添加到 hive 的 classpath

```sql
hive (default)> add jar /opt/module/hive/lib/udf.jar;
```

6. 创建临时函数与开发好的 java class 关联

```sql
hive (default)> create temporary function myLen as "com.swenchao.udf.MyUDF";
```

7. 即可在 hql 中使用自定义的函数 strip

```sql
hive (default)> select ename, mylower(ename) lowername from emp;
```

### 自定义 UDTF 函数

需求

自定义一个 UDTF 实现将一个任意分割符的字符串切割成独立的单词，例如：

```sql
hive(default)> select myudtf(("hello,world,hadoop,hive", ",");
——>
hello
world
hadoop
hive
```

1. 代码实现代码实现

```java
package com.swenchao.udf;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: Swenchao
 * @Date: 2020/11/29 13:07
 * @Description:
 * @Modified: NULL
 * @Version: 1.0
 * 输入：hello,hadoop,hive
 * 输出：
 *      hello
 *      hadoop
 *      hive
 */
public class MyUDTF extends GenericUDTF {

    // 输出数据集合
    private ArrayList<String> outPutList = new ArrayList<>();

    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {

        // 输出数据默认列明，可以被别名覆盖
        List<String> fieldNames = new ArrayList<>();
        fieldNames.add("word");

        // 输出数据类型
        List<ObjectInspector> fieldOIs = new ArrayList<>();
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        // 最终返回值
        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);

    }

    /**
     * 处理输入数据
     * @param objects
     * @throws HiveException
     */
    @Override
    public void process(Object[] objects) throws HiveException {
        // 取出数据
        String input = objects[0].toString();

        // 按","分隔字符串
        String[] words = input.split(",");

        // 遍历写出
        for (String word:words) {
            // 清空集合
            outPutList.clear();

            // 将数据放入集合
            outPutList.add(word);

            // 输出数据
            forward(outPutList);
        }
    }

    /**
     * 收尾方法
     * @throws HiveException
     */
    @Override
    public void close() throws HiveException {

    }
}
```

2. 打成打成 jar 包上传到服务器包上传到服务器/opt/module/hive/lib/udtf.jar

3. 将 jar 包添加到 hive 的 classpath 下

```sql
hive (default)> add jar /opt/module/hive/lib/udtf.jar;
```

4. 创建临时函数与开发好的 java class 关联

```sql
hive (default)> create temporary function myudtf as "com.swenchao.hive.MyUDTF";
```

5. 使用自定义的函数

```sql
hive (default)> select myudtf("hello,world,hadoop,hive",",");
```