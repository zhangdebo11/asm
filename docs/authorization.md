# AuthorizationPolicy

AuthorizationPolicy是istio中用来限制访问的，可以设置黑名单和白名单，规定哪些请求可以通过，哪些不能通过。

在SSC相关应用中，我们使用白名单，不在白名单中的请求会被拒绝。

在为应用设置时，需要注意，除了armor规则中包含的API要加入白名单之外，还要把以下API加入：
- 健康检查所用的API，包括 `/`，`/ping`
- prometheus监控调用的API，即`/metrics`

设置后请留意ingress状态是否正常，留意端口监控的数据是否正常。

为便于管理，CI中会自动生成API列表并push到helm模板中。

# 参考资料
https://istio.io/latest/docs/concepts/security/#authorization
