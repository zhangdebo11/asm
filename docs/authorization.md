# AuthorizationPolicy

AuthorizationPolicy是istio中用来限制访问的，可以设置黑名单和白名单，规定哪些请求可以通过，哪些不能通过。

在SSC相关应用中，我们使用白名单，不在白名单中的请求会被拒绝。

在为应用设置时，需要注意，除了armor规则中包含的API要加入白名单之外，还要把以下API加入：
- 健康检查所用的API
- prometheus监控调用的API，即`/metrics`

# 参考资料
https://istio.io/latest/docs/concepts/security/#authorization
