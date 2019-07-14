###  HystrixDashboard

HystrixDashboardController

http://localhost:8080/hystrix

http://localhost:8080/actuator/hystrix.stream

熔断满足的条件

1、一定时间内请求总数达到配置的数量
2、降级次数百分比达到配置百分比
3、半打开时间是我们自己配置的

限流、降级、熔断

定义fallback的方式
1、返回固定值
2、调用备用接口
3、查询缓存

HystrixCircuitBreakerConfiguration

HystrixCommandAspect
hystrix针对@HystrixCommand注解方法的切面

AbstractCommand
	HystrixCommand

HystrixCircuitBreaker
	HystrixCircuitBreakerImpl
	NoOpCircuitBreaker 返回固定值，不做其他操作

1、有缓存，直接返回
2、断路器是否为打开状态，如果是打开状态，直接调用fallback；如果非打开状态，继续向后执行；
3、

降级就是在执行主流程时，主流程突然出现意外执行不下去了，那就执行另外一个方法让主流程看起来是正常的（这个方法通常就是降级方法，似乎有些牵强)。

过了熔断器工作的时间窗以后，尝试放一个请求进去，执行成功就关闭熔断器，失败则继续打开，如此往复循环进行工作。
熔断器打开了，后续请求直接走降级方法fallback，不再调用主流程方法。 