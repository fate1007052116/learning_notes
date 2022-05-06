# favicon.ico 异常

```html
<link rel="shortcut icon" href="../resources/favicon.ico" th:href="@{/static/favicon.ico}"/>
```

# 需要放在<head></head> 里面

```html
<link rel="icon" th:href="@{/favicon.ico}" type="image/x-icon"/>
<link rel="bookmark" th:href="@{/favicon.ico}" type="image/x-icon"/>
```

# 当还有一个时，一定报错

```html
<link th:href="@{favicon.ico}" rel="stylesheet"/>
<link rel="shortcut icon" href="../resources/favicon.ico" th:href="@{/static/favicon.ico}"/>
```

