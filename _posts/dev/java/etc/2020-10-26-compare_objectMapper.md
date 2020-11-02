---
title: "Object Mapper 성능 비교"
categories: "Java"
tags:
  - Object Mapper
  - Okira
  - ModelMapper
  - MapStruct
---

# Object Mapper와의 만남  
세상에 수많은 라이브러리들이 존재한다.  
'이런게 불편한데.. 이런걸 구현해주는 라이브러리가 없을까?'  
**있다.** 우리가 생각하기에 '필요한', '있으면 좋겠다' 싶은 라이브러리들은 대부분 존재한다.  
혹자는 라이브러리를 가져다 쓸게 아니라, 직접 구현해야하지 않는가 라고 할 수 있지만, 훌륭한 기능을 구현하는 능력만큼 완성도 높은 라이브러리를 잘 활용할 줄 아는 것 또한 개발자의 중요한 몫이라 생각한다.  

그런데 전제조건이 있다. **라이브러리를 제대로 이해하고 잘 활용해야 한다는 것이다.**  

최근 이렇게 잘 알지 못한채 라이브러리를 도입하며 서비스에 큰 문제를 일으켰던 악몽이자 좋은 경험을 했다.  
바로 `Object Mapping`라이브러리이다.

Object Mapping 라이브러리는 일반적으로 Entity -> DTO 혹은 DTO -> Enttiy 등 `Obect to Object`의 맵핑을 위하여 사용한다.  
Object Mapping을 도와주는 여러 라이브러리들이 있었지만, 그 중 직관적으로 이해가되는 api를 제공해주는 `Model Mapper`를 적용했다.  

# ModelMapper 악몽의 시작  
배포 후 BMT를 진행했다. 기존에 서비스 중이던 API였고, 간단한 로직추가와 레거시 코드를 걷어내고 JPA를 적용하면서 리팩토링을 했기 때문에 큰 차이가 없을 것으로 기대하며 가벼운 마음으로 BMT를 시작했다.

![](/assets/images/study/dev/2020/10/1026_pinpoint_graph.png)
<span style="color:#c2c9d4; font-size: 20px;"> 배포 후 발생한 전봇대들.. </span>
{: .text-center }

적용 후 내 눈을 의심할 수 밖에 없었다. 기존에 아주 간헐적으로 튀는 현상은 있었지만 대체로 빠른 응답속도로 안정적인 그래프 모습이지만,  
배포 후 짧은 시간에 **여러개의 기둥들이 생기면서 그래프의 여백을 허용하지 않았다.**

![](/assets/images/study/dev/2020/10/1026_compute_resources_usage.png)
<span style="color:#c2c9d4; font-size: 20px;"> 시스템 리소스 현황 </span>
{: .text-center }  

실제로 시스템 리소스를 모니터링했을때, 메모리가 줄어들지 않고 계속해서 증가하는 것으로 보아 `메모리 누수(memory leak)`가 발생하고 있음을 파악할 수 있었다.

- RDB에 간단한 I/O만 하고있는데 왜 메모리 누수가 발생할까?

그리고 핀포인트의 도움을 받아 우선 원인이 되는 구간을 파악할 수 있었다.

![](/assets/images/study/dev/2020/10/1026_method_bottleNeck.png)

이미지상의 특정 메서드에서 병목이 발생하고 있었다.  
스크린샷을 추가해놓진 못했지만, 이후 Heap Dump를 통해서 원인을 파악할 수 있었다.  

`ModelMapper`를 통해서 DTO->Entity간 맵핑을 했는데, API를 호출할때마다 `ModelMapper`에서 새로운 인스턴스를 생성하며 오용되고 있었던 것이다.  
문제는 사용이 끝난 객체가 GC를 통해 소멸되지 않아 메모리 누수가 발생된 것으로 이해했다.  
(명확히 왜 객체가 소멸되지 않았는지는 파악하지 못했다ㅠㅠ)

사실 원인을 파악하면 문제해결은 의외로 간단하다. ModelMapper 제거하여 문제를 해결했다.  
추후에 왜 ModelMapper를 제거했는지 거론되겠지만, 우선은 트러블슈팅을 이야기 하고자 한 것이 아니다.  
이번 이슈를 통해서 무분별하게 사용한 라이브러리가 얼마나 예상치 못한 문제를 일으키는지를 통감하며 `Object Mapper`에 관심을 갖게 되었다.

# Object Mapper 비교

꽤 다양한 객체간 맵핑 기술이 있다. *(Dozer, JMapper, Selma, ReMap, ModelMapping, Orika, MapStruct 등)*  
이번 포스팅은 이 중 문제를 일으켰던 `ModelMapper`를 기준으로 `Orika` 그리고 최근 가장 선호도가 높은(?) `MapStruct`의 비교를 해보려 한다.  
코드레벨에서 각 기술들이 어떻게 구현되었는지에 대한 비교는 아니다.  
각 기술의 **포포몬쓰**만 비교해 보았다.

## Object Mapper 구현하기

### gradle 의존성 추가  

```bash
    compile group: 'ma.glasnost.orika', name: 'orika-core', version: '1.5.4'

    compile group: 'org.mapstruct', name: 'mapstruct', version: '1.4.1.Final'
    compile group: 'org.mapstruct', name: 'mapstruct-jdk8', version: '1.4.1.Final'
    compile group: 'org.mapstruct', name: 'mapstruct-processor', version: '1.4.1.Final'

    compile group: 'org.modelmapper', name: 'modelmapper', version: '2.3.8'

    annotationProcessor 'org.projectlombok:lombok'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.1.Final'
```

### DTO & Enttiy 객체 생성  

~~~java
  @Setter
@Getter
@NoArgsConstructor
public class OrderDto {
    private long serialNo;
    private long orderId;
    private String orderingUserName;
    private int status;
    private List<Product> orderProducts;
    private LocalDateTime orderDate;
    private LocalDateTime deliveryDate;

    public static List<OrderDto> getOrderDtoList(int size){
        List<OrderDto> orderList = new ArrayList<>();

        for (int i = 0; i < size; i++) {
            OrderDto orderDto = new OrderDto();
            orderDto.setSerialNo(new Random().nextLong());
            orderDto.setOrderId(new Random().nextLong());
            orderDto.setOrderingUserName("betterFLY_Test");
            orderDto.setStatus(new Random().nextInt(5));
            orderDto.setOrderDate(LocalDateTime.now());
            orderDto.setOrderDate(LocalDateTime.now().plusDays(new Random().nextInt(10)));


            orderDto.setOrderProducts(generateProduct());
            orderList.add(orderDto);
        }

        return orderList;
    }

    private static List<Product> generateProduct(){
        List<Product> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Product p = new Product();
            p.setName("p_"+i);
            p.setPrice((38890L * new Random().nextInt(5)));
            p.setProductDate(LocalDateTime.now().minusDays(new Random().nextInt(20)));

            list.add(p);
        }

        return list;
    }
}

@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class OrderEntity {
    private long serialNo;
    private long orderId;
    private String orderingUserName;
    private int orderStatus;
    private List<Product> orderProducts;
    private LocalDateTime orderDate;
    private LocalDateTime deliveryDate;
}

~~~

DTO에 구현되어 있는 메서드는 추후에 테스트를 위한 객체를 생성하기 위한 부분이기 때문에 크게 내용을 분석할 필요는 없어 보인다.

### Object Mapper 구현하기  

~~~java

// 각 mapper에서 구현할 인터페이스
public interface Converter {
    OrderEntity convertDtoToEntity(OrderDto orderDto);
}

// Orikia
public class OrikaConverterImpl implements Converter {

    private MapperFacade mapperFacade;

    public OrikaConverterImpl() {
        MapperFactory mapperFactory = new DefaultMapperFactory
                .Builder().build();

        mapperFactory.classMap(OrderDto.class, OrderEntity.class)
                .field("status", "orderStatus").byDefault().register();
        mapperFacade = mapperFactory.getMapperFacade();
    }

    @Override
    public OrderEntity convertDtoToEntity(OrderDto orderDto) {
        return mapperFacade.map(orderDto, OrderEntity.class);

    }
}

// ModelMapper
public class ModelMapperConverterImpl implements Converter {
    private ModelMapper modelMapper;

    public ModelMapperConverterImpl() {
        modelMapper = new ModelMapper();
        PropertyMap<OrderDto, OrderEntity> entityMap = new PropertyMap<OrderDto, OrderEntity>() {
            protected void configure() {
                map().setOrderStatus(source.getStatus());
            }
        };
        modelMapper.addMappings(entityMap);
    }

    @Override
    public OrderEntity convertDtoToEntity(OrderDto orderDto) {

        return modelMapper.map(orderDto, OrderEntity.class);
    }
}


// MapStruct
@Mapper
public interface MapStructConverter extends Converter {
    MapStructConverter INSTANCE = Mappers.getMapper(MapStructConverter.class);

    @Mapping(source = "status", target="orderStatus")
    @Override
    OrderEntity convertDtoToEntity(OrderDto orderDto);
}
~~~

`Orika`, `ModelMapper`, `MapStruct` 3가지 Mapping 기술을 구현했다.
모든 필드명이 같은 것을 맵핑하는 것은 심심할 것 같아서 OrderDTO에서 `status`라는 변수명이 Entity에서 `orderStatus`로 맵핑되는 부분만 별도로 맵핑처리 해주었다.

이제 이 맵핑 클래스들을 실제로 실행해보며, 소요시간을 체크해보면 된다.

```bash
// 시작 시간 timestamp
// mapping logic
// 종료시간 - 시작시간 = 소요시간 확인
```

위와 같은 흐름이기 때문에 간단하게 추상클래스를 통해서 구현해보았다.

~~~java

@Slf4j
public abstract class AbstractMappingService {
    public void execute(List<OrderDto> list){
        long start = System.currentTimeMillis();

        converter(list);

        double secDiffTime = (System.currentTimeMillis() - start)/1000.0;
        log.info("실행시간(s), {}", secDiffTime);
    }

    abstract List<OrderEntity> converter(List<OrderDto> list);
}

/*
  Mapper 실행 구하기
*/

public class OrikaServiceImpl extends AbstractMappingService{
    private OrikaConverterImpl orikaConverter;

    public OrikaServiceImpl(){
        orikaConverter = new OrikaConverterImpl();
    }

    @Override
    public List<OrderEntity> converter(List<OrderDto> list) {
        return list.stream()
                .map(orikaConverter::convertDtoToEntity)
                .collect(Collectors.toList());
    }
}

public class ModelMapperServiceImpl extends AbstractMappingService{
    private ModelMapperConverterImpl modelMapperConverter;

    public ModelMapperServiceImpl(){
        modelMapperConverter = new ModelMapperConverterImpl();
    }

    @Override
    public List<OrderEntity> converter(List<OrderDto> list) {
        return list.stream()
                .map(modelMapperConverter::convertDtoToEntity)
                .collect(Collectors.toList());
    }
}

public class MapStructServiceImpl extends AbstractMappingService{

    @Override
    public List<OrderEntity> converter(List<OrderDto> list) {
        return list.stream()
                .map(MapStructConverter.INSTANCE::convertDtoToEntity)
                .collect(Collectors.toList());
    }
}
~~~

자 이제 실행하기 위한 모든 준비가 완료되었다.  
실행시간을 늘리기 위해서 100만 이상의 객체를 생성해서 테스트해보려고 하니까, JUnit을 통해서 실행할 경우 heap size가 낮아 실행이 되지 않아, Main메서드를 통해서 실행했다.

~~~java
public class Starter {
    public static void main(String[] args) {
        List<OrderDto> orderDtoList = OrderDto.getOrderDtoList(1_300_000);

//        AbstractMappingService mapStructService = new MapStructServiceImpl();
//        AbstractMappingService orikaService = new OrikaServiceImpl();
        AbstractMappingService mapperService = new ModelMapperServiceImpl();
        for (int i = 0; i <10 ; i++) {
//            mapStructService.execute(orderDtoList);
//            orikaService.execute(orderDtoList);
            mapperService.execute(orderDtoList);
        }
    }
}
~~~

130만건 정도를 실행하니까 조금 더 분별력있는 결과가 나왔다.

### 실행 결과 분석 및 결과  

![](/assets/images/study/dev/2020/10/1026_result_orika.jpeg)
![](/assets/images/study/dev/2020/10/1026_result_modelMapper.jpeg)
![](/assets/images/study/dev/2020/10/1026_result_mapStruct.jpeg)

130만건의 맵핑을 각각 10회씩 실행했다.  
훨씬 더 많은 케이스를 테스트해봤는데 오차의 범위가 크지 않아 1,2차의 케이스로 요약했다.  
실행 케이스마다 리소스 사용량도 대동소이했다.

비교가 의미 없을정도(?)로 `MapStruct`의 성능이 압도적이다.  
ModelMapper가 Orika보다는 조금 더 나은 실행속도를 보이고, CPU사용률도 비교적 안정적으로 사용하는 모습이다.(하지만 Memory를 많이 사용한다.)  

`ModelMapper`나 `Orika`에서 메모리를 많이 활용하고 성능이 저하되는 것은 **runtime 시점에 reflection**을 통해 맵핑을 하기 떄문에 맵핑 객체의 사이즈가 커질수록 선형적으로 메모리를 많이 사용하게 될 것이며 성능도 저하될 수 밖에 없다.(비효율)  

~~~java
  // ModelMapper에서 reflect를 활용
  public <S, D> void addConverter(Converter<S, D> converter) {
    Assert.notNull(converter, "converter");
    Class<?>[] typeArguments = TypeResolver.resolveRawArguments(Converter.class, converter.getClass());
    Assert.notNull(typeArguments, "Must declare source type argument <S> and destination type argument <D> for converter");
    config.typeMapStore.<S, D>getOrCreate(null, (Class<S>) typeArguments[0],
        (Class<D>) typeArguments[1], null, null, converter, engine);
  }

  // Orika에서 reflect 활용
  <S, D> MappingStrategy resolveMappingStrategy(final S sourceObject, final java.lang.reflect.Type sourceType,
            final java.lang.reflect.Type destinationType, boolean mapInPlace, final MappingContext context);
~~~

`MapStruct`의 성능이 우수한 이유는 Lombok과 같이 *annotation processor*를 통해서 **compile 시점**에 객체간 맵핑이 이루어지기 떄문에 매우 월등한 성능을 보일 수 있다.

# 정리

처음 큰 고민없이 사용했던 `ModelMapper`에서 발생한 문제가 나비효과가 되어 여기까지 오게 되었다.  
결과적으로 좋은 경험이었고, 나름 의미 있는 결과와 교훈이 되었다.  
단순히 *남들이 쓰기 때문에*, *레퍼런스가 많아서*, *좋다고하니까* 사용하는 기술이 아닌,  
직접 체득해보는 과정에서 앞으로 사용할 기술을 선정할 때 어떤 부분을 신경쓰는게 좋을지, 특징이 무엇인지를 파악해보는 습관을 기를 수 있는 기회가 된 것 같다.

---

*Reference*

- https://github.com/arey/java-object-mapper-benchmark
- https://www.baeldung.com/java-performance-mapping-frameworks
- https://mapstruct.org/
- https://github.com/modelmapper/modelmapper/issues/375