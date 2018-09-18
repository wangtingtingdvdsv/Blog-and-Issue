# 今天在做toDo这个Demo时，遇见了个奇葩的问题，
# 在地址栏输入http://localhost:8001/会重定向到http://localhost:8001/login,
但是我如果在地址栏直接输入http://localhost:8001/login则会报这样的错

```

Refused to load the font '<URL>' because it violates the following Content Security Policy directive: "default-src 'self'". Note that 'font-src' was not explicitly set, so 'default-src' is used as a fallback.

```
