### 精通 Spring 源码 | InstantiationAwareBeanPostProcessor(1)

#### 一、简介
InstantiationAwareBeanPostProcessor 是 Spring 的一个扩展点，他是 BeanPostProcessor 的子类，扩展了 BeanPostProcessor ，而外提供了 3 个方法：

1、Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName)
2、boolean postProcessAfterInstantiation(Object bean, String beanName)
3、PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName)

#### 二、源码分析

本篇我们来探究 postProcessBeforeInstantiation 这个方法的作用，这个方法会返回一个 Object 类型的对象，在源码里面，如果有一个类实现了 InstantiationAwareBeanPostProcessor 这个后置处理器，在 postProcessBeforeInstantiation 方法里面返回了一个对象，那么直接返回，后续只会实现 BeanPostProcessor 的 postProcessAfterInitialization 方法，否则，按照正常流程走。我们可以看看源码：

```
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			//如果bean不为空直接返回
			if (bean != null) {
				return bean;
			}
```

```
	@Nullable
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					//1、调用 InstantiationAwareBeanPostProcessor 的 postProcessBeforeInstantiation 方法
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						//2、调用 BeanPostProcessor 的 postProcessAfterInitialization 方法
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
```
下面的方法里面实现的是 postProcessBeforeInstantiation 这个方法
```
bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
```

```
	@Nullable
	protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
		//拿到实现了BeanPostProcessors的接口
		for (BeanPostProcessor bp : getBeanPostProcessors()) {
			if (bp instanceof InstantiationAwareBeanPostProcessor) {
				InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
				Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
				if (result != null) {
					return result;
				}
			}
		}
		return null;
	}
```

默认情况上面的方法返回的是空,如果不为空, 调用 BeanPostProcessor 的 postProcessAfterInitialization 方法 

```
if (bean != null) {
//2、调用 BeanPostProcessor 的 postProcessAfterInitialization 方法
   bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
}
```

之后便是直接返回得到 Bean

#### 三、示例

源码比较复杂，可能文字无法表达清楚，下面通过具体代码演示结果

```
@ComponentScan("com.javahly.spring59")
@Configuration
public class Appconfig {
}
```

```
@Component
public class IndexService {

}
```

```
@Component
public class OrderService {

}
```

```
@Component
public class UserService {

}
```

```
@Component
public class MyInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

	@Override
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		if (beanName.endsWith("indexService"))
			return new UserService();
		return null;
	}
}
```

```
public class Test {

	public static void main(String[] args){
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
		applicationContext.register(Appconfig.class);
		applicationContext.refresh();
		System.out.println("orderService："+applicationContext.getBean("orderService"));
		System.out.println("userService："+applicationContext.getBean("userService"));
		System.out.println("indexService："+applicationContext.getBean("indexService"));
		System.out.println("indexService："+applicationContext.getBean("indexService"));
	}
}
```

这里我们把三个对象使用 @Component 标注，依旧是说我们想把这三个对象注入容器，不出意外，我们在运行 main 方法的时候会打印出这三个对象，但是，我们自定义了一个 MyInstantiationAwareBeanPostProcessor 扩展了InstantiationAwareBeanPostProcessor，住了一个偷梁换柱，把原本需要放入容器的 indexService 换成了 userService，打印结果如下：

orderService：com.javahly.spring59.service.OrderService@d7b1517
userService：com.javahly.spring59.service.UserService@16c0663d
indexService：com.javahly.spring59.service.UserService@23223dd8
indexService：com.javahly.spring59.service.UserService@23223dd8

—— 完


#### ABOUT


>我的 Github：[Github](https://github.com/huangliangyun)
>CSDN: [CSDN](https://blog.csdn.net/Sirius_hly)
>个人网站: [sirius blog](http://www.javahly.com/)

>推荐阅读
>[史上最全，最完美的 JAVA 技术体系思维导图总结，没有之一!](https://blog.csdn.net/Sirius_hly/article/details/94335233)