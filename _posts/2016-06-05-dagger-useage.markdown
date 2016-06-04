---
layout: post
title:  "Dagger2 依赖注入解析与使用"
date:   2016-06-05 00:29:40 +0800
categories: javascript
---

## 依赖注入

首先，**依赖注入**（ Dependency Injection，简称DI ），是实现 **控制反转** 的一种方式。那控制反转又是什么？

控制反转，是面向对象编程的一种设计原则，用来降低代码之间的耦合度。面向对象的应用程序通过类之间的彼此合作来实现业务逻辑，所以类之间会产生引用，如果这些引用在类内部实例化，就会导致代码的耦合，不利于之后的扩展和维护。

依赖注入是实现控制反转的一种方式，通过 **构造函数** 或者 **方法** 将依赖对象传入当前对象。

```	java
// 没有使用依赖注入
class A {
	private B b;

	public A() {
		b = new B();
	}
}

// 使用依赖注入
class A {
	private B b;

	public A(B b) {
		this.b = b
	}
}

```

## Dagger

Dagger 是一个针对 Java 语言的依赖注入框架

`Dagger is a fully static, compile-time dependency injection framework for both Java and Android. It is an adaptation of an earlier version created by Square and now maintained by Google.`

如何使用 Dagger 来实现依赖注入？先看一下 Dagger 提供的 Api ，

```java
public @interface Component {
    Class<?>[] modules() default {};
    Class<?>[] dependencies() default {};
}

public @interface Subcomponent {
    Class<?>[] modules() default {};
}

public @interface Module {
    Class<?>[] includes() default {};
}

public @interface Provides {
}

public @interface MapKey {
    boolean unwrapValue() default true;
}

public interface Lazy<T> {
    T get();
}
```

上面是 Dagger 定义的注解类型，Dagger 中还使用了一些 JSR-330 定义的标准注解，

```java
public @interface Inject {
}

public @interface Scope {
}

public @interface Qualifier {
}
```

下面详细说明一下这些注解的含义和用法

- @Inject

	Inject 用来告诉 Dagger 哪些依赖需要它来提供，主要有三种使用方式。

	1、构造函数
	
		class Thermosiphon implements Pump {
			private final Heater heater;
	
		  	@Inject
		  	Thermosiphon(Heater heater) {
		   		this.heater = heater;
		  	}
	
		  	...
		}
	
	
	需要注意的是如果一个类有多个构造函数，我们只能在其中一个构造函数上使用 @Inject 。

	2、成员变量
	
		class CoffeeMaker {
		  	@Inject Heater heater;
		  	@Inject Pump pump;
	
		  	...
		}

	这里的成员变量不能是 private 或者 protected，因为 Dagger 需要能够访问这些成员来为它们赋值。

	3、方法
	
		class CoffeeMaker {
	
			@Inject Heater heater;
	
			@Inject
			public void heat() {
				heater.register(this);
			}
	
		}
	
	
	只有一种情况下使用方法注入，就是需要传递当前对象的时候，因为 @Inject 标识的方法会在类构造函数调用完成之后被立刻调用。

	如果一个类用 @Inject 注解了构造方法，这个类会被包含在 Dagger 的类图中（有无参构造函数的类也会默认包含进来），也就表示这个类的实例对象可以被 Dagger 生成，生成对象需要调用类的构造函数，如果构造函数没有参数，那直接 new 这个对象就好了；但是构造函数有参数的情况，就需要 Dagger 先去生成参数对象，如果一个参数对象是 Dagger 无法自主生成的呢？比如：

	- 接口
	- 没有用 @Inject 标识构造函数的类，第三方类库

	这就需要我们明确提供这些类的构造过程，Dagger 中 @Provides 注解就是来标识这个用的。

- @Provides

		@Provides Heater provideHeater() {
		  	return new ElectricHeater();
		}
	
		// 这里的参数对象由 Dagger 提供
		@Provides Pump providePump(Thermosiphon pump) {
	        return pump;
	   }
	
		// 这里的 activity 从外面传入，provide方法只负责转发给 Dagger
		Activity activity;
		@Provides Activity provideActivity() {
		  	return activity;
		}

	
	
	有了 @Provides，这些方法放在什么地方呢，Dagger 提供了另一个注解 @Module

- @Module
	
		@Module
		class DripCoffeeModule {
		  	@Provides Heater provideHeater() {
		    	return new ElectricHeater();
		  	}
		  	@Provides Pump providePump(Thermosiphon pump) {
		    	return pump;
		  	}
		}
	
	@Module 用来注解一个类，表示 Dagger 的一个模块，用来提供一些依赖类的实例。

	到这里通过 @Inject、@Provides、@Module，所有需要提供依赖注入的类 Dagger 都可以生成了，那最后如何生成呢，或者控制生成的入口在哪里？这个入口就是 @Component。

- @Component

		@Component(modules = DripCoffeeModule.class)
	interface CoffeeShop {
	
			void inject(MainActivity activity);
	
		  	CoffeeMaker maker();
		}
	
	
	@Component 用来注解一个接口，同时声明需要依赖的 Module，接口中的方法包括两类，一种为特定的类提供依赖注入，如上面的 `inject` 方法，主要用于一些框架类中的依赖注入，像 Android 中 Activity 成员变量的注入；另一种用于向外界提供需要的类对象。


	编译之后，Dagger 会为 CoffeeShop 生成一个 DaggerCoffeeShop 对象，然后

		CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
		    .dripCoffeeModule(new DripCoffeeModule())
		    .build();
	
		CoffeeMaker maker ＝ coffeeShop.maker();
	
	
		// Activity 中使用
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
	
			CoffeeShop coffeeShop = DaggerCoffeeShop.builder()
			    .dripCoffeeModule(new DripCoffeeModule())
			    .build();
	
			coffeeShop.inject(this);
		}
	
	
	Component 之间也可以互相依赖，达到扩展 Component 的目的。有两种方式 dependencies 和 Subcomponent，二者的区别是 dependencies 侧重在重用 Component，比如 ClassB 依赖 ClassA，ClassA 已经有 ModuleA 提供，同时在 ComponentA 中提供了获取 ClassA 的接口，那么ComponentB 就可以直接使用 ComponentA 来提供 ClassA；而 Subcomponent 侧重在重用 module，如果ComponentA 不想把 ClassA 暴露出来，只是在 ModuleA 中给内部其他类提供依赖，这时就需要使用 Subcomponent 的方式来让 ComponentB 获取到 ClassA 的。通过 Subcomponent 注解，ComponentB 可以访问到 ComponentA 内部类图提供的所有类。

		// dependencies 方式
		public class ComponentDependency {
		    @Component(modules = ModuleA.class)
		    public interface ComponentA {
		        SomeClassA1 someClassA1();
		    }
	
		    @Component(modules = ModuleB.class, dependencies = ComponentA.class)
		    public interface ComponentB {
		        SomeClassB1 someClassB1();
		    }
	
		    public static void main(String[] args) {
		        ModuleA moduleA = new ModuleA();
		        ComponentA componentA = DaggerComponentDependency_ComponentA.builder()
		                .moduleA(moduleA)
		                .build();
	
		        ModuleB moduleB = new ModuleB();
		        ComponentB componentB = DaggerComponentDependency_ComponentB.builder()
		                .moduleB(moduleB)
		                .componentA(componentA)
		                .build();
		    }
		}
	
	
		// Subcomponent 方式
		public class SubComponent {
		    @Component(modules = {ModuleA.class, ModuleB.class})
		    public interface ComponentA {
		        ComponentB componentB(ModuleB moduleB);
		    }
	
		    @Subcomponent(modules = ModuleB.class)
		    public interface ComponentB {
		        SomeClassB1 someClassB1();
		    }
	
		    public static void main(String[] args) {
		        ModuleA moduleA = new ModuleA();
		        ModuleB moduleB = new ModuleB();
		        ComponentA componentA = DaggerSubComponent_ComponentA.builder()
		                .moduleA(moduleA)
		                .moduleB(moduleB)
		                .build();
	
		        ComponentB componentB = componentA.componentB(moduleB);
		    }
		}
	
	
- @Qualifier

	有时候在一个 Module 里需要提供有相同接口的不同类实例，比如 Context，需要提供 Activity 类型的和 Application 类型的，但方法的返回值都是 Context，怎么告诉 Dagger 这是两种不同的子类型呢？就是用 @Qualifier 来标识不同的类实例。

		@Qualifier
		@Documented
		@Retention(RUNTIME)
		public @interface Named {
		  	String value() default "";
		}
	
		@Provides @Named("hot plate") Heater provideHotPlateHeater() {
		  	return new ElectricHeater(70);
		}
		@Provides @Named("water") Heater provideWaterHeater() {
		  	return new ElectricHeater(93);
		}
	
		class ExpensiveCoffeeMaker {
		  	@Inject @Named("water") Heater waterHeater;
		  	@Inject @Named("hot plate") Heater hotPlateHeater;
		  	...
		}
	

- @Scope

	@Scope 注解用于提供组建内的单例模式，被 @Scrope 标识的 Component，其 依赖的 Module 里 Provide 的对象都必须有相应的 Scope 或者没有 Scope，被 Scope 标识的 Provide 方法产生的对象，在相应的 Component 生命周期内只会产生一个实例。@Singleton 是 Dagger 默认提供的一个 Scope，我们也可以定义自己的 Scope。

		@Scope
		public @interface ActivityScope {
		}
	
	
		@ActivityScope
		@Component(      
		    modules = SplashActivityModule.class,
		    dependencies = AppComponent.class
		)
		public interface SplashActivityComponent {
		    SplashActivity inject(SplashActivity splashActivity);
	
		    SplashActivityPresenter presenter();
		}
	
- 其他

	- Lazy 提供晚加载，只有在第一次使用的时候才会去加载
	- Provider 每次调用均创建新的实例

	
  ```java
	@Inject Lazy<Message> message;
	message.get().messageMethod();
		
	@Inject Provider<Filter> filterProvider;
  ```
	

## Reference
[https://zh.wikipedia.org/wiki/%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC](https://zh.wikipedia.org/wiki/%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC)

[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0519/2892.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0519/2892.html)

[http://leoray.leanote.com/post/dagger2](http://leoray.leanote.com/post/dagger2)

[http://stackoverflow.com/questions/29587130/dagger-2-subcomponents-vs-component-dependencies](http://stackoverflow.com/questions/29587130/dagger-2-subcomponents-vs-component-dependencies)

[http://frogermcs.github.io/dependency-injection-with-dagger-2-the-api/](http://frogermcs.github.io/dependency-injection-with-dagger-2-the-api/)

[http://google.github.io/dagger/](http://google.github.io/dagger/)
