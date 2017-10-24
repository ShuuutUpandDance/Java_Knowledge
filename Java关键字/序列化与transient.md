## 序列化与transient关键字
平时存在 JVM 内存中的对象，有着自己特有的数据结构，是无法进行 **IO 操作或网络通信**的，因为除了 JVM ，别人都不知道 JVM 中的那堆数据到底是个什么东西，因此必须把对象**以某种别人也能认识的方式**表示出来，**序列化**就是其中一种方式。

- 序列化：将一个对象转换成一串二进制表示的字节数组，通过保存或转移该字节数组来达到持久化对象的目的。
- 反序列化：将序列化得到的字节数组解析成对象。

### 实现序列化的方法
只需让类实现 java.io.Serializable 接口就可以了。序列化的时候会有一个 **serialVersionUID 常量**（private static final long），JAVA 序列化机制就是通过**在运行时**判断类的的 serialVersionUID 来验证**版本一致性**的。在反序列化时，JVM 会把字节流中的 serialVersionUID 与本地相应类的 serialVersionUID 进行比较，如果相同就认为是版本一致的类，就进行反序列化，否则会抛出异常。

serialVersionUID 有两种生成方式：
1. 默认为 **1L**；
2. 根据类名、接口名、成员方法以及属性等生成一个**64位的 Hash 字段**。

如果实现 java.io.Serializable 接口的实体类并没有显式定义一个 long 类型的名为 serialVersionUID 的变量，Java 的序列化机制就会默认根据编译后的 .class 字节码文件自动生成一个 serialVersionUID，只要编译得到的 .class 字节码文件的内容不变，不管编译多少次 serialVersionUID 都不变。也就是说，JAVA 提供了默认的序列化、反序列化方法，也就是 ObjectOutputStream 的 defaultWriteObject 方法和 ObjectInputStream 的 defaultReadObject 方法。

**注意：**
1. 对象序列化之后的字节数组保存了所属类的息以及对象的信息；
2. 类中被**关键字 transient 修饰**的属性**不会被序列化**。
3. 类中被**关键字 static 修饰**的属性**不会被序列化**。因为被**关键字 static 修饰**的静态变量是属于类的，不是属于实例对象的，而序列化针对的是对象，所以不会序列化静态变量。

### 自定义序列化过程
进行序列化、反序列化时，虚拟机会首先试图调用对象里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化。如果没有这样的方法，那么默认调用的是 ObjectOutputStream 的 defaultWriteObject 以及 ObjectInputStream 的 defaultReadObject 方法。

自定义序列化的**应用场景**：
1. 有些场景下，对于某些字段我们不想使用 JAVA 提供的默认序列化方式，比如 ArrayList 的 elementData、HashMap 的 table（原因在分析集合类的时候说明），就可以声明这些字段为 transient ，然后在 writeObject 和 readObject 方法中实现自己的序列化方法。
2. 序列化并不安全，有些场景下需要先对敏感字段加密再序列化，然后在反序列化的时候要先解密。

### 复杂情况总结：
1. 当父类实现 Serializable 接口时，所有子类都可被序列化；
2. 当子类实现 Serializable 接口但是父类没有时，父类中的属性不能序列化（不报错，只是没有数据），子类的属性仍能正常序列化；
3. 如果序列化的属性也是对象，则这个对象也必须实现 Serializable 接口，否则报错；
4. 反序列化时，如果对象的属性有修改，则修改的属性会丢失，但不会报错；
5. 反序列化时，如果 serialVersionUID 被修改，则反序列化失败。