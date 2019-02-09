通知接口

```java
public interface Advisor {
	Object invoke(Invocation inv);
}
```

环绕通知

```java
public class AroundAdvisor implements Advisor {
	private String name;

	public AroundAdvisor(String name) {
		this.name = name;
	}

	@Override
	public Object invoke(Invocation inv) {
		System.out.println("环绕通知" + name + "-前");
		Object result = inv.proceed();
		System.out.println("环绕通知" + name + "-后,获得结果：" + result);
		return result;
	}

}
```

前置通知

```java
public class BeforeAdvisor implements Advisor {
	private String name;

	public BeforeAdvisor(String name) {
		this.name = name;
	}

	@Override
	public Object invoke(Invocation inv) {
		System.out.println("前置通知" + name);
		return inv.proceed();
	}
}
```

后置通知

```java
public class AfterAdvisor implements Advisor {
	private String name;

	public AfterAdvisor(String name) {
		this.name = name;
	}

	@Override
	public Object invoke(Invocation inv) {
		try {
			return inv.proceed();
		} finally {
			System.out.println("后置通知" + name);
		}
	}
}
```

返回通知

```java
public class AfterReturningAdvisor implements Advisor {
	private String name;

	public AfterReturningAdvisor(String name) {
		this.name = name;
	}

	@Override
	public Object invoke(Invocation inv) {
		Object result = inv.proceed();
		System.out.println("返回通知" + name + ",获得结果：" + result);
		return result;
	}

}
```

异常通知

```java
public class AfterThrowingAdvisor implements Advisor {
	private String name;

	public AfterThrowingAdvisor(String name) {
		this.name = name;
	}

	@Override
	public Object invoke(Invocation inv) {
		try {
			return inv.proceed();
		} catch (Exception e) {
			System.out.println("异常通知" + name);
			throw new RuntimeException(e);
		}
	}
}
```

Invocation

```java
import java.lang.reflect.Method;
import java.util.List;

public class Invocation {

	private Object target;
	private Method method;
	private Object[] args;
	private List<Advisor> advisorList;
	private int index;

	public Object proceed() {
		if (index >= advisorList.size()) {
			try {
				return method.invoke(target, args);
			} catch (Exception e) {
				throw new RuntimeException(e);
			}
		}
		Advisor advisor = advisorList.get(index++);
		return advisor.invoke(this);
	}

	public void setTarget(Object target) {
		this.target = target;
	}

	public void setMethod(Method method) {
		this.method = method;
	}

	public void setArgs(Object[] args) {
		this.args = args;
	}

	public void setAdvisorList(List<Advisor> advisorList) {
		this.advisorList = advisorList;
	}

	public void setIndex(int index) {
		this.index = index;
	}
}

```

service类

```java
public class AopService {

	public int doService() {
		System.out.println("执行目标方法：AopService.doService()");
		int i = 100;
		return i;
	}
}
```

Main类

```java
public class AopMain {
	public static void main(String[] args) throws Exception {
		AopService myService = new AopService();
		Method method = AopService.class.getMethod("doService");

		Invocation inv = new Invocation();
		inv.setTarget(myService);
		inv.setMethod(method);
		List<Advisor> advisorList = new ArrayList<>();
		inv.setAdvisorList(advisorList);

		advisorList.add(new AfterReturningAdvisor("1"));
		advisorList.add(new AfterReturningAdvisor("2"));
		advisorList.add(new AfterThrowingAdvisor("1"));
		advisorList.add(new AfterThrowingAdvisor("2"));
		advisorList.add(new AfterAdvisor("1"));
		advisorList.add(new AfterAdvisor("2"));
		advisorList.add(new AroundAdvisor("1"));
		advisorList.add(new AroundAdvisor("2"));
		advisorList.add(new BeforeAdvisor("1"));
		advisorList.add(new BeforeAdvisor("2"));

		Object proceed = inv.proceed();

		System.out.println("result： " + proceed);

	}
}
```

