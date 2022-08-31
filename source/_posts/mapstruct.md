---
title: MapStruct - Java bean 映射，简单的方法！
date: 2022.09.01 
tags: Java，Filter
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 应用程序通常需要在不同的对象模型（例如实体和 DTO）之间进行映射，如：在前后台传输过程中，持久层定义的实体类 经常需要映射到 其它的对象模型 进行相互转换。
---


应用程序通常需要在不同的对象模型（例如实体和 DTO）之间进行映射，
如：在前后台传输过程中，持久层定义的实体类 经常需要映射到 其它的对象模型 进行相互转换。

![](https://wyiyi.github.io/amber/contents/logo/mapstruct.png)

## MapStruct

MapStruct 是一个代码生成器，在不同的对象模型（例如 实体 和 DTO）之间进行映射，它基于约定优于配置的方法，极大地简化了 Java bean 类型之间的映射实现。

MapStruct 是一个 Java注释处理器，用于生成类型安全、高性能和无依赖关系的 bean 映射代码。

与其他映射框架相比，MapStruct 在编译时生成 bean 映射，这确保了高性能，允许开发人员快速的反馈和彻底的错误检查。

## 引用
- [Maven](https://mapstruct.org/documentation/stable/reference/html/#_apache_maven) - pom.xml
```
<properties>
    <org.mapstruct.version>1.5.2.Final</org.mapstruct.version>
</properties>
...
<dependencies>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>
...
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

- [Gradle](https://mapstruct.org/documentation/stable/reference/html/#_gradle) - build.gradle
```
plugins {
    ...
    id "com.diffplug.eclipse.apt" version "3.26.0" // Only for Eclipse
}

dependencies {
    ...
    implementation "org.mapstruct:mapstruct:${mapstructVersion}"
    annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"

    // If you are using mapstruct in test code
    testAnnotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
}
```

## 冰山一角 - 基本映射
从基本开始，假设我们有一个代表汽车的类 `Car`（例如一个JPA 实体）和一个数据传输对象 `CarDto`（DTO）。 示例一：
```
public class Car {
    private String manufacturer;
    private int numberOfSeats;
    private int price;
```

```
public class CarDto {
    private String make;
    private int seatCount;
    private String price;
```

定义映射器的接口，只需在定义的接口上使用注解：`org.mapstruct.Mapper`，示例二：
```
@Mapper
public interface CarMapper {

    @Mapping(target = "manufacturer", source = "make")
    @Mapping(target = "seatCount", source = "numberOfSeats")
    CarDto carToCarDto(Car car);

    @Mapping(target = "fullName", source = "name")
    PersonDto personToPersonDto(Person person);
}
```

在生成的方法实现中，源类型（例如 Car）中的所有可读属性都将被复制到目标类型（例如 CarDto）中的相应属性中：
- 当一个属性与其对应的目标实体同名时，它将被隐式映射。
- 当一个属性在目标实体中具有不同的名称时，可以通过 @Mapping 注解指定其名称

@Mapper 使 MapStruct 代码生成器 CarMapper 在构建时创建接口的实现（在 target 目录下对应的路径下）。示例三：
```
// GENERATED CODE
public class CarMapperImpl implements CarMapper {

    @Override
    public CarDto carToCarDto(Car car) {
        if ( car == null ) {
            return null;
        }

        CarDto carDto = new CarDto();

        if ( car.getFeatures() != null ) {
            carDto.setFeatures( new ArrayList<String>( car.getFeatures() ) );
        }
        carDto.setManufacturer( car.getMake() );
        carDto.setSeatCount( car.getNumberOfSeats() );
        carDto.setDriver( personToPersonDto( car.getDriver() ) );
        carDto.setPrice( String.valueOf( car.getPrice() ) );
        if ( car.getCategory() != null ) {
            carDto.setCategory( car.getCategory().toString() );
        }
        carDto.setEngine( engineToEngineDto( car.getEngine() ) );

        return carDto;
    }

    @Override
    public PersonDto personToPersonDto(Person person) {
        //...
    }
}
```
编译后的代码，看起来像自己写的代码，特别是，这意味着通过普通的 getter/setter 调用而不是反射或类似方法将值从源复制到目标。

MapStruct 在许多情况下会自动处理类型转换。生成的代码考虑了通过指定的任何名称映射 @Mapping。

1、源对象 和 目标对象 不同的类型，也就是 [隐式类型转换：](https://mapstruct.org/documentation/stable/reference/html/#implicit-type-conversions)

目前自动应用以下转换：

- 在所有 Java 原始数据类型和它们对应的包装器类型之间，例如：int，Integer and boolean 等 Boolean。
  生成的代码是有 null 意识的，即当将包装器类型转换为相应的原始类型时，null 将执行检查。
- 在所有 Java 原始数字类型和包装类型之间，例如：int、long 或 byte、Integer。
- 在所有 Java 原始类型（包括它们的包装器）和 String 之间，例如：在 int 和 String 或 Boolean 和 String。
  java.text.DecimalFormat 可以指定被理解的格式字符串。
- 在 Enum类型 和 String 之间。
- 在大数字类型（java.math.BigInteger, java.math.BigDecimal）和 Java 原始类型（包括它们的包装器）以及字符串之间。
  java.text.DecimalFormat可以指定被理解的格式字符串。
- 在 java.util.Date/XMLGregorianCalendar 和 String 之间。java.text.SimpleDateFormat 可以通过选项指定格式字符串。
- ...

示例四：从 int 到 String 的转换
```
@Mapper
public interface CarMapper {

    @Mapping(source = "price", numberFormat = "$#.00")
    CarDto carToCarDto(Car car);

    @IterableMapping(numberFormat = "$#.00")
    List<String> prices(List<Integer> prices);
}
```
示例五：从 BigDecimal 到 String 的转换
```
@Mapper
public interface CarMapper {

    @Mapping(source = "power", numberFormat = "#.##E0")
    CarDto carToCarDto(Car car);

}
```
示例六：从 Date 到 String 的转换
```
@Mapper
public interface CarMapper {

    @Mapping(source = "manufacturingDate", dateFormat = "dd.MM.yyyy")
    CarDto carToCarDto(Car car);

    @IterableMapping(dateFormat = "dd.MM.yyyy")
    List<String> stringListToDateList(List<Date> dates);
}
```

2、源对象 包含其他对象 与 目标对象 的转换，也就是 [映射对象引用：](https://mapstruct.org/documentation/stable/reference/html/#mapping-object-references)

通常，一个对象不仅具有原始属性，而且还引用其他对象。
例如，Car 类可以包含对 Person 应该映射到该类引用的对象的对象（代表汽车的驾驶员）的 PersonDto 引用 CarDto。

在这种情况下，只需为引用的对象类型定义一个映射方法，示例七：
```
@Mapper
public interface CarMapper {

    CarDto carToCarDto(Car car);

    PersonDto personToPersonDto(Person person);
}
```
carToCarDto() 方法的生成代码将调用 personToPersonDto() 用于映射 driver属性 的方法，
而生成的实现用于 personToPersonDto() 执行人员对象的映射。

在生成映射方法的实现时，MapStruct 将为源对象和目标对象中的每个属性对应用以下例程：

- 如果源和目标属性具有相同的类型，则该值将**直接**从源复制到目标。如果该属性是一个集合（例如 a List），则该集合的副本将被设置到目标属性中。
- 如果源属性类型和目标属性类型不同，请检查是否存在**其他映射方法**，其参数类型为源属性类型，返回类型为目标属性类型。如果存在这样的方法，它将在生成的映射实现中调用。
- 如果不存在这样的方法，MapStruct 将查看属性的源和目标类型的**内置转换是否存在**。如果是这种情况，生成的映射代码将应用此转换。
- 如果不存在这样的方法，MapStruct 将应用**复杂**的转换：
  a.映射方法，映射方法映射的结果，像这样：target = method1( method2( source ) )

b.内置转换，通过映射方法映射的结果，如下所示：target = method( conversion( source ) )

c.映射方法，内置转换映射的结果，如下所示：target = conversion( method( source ) )

- 如果没有找到这样的方法，MapStruct 将尝试生成一个自动子映射方法，该方法将在源属性和目标属性之间进行映射。
- 如果 MapStruct 无法创建基于名称的映射方法，则会在构建时引发错误，指示不可映射的属性及其路径。

映射控件 ( MappingControl) 可以在所有级别 ( @MapperConfig, @Mapper, @BeanMapping, @Mapping) 上定义。

3、源对象 和 目标对象 同为 集合类型 进行转换，也就是 [映射集合：](https://mapstruct.org/documentation/stable/reference/html/#mapping-collections)

集合类型（List Set 等）的映射与映射 bean 类型的方式相同，即通过在映射器接口中定义具有所需源和目标类型的映射方法。
MapStruct 支持 Java 集合框架中的各种可迭代类型。示例八：
```
@Mapper
public interface CarMapper {

    Set<String> integerSetToStringSet(Set<Integer> integers);

    List<CarDto> carsToCarDtos(List<Car> cars);

    CarDto carToCarDto(Car car);
}
```
生成的实现为每个元素执行从到 integerSetStringSet 的转换，而生成的方法为每个包含的元素调用方法，
如示例九所示：
```
//GENERATED CODE
@Override
public Set<String> integerSetToStringSet(Set<Integer> integers) {
    if ( integers == null ) {
        return null;
    }

    Set<String> set = new LinkedHashSet<String>();

    for ( Integer integer : integers ) {
        set.add( String.valueOf( integer ) );
    }

    return set;
}

@Override
public List<CarDto> carsToCarDtos(List<Car> cars) {
    if ( cars == null ) {
        return null;
    }

    List<CarDto> list = new ArrayList<CarDto>();

    for ( Car car : cars ) {
        list.add( carToCarDto( car ) );
    }

    return list;
}
```

4、源枚举 中的每个常量都 映射 到 目标枚举类型 中同名的常量。[映射值：](https://mapstruct.org/documentation/stable/reference/html/#_mapping_enum_to_enum_types)

@ValueMapping 如果需要，可以在注释的帮助下 将源枚举 中的 常量 映射到 具有另一个名称的常量。
源枚举中的几个常量可以映射到目标类型中的同一个常量。示例十：
```
@Mapper
public interface OrderMapper {

    OrderMapper INSTANCE = Mappers.getMapper( OrderMapper.class );

    @ValueMappings({
        @ValueMapping(target = "SPECIAL", source = "EXTRA"),
        @ValueMapping(target = "DEFAULT", source = "STANDARD"),
        @ValueMapping(target = "DEFAULT", source = "NORMAL")
    })
    ExternalOrderType orderTypeToExternalOrderType(OrderType orderType);
}
```
生成的代码。示例十一：
```
/ GENERATED CODE
public class OrderMapperImpl implements OrderMapper {

    @Override
    public ExternalOrderType orderTypeToExternalOrderType(OrderType orderType) {
        if ( orderType == null ) {
            return null;
        }

        ExternalOrderType externalOrderType_;

        switch ( orderType ) {
            case EXTRA: externalOrderType_ = ExternalOrderType.SPECIAL;
            break;
            case STANDARD: externalOrderType_ = ExternalOrderType.DEFAULT;
            break;
            case NORMAL: externalOrderType_ = ExternalOrderType.DEFAULT;
            break;
            case RETAIL: externalOrderType_ = ExternalOrderType.RETAIL;
            break;
            case B2B: externalOrderType_ = ExternalOrderType.B2B;
            break;
            default: throw new IllegalArgumentException( "Unexpected enum constant: " + orderType );
        }

        return externalOrderType_;
    }
}
```

5、[多个源参数的映射：](https://mapstruct.org/documentation/stable/reference/html/#mappings-with-several-source-parameters)
例如：将几个实体 组合成 一个数据传输对象。示例十二：
````
@Mapper
public interface AddressMapper {

    @Mapping(target = "description", source = "person.description")
    @Mapping(target = "houseNumber", source = "address.houseNo")
    DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Address address);
}
````
所示的映射方法采用两个源参数并返回一个组合的目标对象。与单参数映射方法一样，属性按名称映射。

@Mapping 如果多个源对象定义具有 相同名称的属性，则必须使用注释指定从中检索属性的源参数，如示例中的 description属性 所示。
对于在给定源对象中只存在一次的属性，可以选择指定源参数的名称，因为它可以自动确定。

6、[条件映射：](https://mapstruct.org/documentation/stable/reference/html/#conditional-mapping)String 属性不为 null 且不为空时 映射。
增加注解 org.mapstruct.Condition 的 @Condition 返回的方法 boolean 类型。
````
@Mapper
public interface CarMapper {

    CarDto carToCarDto(Car car);

    @Condition
    default boolean isNotEmpty(String value) {
        return value != null && !value.isEmpty();
    }
}
````

## 最后

Mapstruct 提供了大量的功能和配置，使我们能够以以安全优雅、简单快捷的方式创建从简单到复杂的映射器，减少转换代码。
本文中所介绍到的只是基础常见用法，还有很多强大的功能文中没有提到，想要探索更多、更详细的使用方式可以参考 [官方文档。](https://mapstruct.org/documentation/stable/reference/html/)