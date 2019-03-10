配置文件访问方式

EnvironmentController

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

