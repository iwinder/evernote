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

