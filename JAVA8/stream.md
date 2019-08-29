* **Stream**
    * [1.Map](#1Map)




## 1.Map

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
