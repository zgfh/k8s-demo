## yaml 模版
本目录列举了常用的yaml写法，你可直接复制根据注释修改即可

### 如何查询最新的写法，支持的参数
1. 如果本目录没有列出，您可以通过k8s api 查看所有参数，api 的参数就是yaml中的参数 [k8s api 文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.12/#-strong-api-overview-strong-)

2. 通过`kubectl explain 资源`，例如: 查看pod的spec支持的参数`kubectl explain pod.spec`

