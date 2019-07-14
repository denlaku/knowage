### 配置中心服务端

ConfigServerAutoConfiguration

ConfigServerMvcConfiguration
	EnvironmentController
	ResourceController

EncryptionAutoConfiguration
ConfigServerEncryptionConfiguration
	EncryptionController

配置文件访问方式

```
/{name}/{profiles:.*[^-].*}
/{name}/{profiles}/{label:.*}
/{name}-{profiles}.properties
/{label}/{name}-{profiles}.properties
/{name}-{profiles}.json
/{label}/{name}-{profiles}.json
/{name}-{profiles}.yml
/{name}-{profiles}.yaml
/{label}/{name}-{profiles}.yml
/{label}/{name}-{profiles}.yaml
```

支持native、git、svn



### 配置中心客户端

