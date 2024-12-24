# Spring Maven Uygulamasında Validation Kullanımı
## Validasyon Bağımlılığını eklenmesi 
Maven Projesi için `pom.xml` dosyasına aşağıdaki bağımlılığı eklemeliyiz.
```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
## Modellere Validasyon eklenmesi
Aşağıda kendi projemde kullandığım örnek bir kaç anotasyon bulunmaktadır.
```java
public class RegisterRequest {
    @Size(min = 4, max = 25, message = "username must be between 4 and 25 chars")
    @NotBlank(message = "Username Cannot be blank")
    private String username;

    @Email(message = "Invalid Email format")
    @NotBlank(message = "Email cannot be blank")
    private String email;

    @NotBlank(message = "Password cannot be blank")
    @Size(min = 6, max = 20, message = "Password must be between 6 and 20 characters")
    private String password;
}
```

## Controllerda Valid kontrölü yapılması
Aşağıda gördüğünüz üzere validasyon uyguladığımız DTO için  `@Valid` anotasyonu ekledik
```java
    @PostMapping("/register")
    public ResponseEntity<String> register(@RequestBody @Valid RegisterRequest request) {
        try {
            String message = authService.register(request);
            return ResponseEntity.status(HttpStatus.CREATED).body(message);
        } catch (RuntimeException e) {
            return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
        }
    }
```

## Hata Yönetimi
Validasyondan çıkan istisna ve hataları yukarıda gördüğünüz controllerdan yönetemeyiz bunun yerini genel hatalarımız yönettiğimiz bir controller olması gerekir. 
GlobalExceptionHandler adında bir sınıf oluşturup burada genel hata yönetimi yapabiliriz. Kendi projemde kullandığım sınıf aşağıda mevcuttur.
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(JwtValidationException.class)
    public ResponseEntity<String> handleJwtValidationException(JwtValidationException ex) {
        System.err.println("JWT Validation Error: " + ex.getMessage());


        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Token süresi dolmuş. Lütfen yeniden giriş yapın.");
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
        );
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```
Bu sınıf içerisinde JWT ve Validasyon hatalarını yönetebileceğimiz iki adet fonksiyon bulunmaktadır.
`handleValidationExceptions` isimli fonksiyon ise validasyon için yeterlidir.

## Dönüş Tipi
Hataları ve model içerindekş alanları map ederek kullanıcaya hatalı olan alanları ve hata mesajlarını gösteririr.


