恒煊注册【Q-——333307——】恒煊注册【 辋芷《888yx●vip》 】
恒煊注册【Q-——333307——】恒煊注册【 辋芷《888yx●vip》 】

 Spring Boot 3 + MyBatis-Plus 多租户实战：3分钟搞定数据隔离，附避坑指南

> 多租户不再是大型平台的专利，用对工具，小团队也能轻松实现。

在实际业务中，SaaS 系统最头疼的问题之一就是数据隔离。如果每个租户建一套数据库，运维成本高到怀疑人生；如果共享一张表，又担心数据串味。今天咱们聊聊怎么用 Spring Boot 3 搭配 MyBatis-Plus 3.5.x 的分页插件，无需改业务代码，就能实现行级数据隔离。

 核心方案：拦截器自动拼接条件

MyBatis-Plus 提供了 `TenantLineInnerInterceptor` 插件，它会在 SQL 执行前自动追加 `WHERE tenant_id = ?` 条件，开发者根本感知不到。

第一步：创建租户上下文

```java
public class TenantContext {
    private static final ThreadLocal<Long> CURRENT = new ThreadLocal<>();
    public static void set(Long id) { CURRENT.set(id); }
    public static Long get() { return CURRENT.get(); }
    public static void clear() { CURRENT.remove(); }
}
```

第二步：注册拦截器（关键配置）

```java
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 多租户插件
        TenantLineInnerInterceptor tenant = new TenantLineInnerInterceptor(
            new TenantLineHandler() {
                @Override
                public Expression getTenantId() {
                    // 从上下文取值，没取到就抛异常
                    Long id = TenantContext.get();
                    if (id == null) throw new RuntimeException("租户ID不存在");
                    return new LongValue(id);
                }
                @Override
                public String getTenantIdColumn() {
                    return "tenant_id"; // 对应要忽略的表？在这里排除
                }
            }
        );
        interceptor.addInnerInterceptor(tenant);
        // 分页插件放最后
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

第三步：在请求入口绑定租户

比如在 Filter 或 HandlerInterceptor 里：

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    String tenantId = request.getHeader("X-TENANT-ID");
    TenantContext.set(Long.valueOf(tenantId));
    return true;
}
@Override
public void afterCompletion(...) { TenantContext.clear(); }
```

 避坑指南（重点）

1. 分页 Count SQL 被误加条件 —— 如果你用了 `@SqlParser(filter = true)` 或者表别名，可能会覆盖掉租户条件。建议在多租户插件里体检 `ignoreTable()` 方法，把 `sys_config` 这类公共表排除掉。

2. 线程池异步导致上下文丢失 —— 记住 `ThreadLocal` 不能跨线程。用 `Async` 注解或 `CompletableFuture` 时，一定要手动透传租户 ID。

3. 联合主键或复杂 OR 场景 —— 如果自己的 SQL 里写了 `or`，插件无法智能拆分条件，会导致数据泄漏。解决办法：尽量避免租户字段参与 `or`，或者把租户条件提到最外层。

 互动引导

你有遇到过租户条件丢失导致的数据混乱吗？还是用别的姿势实现的多租户？评论区聊聊你的方案。

点赞 把这篇干货转给后端组的小伙伴，下次联调少踩几个坑。

---

相关标签：Spring Boot 多租户、MyBatis-Plus 教程、SaaS 数据隔离、Java 后端架构

相关推荐：

https://github.com/rossmarissa09/kqyzhh/blob/main/2027%E5%AE%98%E7%BD%91%E7%A7%91%E6%99%AE%EF%BC%9A%E6%84%8F%E6%98%82%E4%BD%93%E8%82%B2%E5%9C%B0%E5%9D%80%E4%B8%8B%E8%BD%BD_%E4%BF%A3%E5%BB%8A%E7%8E%87%E6%A9%99%E8%AF%9CJQTES.md

<img src="https://i.postimg.cc/wMMhx5x6/hengxuan-00014.png" />

相关推荐：

https://github.com/rossmarissa09/kqyzhh/commit/7ffb904f690548f6fcbbfc631474cc7fff24b082

<img src="https://i.postimg.cc/wMMhx5x6/hengxuan-00014.png" />
相关推荐：

https://github.com/carrollduane3403/iavdsm/blob/main/%E4%BF%9D%E5%A7%86%E5%AE%9E%E6%93%8D%E6%94%BB%E7%95%A5%EF%BC%9A%E6%84%8F%E6%98%82%E4%BD%93%E8%82%B2%E5%9C%B0%E5%9D%80app_%E6%8B%90%E9%93%A3%E6%8C%89%E6%AD%A4%E6%B9%8DZFHOU.md

<img src="https://i.postimg.cc/FRDyQC4n/hengxuan-00007.png" />
相关推荐：

https://github.com/carrollduane3403/iavdsm/commit/804d132a49a95c24a602b9afd5d028a5b9350d67

<img src="https://i.postimg.cc/3NNgrj8n/hengxuan-00011.png" />

资讯来源：新华网、人民网、央视新闻、中新网、凤凰网、澎湃新闻、界面新闻、新浪新闻、搜狐网、财新网、观察者网、第一财经等主流平台，以独树一帜的观察视角与扎实的深度报道能力，在资讯领域收获广泛关注。
