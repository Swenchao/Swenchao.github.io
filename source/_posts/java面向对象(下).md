---
title: java面向对象（下）
top: false
cover: false
toc: true
mathjax: true
date: 2020-03-22 16:32:42
password:
summary:
tags:
- JAVA
categories:
- JAVA
---

忘了从某天决定学学java，刚开始笔记都记在了本子上，实在不想再用电子版重新写了，就写从面向对象（下）开始写吧.
陆陆续续终于把面向对象下补充完了，后续接着更新吧~

# 面向对象（下)
	1、内部类
	2、枚举
	3、注解

## 内部类

### 理解
一个类中又完整的嵌套了另一个类结构，被嵌套的类称为内部类。外面的类称为外部类,和内部类无关的外部类称为外部其他类。

`class A{   //外部类
	String name;
	public void method(){
		class C{   //内部类（也可放在方法里面）
			String anme;
		}
		for(){
			class D{   //内部类(也可放在循环中)
				String anme;
			}
		}
	}
	class B{   //内部类
		String anme;
	}
}
class Other{   //外部其他类
}`

### 好处

可以直接访问外部类中的所有成员，包含私有的！！！

### 分类

定义在成员位置上:（类B）
	成员内部类（没有使用static修饰) 相对使用较多
	静态内部类（使用static修饰）

定义在局部位置上：（类 C D）
	局部内部类（有类名）
	匿名内部类(没有类名) 相对使用较多

### 特点

#### 成员内部类

① 类中可以有五大普通成员，不能有静态成员！（因为静态成员随着外部类的加载而加载，但成员内部类随着外部类的对象创建而加载）

② 可以添加任意访问修饰符，遵循访问权限限定！

**例：**

`class Outer{
	protected class Inner{
//		public Inner(){
//			
//		}
//		static{
//			
//		}
//		class InnerInner{
//			class InnerInner2{
//				
//			}
//			
//		}
	}
	private String name = "张翠山";
	public void method(){
	}
}`

③ 互访原则：
>内部类———>外部类
>	直接访问，包含任意成员（私有的也可以）
>外部类———>内部类
>	不能直接访问，需要通过创建内部类对象再去访问，包含任意成员（私有的也可以）
>**语法：**new Inner().方法();
>外部其他类———>内部类
>不能直接访问，需要通过创建内部类对象再去访问，只能访问在权限范围内的成员（私有的不可以！！！）
>**语法：**new Outer().new Inner().方法();


**例：**

`public class TestInner1 {
	public static void main(String[] args) {
		Outer o = new Outer();
		o.new Inner().show();  //外部其他类访问内部类，先创建对象再调成员
//	           o.new Inner().color="";  //私有的不可访问
	}
}
class Outer{
	protected class Inner{
		private String color;
		private String name="张无忌";
		public  void show(){
			System.out.println(Outer.this.name);
			method();  //内部类访问外部类，可直接访问
		}
	}
	private String name = "张翠山";
	public void method(){
		new Inner().show();  //外部类访问内部类成员方式
		new Inner().color="";  //外部访问内部，私有成员也可
	}
}`

④ 当外部类和内部类成员重名时，默认访问的是内部类的成员，遵循就近原则。如果非要访问外部类的成员，可以通过外部类名.this.成员的方式！

**例：**

`public class TestInner1 {
	public static void main(String[] args) {
	}
}
class Outer{
	protected class Inner{
		private String color;
		private String name="张无忌";
		public  void show(){
			System.out.println(Outer.this.name);  //张翠山
			System.out.println(name);  //张无忌
//			System.out.println(TestInner1.this.color);  //报错
		}
	}
	private String name = "张翠山";
	public void method(){
		new Inner().show();  //外部类访问内部类成员方式
		new Inner().color="";  //外部访问内部，私有成员也可
	}
}`

#### 静态内部类

① 类中可以有任意五大成员，包含普通和静态

② 可以添加任意访问修饰符，遵循访问权限限定！

**例：**

`class Outer2{
	private static String name="段正淳";
	//静态内部类
	public static class Inner2{
		static int age;
		public void show(){
			System.out.println(Outer2.name);
		}
		static{
		}
		public Inner2(){
		}
		static class InnerInner{
		}
	}
	//外部类的方法
	public void method(){
		new Inner2().show();
		new Inner2().age=12;
	}
}`

③ 互访原则：
>内部类———>外部类
>	可以直接访问外部类的静态成员，包含私有的，但不能直接访问外部类的普通成员（遵循静态成员的特点）
>外部类———>内部类
>	不能直接访问，必须通过创建内部类对象去访问，包含私有的！
>	**语法：**new 内部类对象().方法();
>外部其他类——>内部类
>	不能直接访问，必须通过创建内部类对象去访问，必须遵守访问权限限定，不包含私有的！
>	**语法：**new 外部类.内部类().方法();

**例：**

`public class TestInner2 {
	public static void main(String[] args) {
		new Outer2.Inner2().show();  //其他外部类访问静态内部类，类是静态的，因此不需要创建对象
	}
}
class Outer2{
	private static String name="段正淳";
	private String name1;
	//静态内部类
	public static class Inner2{
		String name = "段誉";
		private int age;
		public void show(){
//			System.out.println(Outer2.name1);  //内部类访问外部类，报错
			System.out.println(Outer2.name);  //内部类访问外部类，不能访问普通成员
			System.out.println(name);  //内部类访问外部类，不能访问普通成员
		} 
	}
	//外部类的方法
	public void method(){
		new Inner2().show();  //外部类访问内部类
		new Inner2().age=12;
	}
}`

④ 当外部类和内部类成员重名时，默认访问内部类的成员，遵循就近原则，如果非要访问外部类的成员,可以通过外部类名.成员 的方式！

**例：**

`class Outer2{
	private static String name="段正淳";
	//静态内部类
	public static class Inner2{
		String name = "段誉";
		public void show(){
			System.out.println(Outer2.name);  //段正淳（name是静态的，因此类可直接调）
			System.out.println(name);  //段誉（内部类成员）
		} 
	}
}` 

#### 局部内部类

① 类中可以有五大成员,但不能有静态成员（和类的加载顺序有关）

② 不能有访问修饰符和static修饰符；作用域：仅仅是定义它的方法或代码块中，而且遵循前向引用（先声明再使用）

**例：**

`class Outer3{
	private String name;
	public Object method(){
//		Inner3 i = new Inner3();  //会报错，因为在此之前并没有inner3
		 class Inner3{
			private String color;
			public void show(){
				System.out.println(name);
			}
		}
		 Inner3 i = new Inner3();  //前向引用
	}
}`

③ 互访原则

>内部类————>外部类
>直接访问，包含私有的
>外部类————>内部类
>只能在作用域范围内，通过创建对象并访问(包含私有的！)
>**语法：**new Inner().方法();

**补充：**
	局部内部类可以访问外部类的局部变量，但只能访问，不能更新！（只能访问外部类的final修饰的局部变量！）

	原因：局部内部类的生命周期>局部变量生命周期，所以不能直接访问一个可能已经消亡的变量。于是将局部变量复制了一个拷贝让局部内部类使用，但不能更新，因为如果更新，会导致两个变量的值不一致！（见下面例2解释）
	jdk7:要求局部内部类中访问的局部变量必须显式的声明final修饰符
	jdk8:局部内部类中访问的局部变量不用显式的声明final修饰符

**例1：**

`class Outer3{
	private String name;
	public void print(){
//		new Inner3().show();  //外部类访问内部类，报错，无法访问
	}
	public Object method(){
		int age =99;  //外部类局部变量
		final int ageTemp = 100;
		 class Inner3{
			private String color;
			public void show(){
//				System.out.println(name);  //访问外部类成员可以
//				name="john";  //修改也可以
				System.out.println(age);  //访问外部类局部变量可以
//				age++;  //修改外部类局部变量报错
				ageTemp++;  //不会报错（只能访问final修饰的）
			}
		}
		 Inner3 i = new Inner3();
		 i.show();  //外部类访问内部类
	}
}`

**例2：**

`class Outer3{
	private String name;
	public void print(){
		Object x = method();  //接下来对象x可能会有操作
	}
	public Object method(){
		int age =99;  //外部类局部变量
		 class Inner3{
			private String color;
			public void show(){
				System.out.println(age);  //访问外部类局部变量可以
//				age++;  //修改外部类局部变量报错
			}
		}
		 Inner3 i = new Inner3();
		 return i;
	}
}`

**解释：**从上面代码可知，age是method()方法中的局部变量，因此在method() 方法执行完消亡的时候，age也会随着消亡，但是Inner3却在方法结束的时候，作为参数传回了print方法，因此此时并没有消亡，甚至还会有下一步操作，所以生命周期不同。

#### 匿名内部类

**语法**

	new 类名或接口名(参数){
		//类体
	};

因为其为匿名类，因此并没有名字，所以直接new。其中，类名或接口名是其依赖的父类或接口

**功能**

创建了一个匿名内部类&创建了一个该类对象

	new  A(){
		//类体
	};

创建A的子类&创建A的子类对象

	new A();

创建A本类的对象

**例：**

`class Outer4{
	private String name;
	public void method(){
		//定义匿名内部类
		new Fly(){
			@Override
			public void fly() {
				// TODO Auto-generated method stub
			}
		};
	}
}
interface Fly{
	void fly();
}`

**特点**

① 类中可以有除了构造器之外的其他成员（属性、方法、初始化块、内部类），不可以有静态成员！

② 不能添加访问修饰符和static；作用域：仅仅在定义它的方法或代码块中使用，并且遵循前向引用的特点！

**例：**

`class Outer4{
	private String name;
	public void method(){
		int i=999;
		//定义匿名内部类
		new Fly(){
			class Inner{}
			String color;
			public void show(){
			}
			{
			}
			@Override
			public void fly() {
				// TODO Auto-generated method stub
			}
		};
		//使用1
		Fly a = new Fly(){
			@Override
			public void fly() {
				// TODO Auto-generated method stub
			}
		};
		a.fly();
		//使用2
		new Fly(){
			@Override
			public void fly() {
				// TODO Auto-generated method stub
			}
		}.fly();
	}
}
interface Fly{
	void fly();
}`

③ 互访原则：
>内部类———>外部类的成员
>直接访问，包含私有的成员
>外部类———>内部类的成员
>只能在作用域内，通过匿名内部类的对象去访问，包含私有的成员
>**语法：**父类或接口名  引用名 = 匿名内部类对象;
>引用名.方法();

**补充**
	匿名内部类可以访问外部类的局部变量，但只能读取，不能更新！（只能访问外部类的final修饰的局部变量！）
	原因：同上局部内部类
	jdk7:要求局部内部类中访问的局部变量必须显式的声明final修饰符
	jdk8:局部内部类中访问的局部变量不用显式的声明final修饰符

④ 应用场景：当做接口的实例作为实参传递给方法！

**例：**

`public class TestInner4 {
	public static void main(String[] args) {
//		method(new Fly());  //接口不可直接这么掉
//		method(new MyClass());//方式一：传统的方式（先声明一个类，然后传参）
		//匿名内部类
		method(new Fly(){
			@Override
			public void fly() {
				System.out.println("我要飞啊飞");
			}
		});
		//代表示例1：
		TreeSet set = new TreeSet(new Comparator(){
			@Override
			public int compare(Object o1, Object o2) {
				// TODO Auto-generated method stub
				return 0;
			}
		});
		//代表示例2：
		Thread t= new Thread(new Runnable(){
			@Override
			public void run() {
				// TODO Auto-generated method stub
			}
		});
	}
	public static void method(Fly a){
		a.fly();
	}
}
interface Fly{
	void fly();
}`

## 枚举  

### 枚举的理解

枚举其实就是一个类，枚举类的实例是一组限定的对象 

### 传统的方式创建枚举（单例类） 【了解】

对比

单例类
>1、构造器私有化
>2、本类内部创建对象
>3、通过public static方法，对外暴露该对象

枚举类
>1、构造器私有化	
>2、本类内部创建一组对象，添加public static修饰符，直接暴露对象（其实就是单例类中后两步的合并，若不合并，则枚举类中有几种就会有几个public static方法，比较繁琐）

**例1：**

```java
public class TestEnum1 {
	public static void main(String[] args) {
		//引用枚举类的对象
//		Gender g = new Gender();//错误
		Gender g1 = Gender.BOY;
		System.out.println(g1);//如果没有重写toString方法，则显示的是地址
	}

}
class Gender{
    //1、构造器私有化	
    private Gender(){}
    //2.本类内部创建一组对象，添加public static修饰符，直接暴露对象（因为boy和girl不会更改，所以加了final修饰符）
    public static final Gender BOY = new Gender();
    public static final Gender GIRL = new Gender();
    public String toString(){
        return "这是一个性别";
    }
}
```

**例2：**

```java
public class TestEnum1 {
    public static void main(String[] args) {
        //引用枚举类的对象
        System.out.println(Season.SPRING);
        System.out.println(Season.SUMMER);
    }
}
//简单示例2：提供有参构造
class Season{
    private String name;//季节名称
    private String description;//季节描述
    //2.本类内部创建一组对象，添加public static修饰符，直接暴露对象
    public static final Season SPRING = new Season("春天","春风又绿江大南");
    public static final Season SUMMER = new Season("夏天","接天莲叶无穷碧");
    public static final Season AUTUMN = new Season("秋天","霜叶红于二月花");
    public static final Season WINTER = new Season("冬天","千树万树梨花开");	
    private Season(String name, String description) {
        super();
        this.name = name;
        this.description = description;
    }
    public String getName() {
        return name;
    }
    public String getDescription() {
        return description;
    }
    @Override
    public String toString() {
        return "Season [name=" + name + ", description=" + description + "]";
    }
}
```

### 使用enum关键字定义枚举（jdk5.0引入）【掌握】

**例1（Gender类改写）：**

```java
//class换成enum
enum Gender{
	//匿名类对象必须放在枚举类第一行，另外匿名类每次都是public static final 的因此修饰符也可去掉；另外枚举类的类型当然都是一样的，所以gender可省略；默认都是无参构造，因此可省略构造方法，所以new Gender()可省略
	BOY,GIRL;
	/*
	//1、构造器私有化	
	private Gender(){}
	//2.本类内部创建一组对象，添加public static修饰符，直接暴露对象（因为boy和girl不会更改，所以加了final修饰符）
	public static final Gender BOY = new Gender();
	public static final Gender GIRL = new Gender();
	*/
	public String toString(){
		return "这是一个性别";
	}
}
```

**例2（使用）：**

```java
public class TestEnum2 {
    public static void main(String[] args) {
        //引用枚举类的对象
        System.out.println(Gender2.GIRL);//默认调用toString()方法，此时此方法已经在enum类中进行了改写，返回的是匿名类对象名字（GIRL）
    }
}
```

**例3（Season类改写）：**

```java
enum Season2{
    //2.本类内部创建一组对象，添加public static修饰符，直接暴露对象
    //有参构造
    SPRING("春天","春风又绿江大南"),
    SUMMER ("夏天","接天莲叶无穷碧"),
    AUTUMN ("秋天","霜叶红于二月花"),
    WINTER("冬天","千树万树梨花开");
    private String name;
    private String description;
    //1、构造器私有化	
    private Season2(String name, String description) {
        this.name = name;
        this.description = description;
    }
    public String getName() {
        return name;
    }
    public String getDescription() {
        return description;
    }
}
```

特点：

>1、使用enum关键字代替class关键字
>
>2、对象（常量）的创建必须放在枚举类中的第一句
>**语法：** 对象名(实参列表),对象名(实参列表);
>
>3、如果是无参构造，则无参构造的定义和实参列表都可以省略

### 介绍枚举类的常见方法【了解】

**toString：**Enum类已经重写过了，返回的是当前对象的常量名。自定义的枚举类可以继续重写该方法

**name：**Enum类中的name方法返回的是当前对象的常量名（同toString），但自定义的枚举类不可以继续重写该方法

**values：**一个静态方法，用于返回指定的枚举类中的所有枚举常量
	
**valueOf：**一个静态方法，将一个有效的字符串转换成枚举对象

**例：**

```java
public class TestEnum3 {

    public static void main(String[] args) {
//		System.out.println(Color2.RED.toString());
//		System.out.println(Color2.RED.name());
        //返回Color2中所有的枚举常量
        Color2[] values = Color2.values();
        //输出所有枚举常量
		for(int i=0;i<values.length;i++){
//			System.out.println(values[i].name());
//		}
		//将字符串转换成枚举对象（RED变量必须是枚举类存在的）
        Color2 c = Color2.valueOf("RED");
        System.out.println(c.name());
    }
}
enum Color2{
    RED(255,0,0),
    BLUE(0,0,255),
    BLACK(0,0,0),
    YELLOW(255,255,0),
    GREEN(0,255,0);
    private int redValue;
    public int getRedValue() {
        return redValue;
    }
    public int getGreenValue() {
        return greenValue;
    }
    public int getBlueValue() {
        return blueValue;
    }
    private int greenValue;
    private int blueValue;
    private Color2(int redValue, int greenValue, int blueValue) {
        this.redValue = redValue;
        this.greenValue = greenValue;
        this.blueValue = blueValue;
    }
//	public String toString(){
//		return redValue+"\t"+greenValue+"\t"+blueValue;
//	}
}
```


**练习**

声明Week枚举类，其中包含星期一至星期日的定义；
在TestWeek类中声明方法中printWeek(Week week)，根据参数值打印相应的中文星期字符串。
在main方法中接受一个命令行参数，代表星期一至星期日，打印该值对应的枚举值，然后以此枚举值调用printWeek方法，输出中文星期。

```java
public class TestEnum2 {
    public static void main(String[] args) {
        //将传入的参数转换成枚举类型
        String s =args[0];
        Week week = Week.valueOf(s);
        printWeek(week);
    }
    public static void printWeek(Week week){
        switch(week){//switch中判断的变量的类型：int、char、byte、short、String、枚举
        case MONDAY:System.out.println("星期一");break;
        case TUESDAY:System.out.println("星期二");break;
        case WEDNESDAY:System.out.println("星期三");break;
        case THURSDAY:System.out.println("星期四");break;
        case FRIDAY:System.out.println("星期五");break;
        case SATURDAY:System.out.println("星期六");break;
        case SUNDAY:System.out.println("星期日");break;
        }
    }
}
enum Week{
    MONDAY(1),
    TUESDAY(2),
    WEDNESDAY(3),
    THURSDAY(4),
    FRIDAY(5),
    SATURDAY(6),
    SUNDAY(7);
    private int value;
    private Week(int value) {
        this.value = value;
    }
    public int getValue() {
        return value;
    }
}
```


### 枚举类如何实现接口【掌握】

特点：

1.和普通类实现接口一样，只是允许枚举常量也有自己对抽象方法的特有实现！

**语法**

    enum A implements 接口1，接口2{
        常量1(参数){
        //特有抽象方法的实现
        },
        常量2(参数){
        //抽象方法的实现
        }
        //类对抽象方法的实现
    }

**例：**


```java
public class TestEnum4 {
    public static void main(String[] args) {
        Show s = Color4.RED;
        s.display(); //调用自己特有重新
        Color4.BLACK.display(); //调用枚举类中重写
    }
}
interface Fly{
	void fly();
}
interface Show{
    void display();
}
//可多实现接口
enum Color4 implements Show,Fly{
    RED(255,0,0){
    	//对抽象方法特有实现
        public void display(){
            System.out.println("我是红色");
        }
    },
    BLUE(0,0,255){
        public void display(){
            System.out.println("我是蓝色");
        }
    },
    Green(0,255,0){
    	public void display(){
    		System.out.println("我是绿色");
    	}
    }
    BLACK(0,0,0);
    private int redValue;
    private int greenValue;
    private int blueValue;
    public int getRedValue() {
        return redValue;
    }
    public int getGreenValue() {
        return greenValue;
    }
    public int getBlueValue() {
        return blueValue;
    }
    private Color4(int redValue, int greenValue, int blueValue) {
        this.redValue = redValue;
        this.greenValue = greenValue;
        this.blueValue = blueValue;
    }
	@Override
	public void display() {
		System.out.println("我是一个颜色：三原色："+redValue+","+greenValue+","+blueValue);
	}
	@Override
	public void fly() {
		// TODO Auto-generated method stub
	}
}
```


2.enum类不能再继承其他类，因为已经隐式的直接继承了Enum类

## 注解

### 注解的理解

定义：用于修饰java中的数据（属性、方法、构造器、类等），相当于程序中的补充文字。不改变程序的逻辑，但可以被编译器或运行时解析，并做相应处理

### 内置的三种基本注解【掌握】

#### @Override

**源码**

```java
@Target(ElementType.METHOD)  //只能用于修饰方法
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```


只能用于修饰方法，检测被修饰的方法是否为有效的重写，如果不是，则报编译错误！ 

#### @Deparecated

用于表明被修饰的数据已经过时，不建议使用，为了新老版本的兼容，没有贸然的废弃，只是提醒！

可以用于修饰：类或接口、属性、方法、构造、局部变量、包、参数

#### @Suppresswarnings

**源码**

```java
//用于修饰类或接口、属性、方法、构造、局部变量、参数
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```


用于抑制程序中的编译警告

### 自定义注解【了解】

1. 定义注解的关键字@interface

2. 注解类体中的成员是：
	参数类型 方法名();

**注意：**
>①参数类型只能是 八大基本类型、String、枚举类、Class类型或上述类型的数组类型
>
>②方法名遵循标识符的命名规则和规范，但建议使用value。
>    因为使用时，可以省略方法名
>
>③可以在定义方法时，指定默认值，语法：
>    参数类型 方法名() default 默认值;

**例：**

```java
public class TestAnn2 {
    @MyAnn1("yy") //若方法名为value，则这么写
//	@MyAnn1(name = "yy") //若方法名为name，则这么写
//	@MyAnn1(name = "xx", value = "yy") //若方法有两个，则都要写上
    public static void main(String[] args) {
    }
}
enum Gender{
}
@Retention(RetentionPolicy.SOURCE)
@interface MyAnn1{
    String value();
//	String name() default "xx"; //这里加了默认值，则在使用的时候就可不用传值
//	Gender name(); //没错，参数类型可以为枚举类型
//	Gender[] name(); //参数类型可为数组
//	Person name(); //报错，参数类型不能是Person（自己的定义的）
}
class Person{
}
```

3. 使用注解

在被修饰的数据上方，添加注解即可，语法：
    @注解类型(方法名=值)
若参数类型为数组，则用{}，包起来，如：Suppresswarnings
    @Suppresswarnings(value = {"...","...","..."})

### 元注解

Retention:用于指明被修饰的注解可以保留多长
    RententionPolicy:SOURCE（源码） CLASS（编译器-默认） RUNTIME（运行时）

Target：用于指明被修饰的注解可以用于修饰哪些数据
    ElementType:TYPE LOCAL_VARIABLE FIELD METHOD等

Documented：能否在生成的帮助文档中显示

Inherited：注解是否具备继承性