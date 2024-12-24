# ModelMapper'ı Spring Projesinde Kullanmak
## Bağımlılık eklenmesi (Maven)
bu bağımlılığı ekleyim eğer projemiz çalışıyorsa bağımlılıkları yükleyip tekrar başlatalım.
```
<dependency>
  <groupId>org.modelmapper</groupId>
  <artifactId>modelmapper</artifactId>
  <version>3.1.1</version>
</dependency>
```

## ModelMapperConfig
Bağımlılıklarınız doğru yüklendiyse bu aşamada hata almadan ilerleyebilirsizin.
Burada Spring Uygulamasına Mapper'ı Tanıtıyoruz.
```java
@Configuration
public class ModelMapperConfig {

    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

## Servis olarak tanımlanması
öncelikle Abstract Concrete Patterni kullanıoyrum projemde örneğimi de bu şekilde vermek istedim.

```java
public interface ModelMapperService {
    ModelMapper forResponse();
    ModelMapper forRequest();
}
```
### Method gövdeleri ise implemet eden sınıfta yani
```java
@Service
@AllArgsConstructor
public class ModelMapperManager implements  ModelMapperService{
    private ModelMapper modelMapper;


    @Override
    public ModelMapper forResponse() {

        //Gevşek bağlı bir mapper
        this.modelMapper.getConfiguration().setAmbiguityIgnored(true).setMatchingStrategy(MatchingStrategies.LOOSE);
        return this.modelMapper;
    }

    @Override
    public ModelMapper forRequest() {

        //Normal bağlı bir mapper
        this.modelMapper.getConfiguration().setAmbiguityIgnored(true).setMatchingStrategy(MatchingStrategies.STANDARD);
        return this.modelMapper;
    }

}
```

