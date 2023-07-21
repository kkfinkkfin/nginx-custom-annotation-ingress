# Custom Annotations

Custom annotations enable you to quickly extend the Ingress resource to support many advanced features of NGINX, such as rate limiting, caching, etc.

## Prerequisites 

* Read the [custom annotations doc](https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/custom-annotations/) before going through this example first.
* Read about [custom templates](../custom-templates).

## 1 - Rewrite With HostName

### [rewrite]
* `custom.nginx.org/rewrite-with-host: "https://www.baidu.com"` - configures the URL with hostname to rewrite

### Ingress Example
```ingress
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/rewrite-with-host: "https://www.baidu.com"
spec:
  # ingressClassName: nginx # use only with k8s version >= 1.18.0
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

## 2 - Proxy Set Header

### [proxy_set_header]
* `custom.nginx.org/set-header: "Connection close, X-Real-IP $remote_addr"`

### Ingress Example
```ingress
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/set-header: "Connection close, X-Real-IP $remote_addr"
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

## 3 - Response Add Header

### [add_header]
* `custom.nginx.org/add-header: "Cache-Control private, test 111"`

### Ingress Example
```ingress
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/add-header: "Cache-Control private, test 111"
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

## 4 - Limit Connection

### [limit_conn]
* `custom.nginx.org/conn-limiting: "on"`
* `custom.nginx.org/conn-limiting-num: "1000"`

### Ingress Example
```ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/rate-limiting: "on"
    custom.nginx.org/conn-limiting: "on"
    custom.nginx.org/conn-limiting-num: "1000"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /tea
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /coffee
```

## 5 - Limit Request

### [limit_req]
* `custom.nginx.org/rate-limiting: "on"`
* `custom.nginx.org/rate-limiting-rate: "500r/s"`
* `custom.nginx.org/rate-limiting-burst: "3"`

### Ingress Example
```ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/rate-limiting: "on"
    custom.nginx.org/rate-limiting-rate: "500r/s"
    custom.nginx.org/rate-limiting-burst: "3"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /tea
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /coffee
```

## 6 - Weight Canary

### [weight]
* `custom.nginx.org/canary-by-path: "/"`
* `custom.nginx.org/canary-by-weight: "70% /match0, 30% /match1"`
* `nginx.org/rewrites: nginx.org/rewrites: "serviceName=tea-svc rewrite=$request_uri;serviceName=coffee-svc rewrite=$request_uri"`

### Ingress Example
```ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/canary-by-path: "/"
    custom.nginx.org/canary-by-weight: "70% /match0, 30% /match1"
    nginx.org/rewrites: "serviceName=tea-svc rewrite=$request_uri;serviceName=coffee-svc rewrite=$request_uri"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /match1
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match0
```

## 7 - Condition Route

Condition Route（条件路由） 和 Weight Canary (按比例灰度) 不能同时使用。
条件路由支持有：Header, Cookie，Argument, Variable，IPSET（黑白名单）可以做1到多个条件随意组合，每个条件里面可以是1到多组值。支持逻辑与或非组合。

### 单个条件必须携带此参数标记
* `custom.nginx.org/canary-single: "true"`
* `custom.nginx.org/canary-by-multicondition: "1 /match1, 0 /match0, default /match0"`
`custom.nginx.org/canary-by-multicondition` 值根据实际的条件数量进行填写

### 1个以上的条件必须携带此参数标记
* `custom.nginx.org/canary-multi: "true"`
* `custom.nginx.org/canary-by-multicondition: "11 /match11, 10 /match10, 01 /match01, 00 /match00"`
`custom.nginx.org/canary-by-multicondition` 值根据实际的条件数量进行填写
1 - 表示满足条件
0 - 表示不满足条件


### [cookie]
* `custom.nginx.org/canary-by-path: "/"`
* `custom.nginx.org/canary-by-cookie: "version"`
* `custom.nginx.org/canary-by-cookie-value: "v1 /match0, default /match1"`
* `custom.nginx.org/canary-by-multicondition: "1 /match1, 0 /match0, default /match0"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=$request_uri;serviceName=coffee-svc rewrite=$request_uri"`

### Single Condition Ingress Example 
```ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "yunke"
    custom.nginx.org/canary-single: "true"
    custom.nginx.org/canary-by-path: "/"
    custom.nginx.org/canary-by-cookie: "version"
    custom.nginx.org/canary-by-cookie-value: "v1 1, default 0"
    custom.nginx.org/canary-by-multicondition: "1 /match1, 0 /match0, default /match0"
    nginx.org/rewrites: "serviceName=tea-svc rewrite=$request_uri;serviceName=coffee-svc rewrite=$request_uri"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /match1
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match0
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
```

### Multi Condition Ingress Example 
```ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    custom.nginx.org/canary-multi: "true"
    custom.nginx.org/canary-by-path: "/"
    custom.nginx.org/canary-by-cookie: "version"
    custom.nginx.org/canary-by-cookie-value: "v1 1, default 0"
    custom.nginx.org/canary-by-header: "header"
    custom.nginx.org/canary-by-header-value: "foo 1, default 0"
    custom.nginx.org/canary-by-multicondition: "11 /match11, 10 /match10, 01 /match01, 00 /match00"
    nginx.org/rewrites: "serviceName=tea-svc rewrite=$request_uri;serviceName=coffee-svc rewrite=$request_uri"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /match11
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match00
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match10
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match01
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
```

## 8 - Others Canary
Header, Argument, Variable Canary 同 Cookie Canary

### Header
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-header: "version"`
* `custom.nginx.org/canary-by-header-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea$request_uri;serviceName=tea-v2-svc rewrite=/tea$request_uri"`

### Argument
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-argument: "version"`
* `custom.nginx.org/canary-by-argument-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea$request_uri;serviceName=tea-v2-svc rewrite=/tea$request_uri"`

### Variable
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-variable: "version"`
* `custom.nginx.org/canary-by-variable-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea$request_uri;serviceName=tea-v2-svc rewrite=/tea$request_uri"`


### IP_SET
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-ipset: "172.16.240.0/24 1, default 0"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea$request_uri;serviceName=tea-v2-svc rewrite=/tea$request_uri"`


## 9 - Canary Add Header 灰度流量标记
* `custom.nginx.org/canary-add-header-svc: "tea-svc 1, coffee-svc 0"`
* `custom.nginx.org/canary-add-header: "Connection close, X-Real-IP $remote_addr"`
`custom.nginx.org/canary-add-header-svc` 值里面声明Service Name，对应值表示：
1 - 添加
0 - 不添加
一般与灰度比例或者条件路由配合使用

### Ingress Example
```Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    custom.nginx.org/canary-multi: "true"
    custom.nginx.org/canary-by-path: "/"
    custom.nginx.org/canary-by-cookie: "version"
    custom.nginx.org/canary-by-cookie-value: "v1 1, default 0"
    custom.nginx.org/canary-by-header: "header"
    custom.nginx.org/canary-by-header-value: "foo 1, default 0"
    custom.nginx.org/canary-by-multicondition: "11 /match11, 10 /match10, 01 /match01, 00 /match00"
    custom.nginx.org/canary-add-header-svc: "tea-svc 1, coffee-svc 0"
    custom.nginx.org/canary-add-header: "Connection close, X-Real-IP $remote_addr"
    nginx.org/rewrites: "serviceName=tea-svc rewrite=$request_uri;serviceName=coffee-svc rewrite=$request_uri"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /match11
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match00
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match10
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /match01
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
```

## 10 - Service Degrade 服务降级
* `custom.nginx.org/degrade-svc: "/nonecore 1, /core 0"`
`custom.nginx.org/degrade-svc`值里面声明api对应的值表示：
1 - 降级
0 - 正常转发
一般与限流限速（limit connection, limit request）配合使用

### Ingress Example
```Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    custom.nginx.org/rate-limiting: "on"
    custom.nginx.org/rate-limiting-rate: "500r/s"
    custom.nginx.org/rate-limiting-burst: "3"
    custom.nginx.org/degrade-svc: "/nonecore 1, /core 0"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /nonecore
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /core
```

## 11 - Custome Access Log and Error Log 租户自定义访问日志和错误日志
* `custom.nginx.org/custom-log-format: "$remote_addr - $remote_user [$time_local] \"$request\""`
* `custom.nginx.org/custom-access-log-off: "true"` #默认不配置即开启
* `custom.nginx.org/custom-error-log: "debug|info|notice|warn|error|crit|alert|emerg"` # 默认不配置即没有

说明：
1. 如果配置`custom.nginx.org/custom-log-format`即表示启用access log
2. 如果需要关闭access log则不需要配置`custom.nginx.org/custom-log-format`，配置`custom.nginx.org/custom-access-log-off: "true"`
3. error log默认关闭，如需启用配置`custom.nginx.org/custom-error-log: "info"`

### Ingress Example： access log and error log ON
```Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    custom.nginx.org/custom-log-format: "$remote_addr - $remote_user [$time_local] \"$request\""
    custom.nginx.org/custom-error-log: "info"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /tea
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /coffee
```

### Ingress Example: access log and error log OFF
```Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    custom.nginx.org/custom-access-log-off: "true"
  name: cafe-ingress
  namespace: default
spec:
  rules:
  - host: cafe.example.com
    http:
      paths:
      - backend:
          serviceName: tea-svc
          servicePort: 80
        path: /tea
      - backend:
          serviceName: coffee-svc
          servicePort: 80
        path: /coffee
```

