# Custom Annotations

Custom annotations enable you to quickly extend the Ingress resource to support many advanced features of NGINX, such as rate limiting, caching, etc.

## Prerequisites 

* Read the [custom annotations doc](https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/custom-annotations/) before going through this example first.
* Read about [custom templates](../custom-templates).

## 1 - Rewrite With HostName

### [rewrite]
* `custom.nginx.org/rewrite-with-host: "https://www.baidu.com"` - configures the URL with hostname to rewrite
### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
...
server {
  ...
  location {
    ...
    # handling custom.nginx.org/rewrite-with-host  
    {{if index $.Ingress.Annotations "custom.nginx.org/rewrite-with-host"}}
    {{with $rewritehost := index $.Ingress.Annotations "custom.nginx.org/rewrite-with-host"}}
    rewrite ^{{$location.Path}}/?(.*) {{$rewritehost}} break;
    {{end}} 
    {{end}} 
    ...
  }
  ...
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    custom.nginx.org/rewrite-with-host: "https://www.baidu.com"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
```

## 2 - Proxy Set Header

### [proxy_set_header]
* `custom.nginx.org/set-header: "Connection close, X-Real-IP $remote_addr"`

### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
...
server {
  ...
  location {
    ...
    # handling custom.nginx.org/custom.nginx.org/set-header
    {{range $setheader := split (index $.Ingress.Annotations "custom.nginx.org/set-header") ","}} 
    proxy_set_header {{trim $setheader}}; 
    {{end}}
    ...
  }
  ...
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    custom.nginx.org/set-header: "Connection close, X-Real-IP $remote_addr"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
```

## 3 - Response Add Header

### [add_header]
* `custom.nginx.org/add-header: "Cache-Control private, test 111"`

### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
...
server {
  ...
  location {
    ...
    # handling custom.nginx.org/custom.nginx.org/add-header
    {{range $addheader := split (index $.Ingress.Annotations "custom.nginx.org/add-header") ","}} 
    add_header {{trim $addheader}} always; 
    {{end}}
    ...
  }
  ...
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    custom.nginx.org/add-header: "Cache-Control private, test 111"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
```

## 4 - Limit Connection

### [limit_conn]
* `custom.nginx.org/conn-limiting: "on"`
* `custom.nginx.org/conn-limiting-num: "1000"`

### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
# handling custom.nginx.org/conn-limiting and custom.nginx.org/conn-limiting-num 
{{if index $.Ingress.Annotations "custom.nginx.org/conn-limiting"}}
limit_conn_zone $binary_remote_addr zone={{$.Ingress.Namespace}}-{{$.Ingress.Name}}-conn:10m;
{{end}}
...
server {
  ...
  location {
    ...
    # handling custom.nginx.org/conn-limiting-num
    {{if index $.Ingress.Annotations "custom.nginx.org/conn-limiting"}}
    {{with $conn := index $.Ingress.Annotations "custom.nginx.org/conn-limiting-num"}}
    limit_conn {{$.Ingress.Namespace}}-{{$.Ingress.Name}}-conn {{$conn}};
    limit_conn_log_level error;
    limit_conn_status 503;
    {{end}} 
    {{end}}
    ...
  }
  ...
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    custom.nginx.org/conn-limiting: "on"
    custom.nginx.org/conn-limiting-num: "1000"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
```

## 5 - Limit Request

### [limit_req]
* `custom.nginx.org/rate-limiting: "on"`
* `custom.nginx.org/rate-limiting-rate: "500r/s"`
* `custom.nginx.org/rate-limiting-burst: "3"`

### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
# handling custom.nginx.org/rate-limiting and custom.nginx.org/rate-limiting-rate 
{{if index $.Ingress.Annotations "custom.nginx.org/rate-limiting"}}
{{$rate := index $.Ingress.Annotations "custom.nginx.org/rate-limiting-rate"}}
limit_req_zone $binary_remote_addr zone={{$.Ingress.Namespace}}-{{$.Ingress.Name}}-req:10m rate={{if $rate}}{{$rate}}{{end}};
{{end}}
...
server {
  ...
  location {
    ...
    # handling custom.nginx.org/rate-limiting and custom.nginx.org/rate-limiting-burst
    {{if index $.Ingress.Annotations "custom.nginx.org/rate-limiting"}}
    {{with $burst := index $.Ingress.Annotations "custom.nginx.org/rate-limiting-burst"}}
    limit_req zone={{$.Ingress.Namespace}}-{{$.Ingress.Name}}-req {{if $burst}}burst={{$burst}}{{else}}nodelay{{end}};
    limit_req_log_level error;
    limit_req_status 503;
    {{end}} 
    {{end}}
    ...
  }
  ...
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    custom.nginx.org/rate-limiting: "on"
    custom.nginx.org/rate-limiting-rate: "500r/s"
    custom.nginx.org/rate-limiting-burst: "3"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
```

## 6 - Weight Canary

### [weight]
* `custom.nginx.org/canary: "true"`
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-weight: "70% /match0, 30% /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"`

### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
{{if index $.Ingress.Annotations "custom.nginx.org/canary-by-weight"}}
split_clients "${remote_addr}${remote_port}AAA" $match {
{{range $splitclient := split (index $.Ingress.Annotations "custom.nginx.org/canary-by-weight") ","}} 
    {{trim $splitclient}}; 
{{end}}
}
{{end}}
...
server {
  ...
  location {
    ...
  }
  ...
  # handling custom.nginx.org/canary-by-path
  {{if index $.Ingress.Annotations "custom.nginx.org/canary-by-path"}}
  {{with $canarypath := index $.Ingress.Annotations "custom.nginx.org/canary-by-path"}}
  location {{if $canarypath}}{{$canarypath}}{{end}} {
      rewrite ^ $match last;
   }
  {{end}}
  {{end}}
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    custom.nginx.org/canary: "true"
    custom.nginx.org/canary-by-path: "/tea"
    custom.nginx.org/canary-by-weight: "70% /match0, 30% /match1"
    nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /match0
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
      - path: /match1
        pathType: Prefix
        backend:
          service:
            name: tea-v2-svc
            port:
              number: 80
```

## 7 - Cookie Canary

Cookie Canary和 Weight Canary (按比例灰度) 不能同时使用。

### [cookie]
* `custom.nginx.org/canary: "true"`
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-cookie: "version"`
* `custom.nginx.org/canary-by-cookie-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"`

### Step 1 - Customizs the Template
```nginx-plus-ingress.tmpl
{{if index $.Ingress.Annotations "custom.nginx.org/canary-by-cookie"}}
{{with $canarycookie := index $.Ingress.Annotations "custom.nginx.org/canary-by-cookie"}}
map $cookie_{{$canarycookie}} $match {
    {{if index $.Ingress.Annotations "custom.nginx.org/canary-by-cookie-value"}}
    {{range $cookievalue := split (index $.Ingress.Annotations "custom.nginx.org/canary-by-cookie-value") ","}} 
    {{trim $cookievalue}}; 
    {{end}}
    {{end}}
}
{{end}}
{{end}}
...
server {
  ...
  location {
    ...
  }
  ...
  # handling custom.nginx.org/canary-by-path
  {{if index $.Ingress.Annotations "custom.nginx.org/canary-by-path"}}
  {{with $canarypath := index $.Ingress.Annotations "custom.nginx.org/canary-by-path"}}
  location {{if $canarypath}}{{$canarypath}}{{end}} {
      rewrite ^ $match last;
   }
  {{end}}
  {{end}}
}
```

### Step 2 - Use Custom Annotations in an Ingress Resource
```ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cafe-ingress
  annotations:
    custom.nginx.org/canary: "true"
    custom.nginx.org/canary-by-path: "/tea"
    custom.nginx.org/canary-by-cookie: "version"
    custom.nginx.org/canary-by-cookie-value: "v1 /match0, default /match1"
    nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /match0
        pathType: Prefix
        backend:
          service:
            name: tea-svc
            port:
              number: 80
      - path: /match1
        pathType: Prefix
        backend:
          service:
            name: tea-v2-svc
            port:
              number: 80
```

## 8 - Others Canary
Header, Argument, Variable Canary 同 Cookie Canary

### Header
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-header: "version"`
* `custom.nginx.org/canary-by-header-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"`

### Argument
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-argument: "version"`
* `custom.nginx.org/canary-by-argument-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"`

### Variable
* `custom.nginx.org/canary-by-path: "/tea"`
* `custom.nginx.org/canary-by-variable: "version"`
* `custom.nginx.org/canary-by-variable-value: "v1 /match0, default /match1"`
* `nginx.org/rewrites: "serviceName=tea-svc rewrite=/tea;serviceName=tea-v2-svc rewrite=/tea"`


## 9 - Multi Condition Canary
待补充
```
custom.nginx.org/canary-by-condition: |
  {
    "condition": [
      {"cookie": "version", "value": "v1", "opt": "~*"},
      {"header": "testheader", "value": "testheader", "opt": "="},
      {"argument": "john", "value": "far", "opt": "~*"},
      {"variable": "$request_method", "value": "POST", "opt": "!~*"}
    ],
    "pass": "tea"  
  }
```
### 运算符解析：
- 使用"="和"!="运算符将变量与字符串进行比较
- "~"区分大小写匹配
- "~*"不区分大小写匹配
- "!~"区分大小写不匹配
- "!~*"不区分大小写不匹配
