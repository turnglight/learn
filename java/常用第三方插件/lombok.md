# Lombok

~~~xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
</dependency>
~~~

## 注解
+ @getter/@setter
+ @ToString
+ @EqualsAndHashCode: 作用于类，覆盖默认的equals和hashCode
+ @NonNull: 作用于成员变量，标识不能为空，否则抛出空指针异常
+ @NoArgsConstructor, @RequiredArgsConstructor，@AllArgsConstructor：作用于类上，用于生成构造函数。有staticName、access等属性
+ @Data：作用于类上，是以下注解的集合：@ToString @EqualsAndHashCode @Getter @Setter @RequiredArgsConstructor
+ @Builder：作用于类上，将类转变为建造者模式


Asset/GetAssetTreeList_Province?UserID=f6047cca-ba5e-40aa-b5dc-2b44b73959ad

Asset/GetAssetTreeList_province?UserID=f6047cca-ba5e-40aa-b5dc-2b44b73959ad&SearchText=