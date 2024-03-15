* **ES的应用场景**
    * [1.ES中数据更新未同步](#ES数据更新)
    * [2.ES中UTC时间问题](#UTC时间)
    * [3.ES英文大小写查询问题](#查询大小写)


## 1.ES中数据更新未同步

### Q：为什么数据库中数据状态已更新，且同步更新了es，紧接着调用查询接口，发现es中数据并没更新。

#### A： es的默认机制问题,默认1s更新数据才会生效导致。 因为实时刷新会导致性能问题,es性能将会比默认1s刷新降低10倍

### 场景分析
如果无数据严格的同步要求，建议增加1s的响应时间，如，更新操作后等待1s。 同步带来的是内存和cpu的资源紧张，10w数据同步更新，耗时约是
不同步的10倍。（当前场景是单条更新，如果是批量更新，时间还会下降）



## 2.ES中UTC时间问题

### Q：在java代码中，使用Data类型传入时间是23：59：59，在使用es进行查询时发现，时间会自动转化成15：59：59

#### A： 因为ES默认是拿UTC时间存储。如果Java中数据类型是Data类型（会自动转化为系统所在时区时间）。当ES在存储或查询时，ES会判断数据类型，如果是Data类型自动增加或减少时间（根据系统默认时区）；使用LocalDateTime类型即可正常存储，不会额外转换。（数据库使用datetime类型做存储）

### 代码实现

    @Data
    @SuperBuilder
    @AllArgsConstructor
    @NoArgsConstructor
    public abstract class BaseEntity implements Serializable {
        private static final long serialVersionUID = 1L;
    
        @TableId(value = "id", type = IdType.AUTO)
        private Long id;
    
        private Boolean isDelete;
    
        @TableField(fill = FieldFill.INSERT)
        private LocalDateTime createdTime;
    
        private String createdUser;
    
        @TableField(fill = FieldFill.INSERT_UPDATE)
        private LocalDateTime updatedTime;
    
        private String updatedUser;
    }

    /**
     * 
     * handler统一处理
     * 
     */
    @Component
    @Slf4j
    public class MyMetaObjectHandler implements MetaObjectHandler {
        @Override
        public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createdTime", LocalDateTime.class, LocalDateTimeUtil.ofUTC(Instant.now()));
        this.strictUpdateFill(metaObject, "updatedTime", LocalDateTime.class, LocalDateTimeUtil.ofUTC(Instant.now()));
        }
        @Override
        public void updateFill(MetaObject metaObject) {
            this.strictUpdateFill(metaObject, "updatedTime", LocalDateTime.class, LocalDateTimeUtil.ofUTC(Instant.now()));
        }
    }

## 3.ES英文大小写查询问题

### Q：查询时，首字母大写的单词查询不到

#### A：使用es查询时, term不会处理匹配词，会发生首字母大写的单词查询不到，但使用match匹配可以成功搜索到数据，match会对匹配词进行处理，大小写转换，分词等。
