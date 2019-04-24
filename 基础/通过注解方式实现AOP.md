# 通过注解方式实现IOC和AOP

> ​       IOC和AOP是spring的两个特点，通常情况都是通过配置文件将需要依赖的属性和AOP进行配置，然后Spring框架读取配置文件进一步解析。但是具体的如何注入以及AOP是如何运作的，通过下面的例子我们就能很快了解。

## 实现IOC

> ​       IOC实现由框架自动生成对象，然后进行缓存，不需要传统方式new一个对象出来。下面例子简单说明了框架是如何自动创建一个对象的。

### 非单例模式

- 创建一个注解，用来表示自动生成一个对象并且注入到对象中。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)//只能注解在属性上
public @interface SimpleInject {}
```

- 创建两个类，其中一个类是另一个类的属性成员

```java
public class ServiceA {
    @SimpleInject
    private ServiceB serviceB;
    public void sayHello() {System.out.println("hello I am serviceA");}
    public void callB() {serviceB.sayHello();}
}
```

```java
public class ServiceB {
    public String name = "B";
    public void sayHello()
    {
        System.out.println("hello I am serviceB");
    }
}
```

- 只有注解还不能生效，创建一个DI容器，框架通过这个容器实现自动生成对象和依赖注入并进行缓存。

```java
public class SimpleInjectContainer {
    public static <T> T getInstance(Class<T> cls) {
        try {
            T obj = cls.newInstance();
            Field[] fields = obj.getClass().getDeclaredFields();
            for (Field field : fields) {
                if (field.isAnnotationPresent(SimpleInject.class)) {
                    field.setAccessible(true);
                    field.set(obj, getInstance(field.getType()));
                }
            }
            return obj;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

​      上述代码中getInstance方法中传入一个Class参数，首先根据这个Class生成一个对象obj。随后遍历这个类中声明的所有成员对象，field.isAnnotationPresent(clz)用来判断当前Field上是否有clz类型的注解，如果有则进行下一步。找到有SimpleInject注解的属性之后，将先前生成的对象obj对应的属性设置为属性类型的对象，field.getType()获取当前属性的类型。因为有可能在这个属性中有依赖其他的类，所以需要通过递归的方式进行创建依赖的对象。最后将处理好的obj返回给框架。

- 创建一个测试类对上面的代码进行测试

```java
public class TestInject {
    public static void main(String[] args)
    {
        ServiceA serviceA = SimpleInjectContainer.getInstance(ServiceA.class);
        serviceA.callB();
    }
}
```

输出的结果如下：

```te
hello I am serviceB
```

注入成功。

> ​      通过上面的SimpleInject注解方式如果多个地方需要使用ServiceA，则每调用一次就会创建一个新的对象出来，下面就新添加一个注解@SimpleSingleton来实现单例模式。

### 单例模式

- 创建一个单例的注解@SimpleSingleton，这个注解需要使用在类上。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface SimpleSingleton {}
```

- 在ServiceA上添加注解@SimpleSingleton，并在这个类中添加一个属性（String name = "A";）用来测试是否每次生成的都是同一个对象。

```java
@SimpleSingleton
public class ServiceA {
    @SimpleInject
    private ServiceB serviceB;
    public String name = "A";
    public void sayHello() {System.out.println("hello I am serviceA");}
    public void callB() {serviceB.sayHello();}
}
```



- 在SimpleInjectContainer容器中新增生成单例的方法。由于是单例，所以需要将之前生成的对象缓存起来，以确保之后获取的对象是先前创建的。

```java
private static Map<Class<?>, Object> instances = new HashMap<>();
```

```java
public static <T> T getSingleton(Class<T> clz) {
  if (!clz.isAnnotationPresent(SimpleSingleton.class)) {
    return getInstance(clz);
  }
  Object obj = instances.get(clz);
    if (obj == null) {
      synchronized (clz) {
        obj = instances.get(clz);
        if (obj == null) {
          obj = getInstance(clz);
          instances.put(clz, obj);
        }
      }
    }
    obj = instances.get(clz);
    return (T) obj;
}
```

​      首先判断是否在类上添加了单例注解，如果没有添加，则按照正常的逻辑创建一个新的对象返回。getInstance(Class<T> clz)方法是之前@SimpleInject注解创建对象的逻辑，在此可以通用。

​      如果类上添加了单例的注解，则需要判断之前是否有创建过这个类的对象，从缓存instance这个Map中查找，判断是否为空。因为可能同时有多个线程同时访问，此处使用双重检查锁方式，确保并发情况下能够正确的创建单例。

​      最后将创建的单例进行缓存，确保下一次获取的对象为此次创建的对象。

- 验证：

```java
//单例
ServiceA a = SimpleInjectContainer.getSingleton(ServiceA.class);
System.out.println(a.name);
a.name = "a++";
System.out.println(a.name);
ServiceA a1 = SimpleInjectContainer.getSingleton(ServiceA.class);
System.out.println(a1.name);
```

​      输出结果如下：

```java
A
a++
a++
```

​      新创建出来的ServiceA的对象name默认为A，创建ServiceA的对象后将name修改，然后重新通过单例创建，获取的对象name为修改后的值，说明两次获取的对象是同一个。

## 实现AOP

> 目标：在不改变原有代码的情况下，在执行代码前后，以及出现异常时输出相应的信息。
>
> 思路：在新增的逻辑中获取响应的需要修饰的类信息，通过CGLib代理原来的类，在执行原方法前后或者出现异常时执行插入的方法。

- 新增切面注解

```java
@Target(ElementType.TYPE)//注解的目标
@Retention(RetentionPolicy.RUNTIME)//表示注解保留到什么时候SOURCE;CLASS;RUNTIME默认CLASS
public @interface Aspect {
    Class<?>[] values();//作用的类
}
```

​      values数组包含了需要修饰的类，下面会用到。

- 添加包含before和after方法的类

```java
@Aspect(values = {ServiceA.class, ServiceB.class})
public class ServiceLogAspect {
    public static void before(Object obj, Method method, Object[] args) {
        System.out.println("entering " + method.getDeclaringClass().getSimpleName() + "::" + method.getName() + "," + "args:" + Arrays.toString(args));
    }
    public static void after(Object obj, Method method, Object[] args, Object result) {
        System.out.println("leaving " + method.getDeclaringClass().getSimpleName() + "::" + method.getName() + "," + "result:" + result);
    }
}
```

​      在ServiceLogAspect上添加注解@Aspect，并将values设置为ServiceA和ServiceB，在ServiceA和ServiceB中的方法执行顺序为：before()->serviceA或serviceB的方法->after()。

- 添加包含exception方法的类

```java
@Aspect(values = {ServiceA.class,ServiceB.class})
public class ServiceExceptionAspect {
    public static void exception(Object obj, Method method, Object[] args, Throwable e) {
        System.err.println("exception when calling: " + method.getName() + "," + Arrays.toString(args));
    }
}
```

​       在ServiceLogAspect上添加注解@Aspect，并将values设置为ServiceA和ServiceB，那么在serviceA和serviceB的方法抛出异常时会执行exception方法。

>  只有以上注解现在还不能实现自动执行切面逻辑的功能，我们需要创建一个容器，用来在容器中处理AOP逻辑。

### 新增AOP容器

> ​      如果每次执行方法前都去判断这个方法是否需要增加AOP逻辑，那么整个框架会变得很笨重，所以需要在项目启动的时候将需要执行的类进行缓存，这样就减缓了执行时的压力。

- 创建容器类，在容器类中添加初始化方法

```java
public class CGLibContainer {
    //获取被注解的类，正常情况下应该通过扫描，此处写死
    static Class<?>[] aspects = new Class<?>[]{ServiceLogAspect.class, ServiceExceptionAspect.class};
    //初始化
    static {
        init();
    }
    private static void init() {
        for (Class<?> clz : aspects) {
            Aspect aspect = clz.getAnnotation(Aspect.class);
            if (null == aspect) {
                continue;
            }
            //获取各个切面方法
            Method before = getMethod(clz, "before", new Class<?>[]{Object.class, Method.class, Object[].class});
            Method after = getMethod(clz, "after", new Class<?>[]{Object.class, Method.class, Object[].class, Object.class});
            Method exception = getMethod(clz, "exception", new Class<?>[]{Object.class, Method.class, Object[].class, Throwable.class});
            //获取注解的value，也就是需要修饰的类
            Class<?>[] clzs = aspect.values();
            for (Class<?> intercepted : clzs) {
                addInterceptedMethodMap(intercepted, AspectPoint.BEFORE, before);
                addInterceptedMethodMap(intercepted, AspectPoint.AFTER, after);
                addInterceptedMethodMap(intercepted, AspectPoint.EXCEPTION, exception);
            }
        }
    }
}
```

​      aspects对象中保存的是被@Aspect注解的类，正常情况下，这个数组中的对象应该是由扫包得到，为了简化代码，此处写死。

​      因为这个数组中的每个类都是被@Aspect注解修饰的,所以方法中的aspect是否为空的逻辑可以省略。

​     getMethod方法的逻辑如下：

```java
private static Method getMethod(Class<?> clz, String methodName, Class<?>[] parameterTypes) {
    try {
        return clz.getMethod(methodName, parameterTypes);
	} catch (NoSuchMethodException e) {
		return null;
	}
}
```

​      返回了对应的类中方法名是method，参数类型是parameterTypes的方法，这样就能得到每个切面方法。如果找不到则返回null。

​      init方法中的clzs获取的是当前切面方法中需要修饰的类，然后将这个类对应的切面方法缓存。在缓存时我们需要创建一个全局变量以及一个枚举用来对应每个不同的方法。

```java
//可能一个类会被多个切面修饰，使用list保存
static Map<Class<?>, Map<AspectPoint, List<Method>>> interceptMehondMap = new HashMap<>();
```

```java
public enum AspectPoint {
    BEFORE,AFTER,EXCEPTION
}
```

​      addInterceptedMethodMap方法的具体逻辑如下：

```java
private static void addInterceptedMethodMap(Class<?> clz, AspectPoint point, Method method) {
	if (method == null) {
		return;
	}
	if (!interceptMehondMap.containsKey(clz)) {
		interceptMehondMap.put(clz, new HashMap<>());
	}
	Map<AspectPoint, List<Method>> aspectPointListMap = interceptMehondMap.get(clz);
	if (!aspectPointListMap.containsKey(point)) {
		aspectPointListMap.put(point, new ArrayList<>());
	}
	aspectPointListMap.get(point).add(method);
}
```

​      添加缓存的方法比较简单，不做过对讲解。

​     添加完成缓存之后就要对这个缓存加以利用，如何使用这个缓存，在原有的类方法上执行切面方法？此处就需要用到之前介绍的CGLib动态代理。

—— **[动态代理--JDK、CGLib](http://mp.weixin.qq.com/s?__biz=MzI4NjM0NzYzNw==&mid=2247483768&idx=1&sn=889a7cffdd007f7ad39dce998bda0445&chksm=ebdf1962dca89074a3937f7703ea5395e5582fbf5deac375d9eec96742dddcea498e678d4987&scene=21#wechat_redirect)**

​      添加一个内部类AspectInterceptor继承MethodInterceptor

```java
static class AspectInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        //执行before方法
        List<Method> befores = getInterceptMethod(AspectPoint.BEFORE, o.getClass().getSuperclass());
        for (Method m : befores) {
            //方法为静态的时候obj可以被忽略，用null
            m.invoke(null, new Object[]{o, method, objects});
        }
        //执行代理方法
        Object result = null;
        try {
            result = methodProxy.invokeSuper(o, objects);
            //执行after方法
            List<Method> afters = getInterceptMethod(AspectPoint.AFTER, o.getClass().getSuperclass());
            for (Method m : afters) {
                m.invoke(null, new Object[]{o, method, objects, result});
            }
        } catch (Exception e) {
            //执行exception方法
            List<Method> exceptions = getInterceptMethod(AspectPoint.EXCEPTION, o.getClass().getSuperclass());
            for (Method m : exceptions) {
                m.invoke(null, new Object[]{o, method, objects, e});
            }
        }
        return result;
    }

    private static List<Method> getInterceptMethod(AspectPoint point, Class<?> clz) {
        if (!interceptMehondMap.containsKey(clz)) {
            return Collections.emptyList();
        }
        Map<AspectPoint, List<Method>> map = interceptMehondMap.get(clz);
        if (map.containsKey(point)) {
            return map.get(point);
        }
        return Collections.emptyList();
    }
}
```

​      在代理对象执行intercept方法时，分别在methodProxy.invokeSuper(o, objects)前后以及抛出异常时执行相应的方法。执行的方法直接从之前的缓存中获取。

​      然后在添加创建代理对象方法，使用代理类执行原有方法：

```java
//获取对象
public static <T> T getInstance(Class<T> clz) {
	try {
		T obj = createInstance(clz);
		//处理依赖
		Field[] fields = obj.getClass().getSuperclass().getDeclaredFields();
		for (Field field : fields) {
			if (!field.isAnnotationPresent(SimpleInject.class)) {
				continue;
			}
			field.setAccessible(true);
			field.set(obj, getInstance(field.getType()));
		}
		return obj;
	} catch (Exception e) {
		throw new RuntimeException(e);
	}
}

private static <T> T createInstance(Class<T> clz) throws Exception {
	//如果没有被修饰，则直接返回，不需要代理
	if (!interceptMehondMap.containsKey(clz)) {
		return (T) clz.newInstance();
	}
	//获取CGLib代理
	Enhancer enhancer = new Enhancer();
	enhancer.setSuperclass(clz);
	enhancer.setCallback(new AspectInterceptor());
	return (T) enhancer.create();
}
```

​      外部调用getInstance方法获取代理对象，在这个方法中实现了上述的简单的IOC实现。在创建对象的时候使用createInstance方法，这个方法中有一个分支判断，如果之前初始化的缓存中存在当前Class，说明这个类被Aspect引用，需要加入切面方法，此时需要用过CGLib创建一个代理对象，如果没有，则直接返回当前Class的实例对象。

### 测试AOP

​      在测试之前需要在ServiceA中添加一个异常方法用来测试抛出异常时的情况。

```java
public void createException( int i)
{
	int a = 1/0;
}
```

​      添加测试类：

```java
public class TestCGLib {
    public static void main(String[] args) {
        ServiceA serviceA = CGLibContainer.getInstance(ServiceA.class);
        serviceA.callB();
        System.out.println("====================");
        serviceA.createException(1);
    }
}
```

​      执行结果如下：

```java
entering ServiceA::callB,args:[]
entering ServiceB::sayHello,args:[]
hello I am serviceB
leaving ServiceB::sayHello,result:null
leaving ServiceA::callB,result:null
====================
entering ServiceA::createException,args:[1]
exception when calling: createException,[1]
```

​      水命在执行serviceA以及serviceA的成员serviceB的方法时，前后都分别执行了before和after方法，并且在抛出异常时执行了exception方法。至此说明整个AOP执行成功。

PS.文章参考了马俊昌老师《老马说编程》一书。