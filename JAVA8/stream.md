* **Stream**
    * [1.map](#1map)
    * [2.peek](#2peek)
    * [3.distinct](#3distinct)





## 1.map

map生成的是个一对一映射,for的作用
比较常用
而且很简单

1.获取对象的某个属性集合。

	//old
	List<String> list=new ArrayList<>();
    for (PersonModel persion:data) {
        list.add(persion.getName());
    }

    //new 1
	getList().stream().map(list -> list.getXXX()).collect(toList());

	//new 2
	getList().stream().map(List::getXXX).collect(toList());


## 2.peek
对每个元素执行操作并返回一个新的 Stream

	Stream.of("one", "two", "three", "four")
 	.filter(e -> e.length() > 3)
 	.peek(e -> e.setName("new name"+_e))

## 3.distinct
列表去重使用 hashCode() 和 eqauls() 方法来获取不同的元素。
因此，需要去重的类必须实现 hashCode() 和 equals() 方法。

1.1 对于 String 列表的去重
	因为 String 类已经覆写了 equals() 和 hashCode() 方法，所以可以去重成功。
	
	//例
	stringList = stringList.stream().distinct().collect(Collectors.toList());


1.2 对于实体类列表的去重
	Lombok 插件的 @Data注解，可自动覆写 equals() 以及 hashCode() 方法。
	
	//例
	studentList = studentList.stream().distinct().collect(Collectors.toList());