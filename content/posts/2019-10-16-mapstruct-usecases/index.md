+++
title = "Mapper Classes Generated by MapStruct"
description = "对象实体转换操作在分层应用中很常见，比如数据库层的对象实体和表现层的实体很可能具有不同的属性集，需要在互操作时进行属性的映射（或拷贝）。 MapStruct 是一个辅助进行 Java 实体类之间相互转换的类库，与其他具有相似功能的工具库之间的最大区别在于其使用了 Java 注解处理器 APT 技术而不是常见的反射来实现对象实体间属性的映射。"
date = 2019-10-16T17:23:07+08:00
draft = false
template = "page.html"
[taxonomies]
categories =  ["Java"]
tags = ["java", "apt", "mapstruct", "autovalue", "freebuilder"]
+++

对象实体转换操作在分层应用中很常见，比如数据库层的对象实体和表现层的实体很可能具有不同的属性集，需要在互操作时进行属性的映射（或拷贝）。

[MapStruct][mapstruct] 是一个辅助进行 Java 实体类之间相互转换的类库，与其他具有相似功能的工具库之间的最大区别在于其使用了 [Java 注解处理器 APT][apt] 来实现实体间属性的映射而不是使用反射技术。

<!-- more -->

# Maven 配置

类似于 [Auto Value][autovalue]，[MapStruct][mapstruct] 需要引入两个组件：

1. `org.mapstruct:mapstruct` 包含各种定制代码生成行为规则的注解类、接口类和一个辅助构建 Mapper 接口
   实现类实例的 `org.mapstruct.factory.Mappers` 类；需要加入到项目的运行时类路径。
2. `org.mapstruct:mapstruct-processor` 包含在编译时生成 Mapper 实现类及相关代码的注解处理器实现；只
   需要出现在项目的编译时类路径。

对于 [Maven][maven] 版本大于 `3.5` 的，可以如下配置使用 [MapStruct][mapstruct] 。

```
<dependencies>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>${maven_plugin_compiler_version}</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

如欲详细了解如何在 [Maven][maven] 中配置使用注解处理器类实现，请参见 [在 Maven 中支持 Java 的注解处理器 APT](@/posts/2019-09-30-add-supports-for-apt-in-maven/index.md)。

截至 2019-10-16，[MapStruct][mapstruct] 的最新版本是 `1.3.1.Final` 。

# POJO 模式实体对象与代码生成

## 基本操作

如果有两个需要相互转换的类 `SourceEntity` 和 `TargetDto`：

```java
public class SourceEntity {
    private int intProperty;
    private boolean boolProperty;
    private Long longProperty;
    private String srtProperty;
    private OffsetDateTime dateTime;
    private List<BigDecimal> numbers;
    private Map<String, String> types;
}

public class TargetDto {
    private int intProperty;
    private boolean boolProperty;
    private Long longProperty;
    private String srtProperty;
    private OffsetDateTime dateTime;
    private List<BigDecimal> numbers;
    private Map<String, String> types;
}
```

使用 [MapStruct][mapstruct] 来生成转换类，需要定义一个转换接口类：

```java
@Mapper
public interface TypeMapper {
    TargetDto convert(final SourceEntity from);

    SourceEntity convert(final TargetDto from);
}
```

在接口类中声明所需方法，方法的参数是源类型实例，返回值是目标类型；接口类上需要添加 `org.mapstruct.Mapper` 注解。

运行 `mvn clean compile` 之后就能看到 [MapStruct][mapstruct] 生成的 `TypeMapper` 实现类 `TypeMapperImpl`的代码，部分代码如下：

```java
@Override
public TargetDto convert(SourceEntity from) {
    if ( from == null ) {
        return null;
    }

    TargetDto targetDto = new TargetDto();

    targetDto.setIntProperty( from.getIntProperty() );
    targetDto.setBoolProperty( from.isBoolProperty() );
    targetDto.setLongProperty( from.getLongProperty() );
    targetDto.setSrtProperty( from.getSrtProperty() );
    targetDto.setDateTime( from.getDateTime() );
    List<BigDecimal> list = from.getNumbers();
    if ( list != null ) {
        targetDto.setNumbers( new ArrayList<BigDecimal>( list ) );
    }
    Map<String, String> map = from.getTypes();
    if ( map != null ) {
        targetDto.setTypes( new HashMap<String, String>( map ) );
    }

    return targetDto;
}

@Override
public SourceEntity convert(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    SourceEntity sourceEntity = new SourceEntity();

    sourceEntity.setIntProperty( from.getIntProperty() );
    sourceEntity.setBoolProperty( from.isBoolProperty() );
    sourceEntity.setLongProperty( from.getLongProperty() );
    sourceEntity.setSrtProperty( from.getSrtProperty() );
    sourceEntity.setDateTime( from.getDateTime() );
    List<BigDecimal> list = from.getNumbers();
    if ( list != null ) {
        sourceEntity.setNumbers( new ArrayList<BigDecimal>( list ) );
    }
    Map<String, String> map = from.getTypes();
    if ( map != null ) {
        sourceEntity.setTypes( new HashMap<String, String>( map ) );
    }

    return sourceEntity;
}
```

注意到生成的代码中对 `List` 和 `Map` 类型做了特殊处理，在转换时进行了 **深度拷贝** 以避免不必要的数据共享。

## 字段名不一样

使用时，[MapStruct][mapstruct] 需要源类型和目标类型的属性集合类型和名称保持一致，但这个要求通常很难达到；对于这种需求，[MapStruct][mapstruct] 提供了解法。

比如，如果有两个实体类：

```java
public class SourceEntity {
    private int intProperty;
    private String strProperty;
    private OffsetDateTime createdAt;
}

public class TargetDto {
    private int intProperty;
    private String strProperty;
    private OffsetDateTime createTime;
}
```

两个实体类中的属性名有的相同有的不相同 (`createdAt` VS `createTime`) 。

继续使用之前定义的 Mapper，发现生成的代码中忽略了不匹配的属性。

```java
@Override
public TargetDto convert(SourceEntity from) {
    if ( from == null ) {
        return null;
    }

    TargetDto targetDto = new TargetDto();

    targetDto.setIntProperty( from.getIntProperty() );
    targetDto.setStrProperty( from.getStrProperty() );

    return targetDto;
}

@Override
public SourceEntity convert(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    SourceEntity sourceEntity = new SourceEntity();

    sourceEntity.setIntProperty( from.getIntProperty() );
    sourceEntity.setStrProperty( from.getStrProperty() );

    return sourceEntity;
}
```

为了避免这种情况，可以设置 `@Mapper` 注解的属性，如下：

```java
@Mapper(unmappedSourcePolicy = ReportingPolicy.WARN, unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface TypeMapper {
    // Omitted
}
```

此时，编译器就会报错，并提示未匹配属性的错误: `Unmapped target property: "createTime"` 。

然后可以在方法级别添加 `org.mapstruct.Mapping` 注解，并设置其 `source` 和 `target` 属性：

```java
@Mapper(unmappedSourcePolicy = ReportingPolicy.WARN, unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface TypeMapper {

    @Mapping(source = "createdAt", target = "createTime")
    TargetDto convert(final SourceEntity from);

    @Mapping(source = "createTime", target = "createdAt")
    SourceEntity convert(final TargetDto from);
}
```

`source` 属性的值设置为源类型里面的属性名，`target` 的值设置为目标类型里面对应的属性名。`@Mapping`
注解可以应用多次，以指定多个属性的映射规则。

生成的代码中现在已经可以包含预期的代码了：

```java
@Override
public TargetDto convert(SourceEntity from) {
    if ( from == null ) {
        return null;
    }

    TargetDto targetDto = new TargetDto();

    targetDto.setCreateTime( from.getCreatedAt() );
    targetDto.setIntProperty( from.getIntProperty() );
    targetDto.setStrProperty( from.getStrProperty() );

    return targetDto;
}

@Override
public SourceEntity convert(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    SourceEntity sourceEntity = new SourceEntity();

    sourceEntity.setCreatedAt( from.getCreateTime() );
    sourceEntity.setIntProperty( from.getIntProperty() );
    sourceEntity.setStrProperty( from.getStrProperty() );

    return sourceEntity;
}
```

## 字段类型不一致

如果有属性的类型不匹配（但是可以转换）怎么办呢？

例如，源类型里面有一个 `int` 类型的属性表示 `种类` 这个状态，目标类型中对应的属性类型是预定义的枚举值。

```java
public class SourceEntity {
    private int intType;
    private String strProperty;
    private OffsetDateTime createdAt;
}

public enum TypeEnum {
    LARGE, MEDIUM, SMALL
}

public class TargetDto {
    private TypeEnum enumType;
    private String strProperty;
    private OffsetDateTime createTime;
}
```

此时，编译器可能会报错：`Can't map property "int intType" to "TypeEnum enumType". Consider to declare/implement a mapping method: "TypeEnum map(int value)"`

按照错误说明中的提示，为 `TypeEnum` 和 `int` 添加单独的转换方法。

```java
@Mapper(unmappedSourcePolicy = ReportingPolicy.WARN, unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface TypeMapper {
    @Mapping(source = "createdAt", target = "createTime")
    @Mapping(source = "intType", target = "enumType")
    TargetDto convert(final SourceEntity from);

    @Mapping(source = "createTime", target = "createdAt")
    @Mapping(source = "enumType", target = "intType")
    SourceEntity convert(final TargetDto from);

    default int mapType(final TypeEnum v) {
        switch (v) {
            case LARGE: return 1;
            case MEDIUM: return 2;
            default: return 3;
        }
    }

    static TypeEnum mapType(final int v) {
        switch (v) {
            case 1: return TypeEnum.LARGE;
            case 2: return TypeEnum.MEDIUM;
            default: return TypeEnum.SMALL;
        }
    }
}
```

单独的转换方法可以是 **静态方法** 也可以是 **默认方法**。

在生成的实现类代码中，自定义的转换方法会被调用。

```java
@Override
public TargetDto convert(SourceEntity from) {
    if ( from == null ) {
        return null;
    }

    TargetDto targetDto = new TargetDto();

    targetDto.setEnumType( TypeMapper.mapType( from.getIntType() ) );
    targetDto.setCreateTime( from.getCreatedAt() );
    targetDto.setStrProperty( from.getStrProperty() );

    return targetDto;
}

@Override
public SourceEntity convert(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    SourceEntity sourceEntity = new SourceEntity();

    sourceEntity.setCreatedAt( from.getCreateTime() );
    sourceEntity.setIntType( mapType( from.getEnumType() ) );
    sourceEntity.setStrProperty( from.getStrProperty() );

    return sourceEntity;
}
```

## 多个源类型

如果遇到以下的这种情况：源数据有两个或多个类型，比如一个用户的数据被存储于两个数据表中，使用 [MapStruct][mapstruct] 也可以实现多个源和一个目标类型映射的自动代码生成（像你想的那样实现）。

```
public class AdditionalEntity {
    private OffsetDateTime updatedAt;
    private String updateBy;
    private String content;
}

public class SourceEntity {
    private int intType;
    private String strProperty;
    private OffsetDateTime createdAt;
}

public class TargetDto {
    private TypeEnum enumType;
    private String strProperty;
    private OffsetDateTime createTime;
    private OffsetDateTime lastUpdateTime;
}
```

按照下面的用法配置 `@Mapper` 和 `@Mapping` 注解：

```java
@Mapper(unmappedSourcePolicy = ReportingPolicy.WARN, unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface TypeMapper {
    @Mapping(source = "from.intType", target = "enumType")
    @Mapping(source = "from.createdAt", target = "createTime")
    @Mapping(source = "additional.updatedAt", target = "lastUpdateTime")
    TargetDto convert(final SourceEntity from, final AdditionalEntity additional);

    @Mapping(source = "enumType", target = "intType")
    @Mapping(source = "createTime", target = "createdAt")
    SourceEntity convertToSource(final TargetDto from);

    @Mapping(source = "lastUpdateTime", target = "updatedAt")
    @Mapping(target = "updatedBy", constant = "System")
    @Mapping(target = "content", expression = "java(\"Created at \" + java.time.OffsetDateTime.now())")
    AdditionalEntity convertToAdditional(final TargetDto from);

    default int mapType(final TypeEnum v) {
        switch (v) {
            case LARGE: return 1;
            case MEDIUM: return 2;
            default: return 3;
        }
    }

    static TypeEnum mapType(final int v) {
        switch (v) {
            case 1: return TypeEnum.LARGE;
            case 2: return TypeEnum.MEDIUM;
            default: return TypeEnum.SMALL;
        }
    }
}
```

`@Mapping` 注解可以支持为目标属性设置固定值（`constant`）或设置表达式值(`expression`)，而且由于 [MapStruct][mapstruct] 的编译时代码生成特性，不合法的值会直接在编译时报错。

如你所想，生成的代码正确地设置了目标属性值。

```java
@Override
public TargetDto convert(SourceEntity from, AdditionalEntity additional) {
    if ( from == null && additional == null ) {
        return null;
    }

    TargetDto targetDto = new TargetDto();

    if ( from != null ) {
        targetDto.setEnumType( TypeMapper.mapType( from.getIntType() ) );
        targetDto.setCreateTime( from.getCreatedAt() );
        targetDto.setStrProperty( from.getStrProperty() );
    }
    if ( additional != null ) {
        targetDto.setLastUpdateTime( additional.getUpdatedAt() );
    }

    return targetDto;
}

@Override
public SourceEntity convertToSource(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    SourceEntity sourceEntity = new SourceEntity();

    sourceEntity.setCreatedAt( from.getCreateTime() );
    sourceEntity.setIntType( mapType( from.getEnumType() ) );
    sourceEntity.setStrProperty( from.getStrProperty() );

    return sourceEntity;
}

@Override
public AdditionalEntity convertToAdditional(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    AdditionalEntity additionalEntity = new AdditionalEntity();

    additionalEntity.setUpdatedAt( from.getLastUpdateTime() );

    additionalEntity.setUpdatedBy( "System" );
    additionalEntity.setContent( "Created at " + java.time.OffsetDateTime.now() );

    return additionalEntity;
}
```

# Builder 模式实体对象与代码生成

[MapStruct][mapstruct] 除了对 POJO 的支持外，还提供对基于 Builder 模式的不可变对象的支持，只要满足以下条件：

1. 目标类中必须存在一个公共的静态无参方法可以构造一个 Builder 模式对象 (形如 `public static Builder builder()`)
2. 目标类的 Builder 模式类中必须存在一个公共的无参方法可以生成一个目标类对象 (形如 `public UserDto build()`)

在上述条件中，方法名可以自定义，但一般都约定为 `public static Builder builder()` 和 `public UserDto build()` 。

[AutoValue][autovalue] 的 Builder 模式和 [FreeBuilder][freebuilder] 都能够满足上述要求。

下述实体对象分别使用了 [AutoValue][autovalue] 和 [FreeBuilder][freebuilder]。

```java
@AutoValue
public abstract class SourceEntity {
    public abstract int getIntType();
    public abstract String getStrProperty();
    public abstract OffsetDateTime getCreatedAt();

    public static Builder builder() {
        return new AutoValue_SourceEntity.Builder();
    }

    @AutoValue.Builder
    public abstract static class Builder {
        public abstract Builder setIntType(int newIntType);
        public abstract Builder setStrProperty(String newStrProperty);
        public abstract Builder setCreatedAt(OffsetDateTime newCreatedAt);

        public abstract SourceEntity build();
    }
}

@FreeBuilder
public abstract class TargetDto {
    public abstract TypeEnum getEnumType();
    public abstract String getStrProperty();
    public abstract OffsetDateTime getCreateTime();

    public static Builder builder() {
        return new Builder();
    }

    static class Builder extends TargetDto_Builder { }
}
```

注意，在 [AutoValue][autovalue] 和 [FreeBuilder][freebuilder] 的用例中，声明属性的方法名必须符合 Bean 的规范 (即 Getter 方法必须以 `get` 或 `is` 开头，Setter 方法必须以 `set` 开头)。

Mapper 接口类与 POJO 时一样。

```java
@Mapper(unmappedSourcePolicy = ReportingPolicy.WARN, unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface TypeMapper {
    @Mapping(source = "intType", target = "enumType")
    @Mapping(source = "createdAt", target = "createTime")
    TargetDto convert(final SourceEntity from);

    @Mapping(source = "enumType", target = "intType")
    @Mapping(source = "createTime", target = "createdAt")
    SourceEntity convert(final TargetDto from);

    default int mapType(final TypeEnum v) {
        switch (v) {
            case LARGE: return 1;
            case MEDIUM: return 2;
            default: return 3;
        }
    }

    static TypeEnum mapType(final int v) {
        switch (v) {
            case 1: return TypeEnum.LARGE;
            case 2: return TypeEnum.MEDIUM;
            default: return TypeEnum.SMALL;
        }
    }
}
```

生成的实现类中使用了各自的 Builder 类。

```java
@Override
public TargetDto convert(SourceEntity from) {
    if ( from == null ) {
        return null;
    }

    TargetDto.Builder targetDto = TargetDto.builder();

    targetDto.setEnumType( TypeMapper.mapType( from.getIntType() ) );
    targetDto.setCreateTime( from.getCreatedAt() );
    targetDto.setStrProperty( from.getStrProperty() );

    return targetDto.build();
}

@Override
public SourceEntity convert(TargetDto from) {
    if ( from == null ) {
        return null;
    }

    SourceEntity.Builder sourceEntity = SourceEntity.builder();

    sourceEntity.setCreatedAt( from.getCreateTime() );
    sourceEntity.setIntType( mapType( from.getEnumType() ) );
    sourceEntity.setStrProperty( from.getStrProperty() );

    return sourceEntity.build();
}
```

# Mapper 实现类实例的获取

[MapStruct][mapstruct] 自动生成的 Mapper 接口的实现类的类名默认是接口名后面加上 `Impl` 后缀，如果有需要也可以自定义。

使用 Mapper 接口的实现时，可以手动调用实现类的 `new` 表达式。

```java
@Mapper
public interface TypeMapper {
    static TypeMapper instance() {
        return new TypeMapperImpl();
    }
}
```

或者使用 [MapStruct][mapstruct] 提供的 `org.mapstruct.factory.Mappers` 实用类。

```java
@Mapper
public interface TypeMapper {
    TypeMapper INSTANCE = Mappers.getMapper(TypeMapper.class);
}
```

如果使用了依赖注入框架 (如 [Contexts and Dependency Injection for JavaTM EE, CDI](https://jcp.org/en/jsr/detail?id=346) [Spring](https://projects.spring.io/spring-framework/) [Guice](https://github.com/google/guice) 等) 的话，也可以启用 [MapStruct][mapstruct] 对其的支持。

---

完整的示例代码可以参见 [MapStruct Usecases](https://github.com/wbprime/java-mods/tree/master/mapstruct-usecases) 。

想了解更多的 [MapStruct][mapstruct] 用法，可以阅读 [官方文档][documentation]，也可以去 [官方仓库](https://github.com/mapstruct/mapstruct-examples) 了解各种用例。

---

以上。

[apt]: https://docs.oracle.com/javase/7/docs/technotes/guides/apt/ "Annotation Processing Tool (apt)"
[maven]: http://maven.apache.org/ "Apache Maven is a software project management and comprehension tool."
[mapstruct]: https://mapstruct.org/ "Java bean mappings, the easy way!"
[documentation]: https://mapstruct.org/documentation/stable/reference/html/ "MapStruct Reference Guide"
[autovalue]: https://github.com/google/auto/tree/master/value "AutoValue - Immutable value-type code generation for Java 1.6+."
[freebuilder]: https://freebuilder.inferred.org/ "Automatic generation of the Builder pattern for Java"

