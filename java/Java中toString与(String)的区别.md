## 1. toString()方法  
API 中关于 toString()的描述：  
返回该对象的字符串表示。通常， toString 方法会返回一个“以文本方式表示”此对象的字符串。结果应是一个简明但易于读懂的信息表达式。建议所有子类都重写此方法。  
Object 类的 toString 方法返回一个字符串，该字符串由类名（对象是该类的一个实例）、at 标记符“ @ ”和此对象哈希码的无符号十六进制表示组成。换句话说，该方法返回一个字符串，它的值等于:  
```java
getClass().getName() + '@' + Integer.toHexString(hashCode())
```
toString()方法返回的是这个对象的字符串表示，就像是这个对象的名字一样，任何对象都可以有自己的名字，你可以重写其toString()方法，给其赋予任意的名字。

但是调用toString()方法的对象不能为 null，否则会抛出异常：java.lang.NullPointerException。
  
## 2. String.valueOf()方法
调用toString()方法的对象不能是null，但 String.valueOf()方法不管这些，但这个方法也是调用了toString()方法，只不过在调用之前做了点处理，在源码中：
```java
/**
     * Returns the string representation of the <code>Object</code> argument.
     *
     * @param   obj   an <code>Object</code>.
     * @return  if the argument is <code>null</code>, then a string equal to
     *          <code>"null"</code>; otherwise, the value of
     *          <code>obj.toString()</code> is returned.
     * @see     java.lang.Object#toString()
     */
    public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
    }
```
这个方法在调用 **toString()** 之前判断一下这个对象是不是null，如果不是null，则正常调用其toString()方法，如果是null 的话，则返回 字符串 形式的null。

**String.valueOf()** 比起直接用 **toString()** 来说虽然可能会减少报错的机会，但是如果在对比对象值的时要小心如果用 **if(String.valueOf（object）==null)** 肯定是不行的了。

## 3. 强制转换 (String)data
每个对象的类型在对象创建的时候已经确定并且不能更改，所谓强制转换也只是使其表面上换成了另一种类型，可以使用其方法对这个对象进行处理。那么可想而知，把物品A 当成物品B 来使用，当A 能能够被当成 B的时候大家都相安无事，你走你的路，我过我的桥，一旦A 不能被当成B ，它不会去自动调用 toString()方法，而是马上就会报错。  
  
例一：
```java
Integer obj1 = new Integer(100);
String strVal = (String)obj1;　　//Cannot cast from Integer to String
```
因为obj1 在创建的时候就是 Integer 类型，不能转换成 String 类型，所以在编译期间就会报错 **Cannot cast from Integer to String**。
  
例二：
```java
Object obj2 = new Integer(100);
String strVal = (String)obj2;
```
obj2 虽然本质上是 Integer 类型，但其表面上确是 Object 类型，所以在编译的时候没有报错，但因为 obj2 在创建的时候已经确定了其在本质上 Integer 类型，所以这两行代码在运行时依然会报错，因为 Integer 型不能转换成 String 类型。

当然，如果要把 Integer 型转换成 String，可以调用其 toString()方法：Integer.toString(obj1) 或者 String.valueOf(obj1); 对应于其他自定义类型，则调用自己重写的 toString() 方法。

此外，因null值可以被强制转换为任何类型，所以(String)null也是合法的。
