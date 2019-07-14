LoadBalancerAutoConfiguration
	LoadBalancerInterceptorConfig
		RestTemplateCustomizer  对RestTemplate进行定制化配置

ClientHttpRequestInterceptor
	ClientHttpRequestExecution
	LoadBalancerInterceptor

LoadBalancerClient
	RibbonLoadBalancerClient

LoadBalancerRequestFactory -> LoadBalancerRequest

ILoadBalancer
	AbstractLoadBalancer
		NoOpLoadBalancer
			DynamicServerListLoadBalancer
				ZoneAwareLoadBalancer
		BaseLoadBalancer

EurekaRibbonClientConfiguration
	IPing -> NIWSDiscoveryPing
	ServerList  -> DomainExtractingServerList获取服务器列表
	ServerIntrospector

IRule
AbstractLoadBalancerRule



RibbonAutoConfiguration	

