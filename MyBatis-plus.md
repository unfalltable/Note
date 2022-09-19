## 自动填充

- 实现 MetaObjectHandler 接口
  - 实现 insertFill 和 updateFill 方法
    - 可以先判断对象是否有赋值再决定是否需要自动填充
      - `this.getFieldValByName("属性名", metaObject)` 
    - 判断当前对象是否包含需要填充的属性
      - `metaObject.hasSetter("author")` 
  - 在字段上添加 @TableField(fill = FieldFill.INSERT/UPDATE)

## 注解

#### @TableName

- 对应数据库表	

### @TableField

- 对于数据库字段名，可以设置自动填充

### @TableLogic

- 逻辑删除
  - 可以进行数据恢复
- 需要为数据添加一个 is_deleted 字段
- 执行删除操作时实际上是执行更新操作，设置为已删除状态
  - `is_deleted` 需要有默认值0

## 分页插件

- 





