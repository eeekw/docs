# Kubernetes

## Yaml

### 必须字段

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

* `apiVersion`指定用哪个版本的`Kubernetes API`来创建`Kubernetes`对象
* `kind`创建的对象类型
* `metadata`

