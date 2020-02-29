### 面试题2： 实现Singleton模式
**题目：设计一个类，我们只能生成该类的一个实例。**

**思路：单例模式，即构造方法私有化，不允许外部创建实例,且实例为类的静态成员。**

```
	/**
	*	饿汉
	*	缺点：直接创建实例，长时间不用的话浪费内存
	*/
	public class Singleton{

		private static Singleton singleton = new Singleton();

		private Singleton(){
		}

		public static Singleton getInstance(){
			return Singleton.singleton;
		}
	}
```

```
	/**
	*	饱汉
	*	缺点：多线程可能会重复创建
	*/
	public class Singleton{

		private static Singleton singleton;

		private Singleton(){
		}

		public Singleton getInstance(){
			if(singleton == null){
				singleton = new Singleton();
			}
			return Singleton.singleton;
		}
	}
```

```
	/**
	*	饱汉改进
	*	缺点：多线程抢锁浪费时间
	*/
	public class Singleton{

		private static Singleton singleton;

		private Singleton(){
		}

		public synchronized Singleton getInstance(){
			if(singleton == null){
				singleton = new Singleton();
			}
			return Singleton.singleton;
		}
	}
```

```
	/**
	*	Double Check
	*	缺点：会出现错误，一个线程开辟了空间但是属性没有完全初始化就被另一个线程拿去用了
	*/
	public class Singleton{

		private static Singleton singleton;

		private Singleton(){
		}

		public Singleton getInstance(){
			if(singleton == null){
				synchronized(Singleton.class){
					if(singleton == null){
						singleton = new Singleton();
					}
				}
			}
			return Singleton.singleton;
		}
	}
```

```
	/**
	*	改进版Double Check
	*/
	public class Singleton{

		private static volatile Singleton singleton;

		private Singleton(){
		}

		public Singleton getInstance(){
			if(singleton == null){
				synchronized(Singleton.class){
					if(singleton == null){
						singleton = new Singleton();
					}
				}
			}
			return Singleton.singleton;
		}
	}
```

```
	/**
	*	优雅版
	*	优点：通过static关键字线程友好以及JVM类加载的特点，成功完成ace！
	*	首先，内部类InstanceHolder只会在被调用的时候完成初始化，因此只有在外部调用getInstance的时候才会完成内部类的初始化，对单例进行创建。
	*	其次，static关键字修饰的instance只会被创建一次（static关键字告诉了JVM只会被创建一次），因此线程友好
	*/
	public class Singleton{

		private Singleton(){
		}

		private static class InstanceHolder{
			private final static Singleton instance = new Sinleton();
		}

		public static Singleton getInstance(){
			return InstanceHolder.instance;
		}
	}
```

```
	/**
	*	不够那么优雅版
	*/
	public class Singleton{

		private Singleton(){
		}

		private enum Singleton{
			INSTANCE;
			
			private final Singleton singleton;

			private Singleton(){
				instance = new Singleton();
			}

			private Singleton getInstance(){
				return instance;
			}

		}

		public static Singleton getInstance(){
			return Singleton.INSTANCE.getInstance;
		}
	}
```