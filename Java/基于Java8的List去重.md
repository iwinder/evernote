---
title: Java8的List<Object>去重
tags: 基础,List
grammar_cjkRuby: true
---
假设Object为User,此处User类中省略getting/setting以及相关构造方法。
```
public class User {
	private Long id;
	private String name;
	private Windcoder com;
	......
}
```
现有```List<User> userLiset1```与```List<User> userList2```两个List。
## 重写equals()
最简单的就是重写User的equals()方法和hashCode()方法:
```
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        User user = (User) o;

        if (!id.equals(user.id)) return false;
        return name.equals(user.name);

    }

    @Override
    public int hashCode() {
        int result = id.hashCode();
        result = 31 * result + name.hashCode();
        return result;
    }
```
之后通过HashSet或stream+contains去重。List 的contains()方法底层实现使用对象的equals方法去比较的，其实重写equals()就好，但重写了equals最好将hashCode也重写了。

set接口是通过equals来判断是否重复的，HashSet是一种加快判断效率的一种实现，先通过hashCode判断(hashCode通过运算求出数组下标，通过下标判断是否有对象存在)，如果重复，再equal比较。故用HashSet去重时必须重写这两个方法。

### HashSet去重
假设只对userLiset1去重，先将userLiset1转为HashSet，再转回List即可：
```
Set<User> us = new HashSet();
us.addAll(userLiset1);
List<User> newUsers = new ArrayList<User>(us);
```
### stream去重
此为Java8始有的方式stream+lambdas：
```
List<user> newUsers = new ArrayList<User>();
userLiset1.stream().forEach(
		user -> {
			if (!personList.contains(user)) {
				newUsers.add(user);
			}
		}
);
```
## 不重写equals()

实际上很多时候项目中不能或不允许重写对象的equals()，此时去重的核心就是通过TreeSet或ConcurrentSkipListSet去重，两者主要区别是后者为线程安全的。

### 非stream形式
```
Set<User> userSet = new TreeSet<User>(new Comparator<User>(){
	 @Override
	 public int compare(User o1, User o2) {
		  return o1.getId().compareTo(o2.getId()); 
	 }
}) ;
userSet.addAll(userLiset1);
List<User> newUsers = new ArrayList<User>(userSet);
```
### stream形式
```
List<User> newUsers =  userLiset1.stream().collect(
	Collectors.collectingAndThen(Collectors.toCollection(
		() -> new TreeSet<>(
			Comparator.comparing(User::getId))), ArrayList::new)
);
```
若根据User下的Windcoder的id做比较，可以将上面中的Comparator.comparing比较条件改为：
```
Comparator.comparing( user->user.getCom().getId()))), ArrayList::new)
```
collect() 操作会把其接收的元素聚集（aggregate）到一起（这里是 List）。

### 两个List合并及去重
可以使用thenComparing对判重条件进行追加，程序会自动依次对比。
```
Comparator<User> c=Comparator.comparing(user->user.getCom().getId())
						.thenComparing(User::getName);
List<User> result = Stream.concat(userLiset1.stream(), userLiset2.stream())
    			                .filter(new ConcurrentSkipListSet<>(c)::add)
    			                .collect(Collectors.toList());
```
- concat用于合并，这里是合并两个```List<User>```流。

- Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串，这里返回的是List;

- filter 方法用于通过设置的条件过滤出元素,这里相当于过滤掉重复的User，重复的后者将被舍弃。

## 参考资料

1. [Java 8 根据属性值对列表去重](https://blog.csdn.net/wang704987562/article/details/79300393)

2. [合并java 8中的两个对象列表？](https://cloud.tencent.com/developer/ask/51866)

3. [Java List\<Object\>去掉重复对象](https://blog.csdn.net/bestxiaok/article/details/75087649)