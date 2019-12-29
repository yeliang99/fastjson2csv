# fastjson2csv
利用fastjson，将json数组转换为csv，并输出文件
##### 引言：
fastjson目前我了解好像是没有支持将 `json` 转为 `csv` 的，网上大多是用`org.json`的 `CDL.toString` 来将`json`数组转化为`csv`的，但是在我这出现两个问题：
###### 1.  表头指定的顺序会变：
我的`json`数组：
`[{"name":"LiMing","age":"28","gender":"man"},{"name":"LiPing","age":"26","gender":"women"}]`
我用`CDL.toString` 结果得到的是：

```java
gender,name,age
women,LiPing,26
man,LiMing,28
```
但我们其实想要，名字在前面：

```java
name,age,gender
LiMing,28,man
LiPing,26,women
```
这是因为，`CDL.toString` 是根据 传入的json数组中的JSONObject来处理的，而`org.json`的`JSONObject`又是按`a-z`顺序来排序，那么问题就变成了，如何让`org.json`的 `JSONObject` 有序，可以看到他的构造方法中，默认是`HashMap`， 将其改为`LinkedHashMap`即可保持有序，

```java
    public JSONObject() {
        this.map = new LinkedHashMap();
    }
```

但是改源码很不方便，并且还要自己打包使用，不知道为什么`org.json`自己不出一个`orderJSONObject`呢。
###### 2. 项目中都是用的fastjson，用两个容易混乱
即使按上一步所说的修改了源码，也会在同一个项目中用到两个json包，特别不利于维护以及引用时容易搞错。并且`fastjson`支持传入参数指定一个有序的`JSONObject`，这一点比`org.json`好多了，但是`fastjson`不支持转化为`csv`，所以要写一个将`com.alibaba.fastjson.JSONArray`转化为`csv`的方法，主要就是参考`org.json`的 `CDL.java`了。
##### 解决：
以下是代码：
使用示例：

```java
         JSONArray array = new JSONArray();
         
        JSONObject object2 = new JSONObject(new LinkedHashMap<>()); // 传入LinkedHashMap指定有序
        object2.put("name","LiMing");
        object2.put("age","28");
        object2.put("gender","man");
        
        JSONObject object = new JSONObject(new LinkedHashMap<>());
        object.put("name","LiPing");
        object.put("age","26");
        object.put("gender","women");
        
        array.add(object);
        array.add(object2);

        String s = ConvertCsv.toString(array);
        FileUtils.writeStringToFile(new File("E:\\hello.csv"), s);
```

ConcertCsv.java 工具类 
