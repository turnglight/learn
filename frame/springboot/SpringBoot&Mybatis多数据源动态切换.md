## 核心原理
> 初始化多个数据源，通过继承AbstractRoutingDataSource，设置defaultTargetDataSource和targetDataSources。所有的mapper在连接数据库前都要获取数据库连接，指定数据源的核心方法为determineCurrentLookupKey，最终的数据源为该方法返回的数据源key

+ 第一步：配置数据源
~~~yml

server:
  port: 18888

# swagger中存在bug，在使用@ApiModelProperty注解在字段上时，如果字段的类型为Long或是int类型，那么程序启动后，访问swagger-ui.html的页面，程序会报错
logging:
  level:
    io.swagger.models.parameters.AbstractSerializableParameter: error


spring:
  devtools:
    restart:
      enabled: true
      additional-paths: src/main/java
      exclude: WEB-INF/**
  application:
    name: ntms-cloud-asset
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
#    url: jdbc:mysql://101.132.27.103:3306/ntmsprovin_shanxi?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&zeroDateTimeBehavior=convertToNull
#    username: root
#    password: shfcqy
    druid:
      ONE:
        url: jdbc:mysql://192.168.0.2:3111/ntmsprovin_shanxi_20200117?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&zeroDateTimeBehavior=convertToNull
        username: root
        password: roottest
      TWO:
        url: jdbc:mysql://192.168.0.2:3111/ntmsprovin_shanxi_20200117_test?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&zeroDateTimeBehavior=convertToNull
        username: root
        password: roottest
      THREE:
        url: jdbc:mysql://192.168.0.2:3111/ntmsprovin_shanxi_20200401?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&zeroDateTimeBehavior=convertToNull
        username: root
        password: roottest
      initialSize: 1 #初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时
      minIdle: 1  #最小连接池数量
      maxActive: 20 #最大连接池数量
      maxWait: 60000 #获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1
      testWhileIdle: true
      testOnBorrow: true
      testOnReturn: false
      poolPreparedStatements: true  #是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql5.5以下的版本中没有PSCache功能，建议关闭掉。5.5及以上版本有PSCache，建议开启。
      maxPoolPreparedStatementPerConnectionSize: 20
      filters: stat,wall  #属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：监控统计用的filter:stat，日志用的filter:log4j， 防御sql注入的filter:wall
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
      stat-view-servlet:
        allow: 47.111.96.214
  redis:
    database: 0
    host: 101.132.27.103
    port: 6379
#    password: redis-password
    jedis:
      pool:
        min-idle: 0
        max-idle: 8
        max-wait: 1
        max-active: 8
    timeout: 30000
  data:
    web:
      pageable:
        default-page-size: 10
~~~
+ 第二步：读取配置文件，初始化数据源，注入bean
~~~java
package ntms.cloud.asset.config;

import com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

/**
 * 配置多数据源
 * @author 代雅男
 * @date 2020/4/10 13:48
 **/
public class DynamicDataSourceConfig {

    /**
     * 创建 DataSource Bean
     * */

    @Bean
    @ConfigurationProperties("spring.datasource.druid.one")
    public DataSource oneDataSource(){
        DataSource dataSource = DruidDataSourceBuilder.create().build();
        return dataSource;
    }

    @Bean
    @ConfigurationProperties("spring.datasource.druid.two")
    public DataSource twoDataSource(){
        DataSource dataSource = DruidDataSourceBuilder.create().build();
        return dataSource;
    }

    @Bean
    @ConfigurationProperties("spring.datasource.druid.three")
    public DataSource threeDataSource(){
        DataSource dataSource = DruidDataSourceBuilder.create().build();
        return dataSource;
    }

    /**
     * 如果还有数据源,在这继续添加 DataSource Bean
     * */

    @Bean
    @Primary
    public DynamicDataSource dataSource(DataSource oneDataSource, DataSource twoDataSource, DataSource threeDataSource) {
        Map<Object, Object> targetDataSources = new HashMap<>(3);
        targetDataSources.put(DataSourceNames.ONE, oneDataSource);
        targetDataSources.put(DataSourceNames.TWO, twoDataSource);
        targetDataSources.put(DataSourceNames.THREE, threeDataSource);
        // 还有数据源,在targetDataSources中继续添加
        System.out.println("DataSources:" + targetDataSources);
        return new DynamicDataSource(oneDataSource, targetDataSources);
    }

}

~~~
+ 第三步：继承AbstractRoutingDataSource，将所有数据源注入targetDataSources,并设置默认数据源defaultTargetDataSource，实现determineCurrentLookupKey方法
~~~java
package ntms.cloud.asset.config;

import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import javax.sql.DataSource;
import java.util.Map;

/**
 * @author 代雅男
 * @date 2020/4/10 13:46
 **/
public class DynamicDataSource extends AbstractRoutingDataSource {
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    /**
     * 配置DataSource, defaultTargetDataSource为主数据库
     */
    public DynamicDataSource(DataSource defaultTargetDataSource, Map<Object, Object> targetDataSources) {
        super.setDefaultTargetDataSource(defaultTargetDataSource);
        super.setTargetDataSources(targetDataSources);
        super.afterPropertiesSet();
    }

    @Override
    protected Object determineCurrentLookupKey() {
        return getDataSource();
    }

    public static void setDataSource(String dataSource) {
        contextHolder.set(dataSource);
    }

    public static String getDataSource() {
        return contextHolder.get();
    }

    public static void clearDataSource() {
        contextHolder.remove();
    }
}

~~~

+ 第四步：通过一个线程安全的上下文数据源holder来进行数据源切换，保证每一个线程的数据源切换时独立的。
~~~java
private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
~~~


## 切换方式
+ filter切换
> 读取request的header指定数据源
~~~java
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        String city = request.getHeader("city");
        DynamicDataSource.setDataSource(CacheUtil.dbMap.get(city));
        filterChain.doFilter(request,response);
    }
~~~

+ aop切换
> 自定义数据源注解，对注解进行aop数据源切换
~~~java
@Aspect
@Component
@Slf4j
public class DynamicDataSourceAspect {
    @Around("@within(org.apache.ibatis.annotations.Mapper)")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        Class<?> target = point.getTarget().getClass();
        MethodSignature signature = (MethodSignature) point.getSignature();
        boolean resolve = false;
        for (Class<?> clazz : target.getInterfaces()) {
            resolveDataSource(clazz, signature.getMethod());
        }
        Object result = point.proceed();
        try {
            return point.proceed();
        } finally {
            DynamicDataSource.clearDataSource();
            log.debug("clean datasource");
        }
    }

    /**
     * 获取目标对象方法注解和类型注解中的注解
     */
    private void resolveDataSource(Class<?> clazz, Method method) {
        try {
            Class<?>[] types = method.getParameterTypes();

            // 方法上注解
            Method m = clazz.getMethod(method.getName(), types);
            if (m != null && m.isAnnotationPresent(DataSource.class)) {
                DataSource ds = m.getAnnotation(DataSource.class);
                String key = ds.value().name().toLowerCase();
                DynamicDataSourceContextHolder.setDataSource(key);
                log.info("Switch DataSource to [" + DynamicDataSourceContextHolder.getDataSource()
                        + "] in Method [" + method + "]");
                return;
            }
            // 类上注解
            if (clazz.isAnnotationPresent(DataSource.class)) {
                DataSource ds = clazz.getAnnotation(DataSource.class);
                String key = ds.value().name().toLowerCase();
                DynamicDataSourceContextHolder.setDataSource(key);
                log.info("Switch DataSource to [" + DynamicDataSourceContextHolder.getDataSource()
                        + "] in Method [" + method + "]");
                return;
            }
        } catch (Exception e) {
            log.error(clazz + ":" + e.getMessage());
        }
    }
}


~~~