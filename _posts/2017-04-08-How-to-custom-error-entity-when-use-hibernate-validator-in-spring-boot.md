---
layout: post
title:  "How to custom error entity when use hibernate validator in spring boot?"
date:   2017-04-08 20:10:00 +0800
categories: [java, springboot, codefights]
---

How to customized error message when use Hibernate Validate in spring boot/mvc?

**Solution:**

* Project struture
```
├── src
│   └── main
│       └── resources
│           ├── application.yml
│           ├── i18n
│               └── message_zh_CN.properties
│               └── message_en.properties
```

* Initial the message file to system.
```java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.validation.Validator;
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.i18n.CookieLocaleResolver;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;

import java.util.Locale;

@Configuration
@EnableWebMvc
@ComponentScan
public class MvcWebConfig extends WebMvcConfigurerAdapter {

// other configurations may be here ...

    @Bean("messageSource")
    public ReloadableResourceBundleMessageSource messageSource() {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setDefaultEncoding("UTF-8");
        messageSource.setCacheSeconds(1);
        messageSource.setBasename("classpath:/i18n/message");
        return messageSource;
    }

    @Override
    public Validator getValidator() {
       LocalValidatorFactoryBean validator = new LocalValidatorFactoryBean();
       validator.setValidationMessageSource(messageSource());
       return validator;
    }

    // for swagger-ui
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        super.addResourceHandlers(registry);
    }

    @Bean
    public LocaleResolver localeResolver(){
        CookieLocaleResolver resolver = new CookieLocaleResolver();
        resolver.setDefaultLocale(new Locale("zh_CN"));
        resolver.setCookieName("localeCookies");
        resolver.setCookieMaxAge(4800);
        return resolver;
    }
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        LocaleChangeInterceptor interceptor = new LocaleChangeInterceptor();
        interceptor.setParamName("lang");
        registry.addInterceptor(interceptor);
    }
}
```

/i18n/message file content
```
message_zh_CN.properties
PageRequest.pageSize.NotNull=pageSize不能为空
PageRequest.pageIndex.NotNull=pageIndex不能为空

---
message_en.properties
PageRequest.pageSize.NotNull=pageSize is not null
PageRequest.pageIndex.NotNull=pageIndex is not null
```

How to valid request params use model.

```
java
@RestController
@RequestMapping(value = "/v1/class")
public class ClassController {

    @Autowired
    private ClassService classService;

    @RequestMapping(value = "{clientSn}/lessons/before", method = RequestMethod.GET)
    public ResponseEntity findLessonBeforeClass(@PathVariable("clientSn") Integer clientSn, @Valid @ModelAttribute ClassLessonPageRequest request) {

        Page<ClassLessonResponse> page = classService.findLessonByPage(clientSn, request, false);
        return ResponseEntity.ok(page);
    }

}

#Entity
public class PageRequest {
    
    @NotNull(message = "{PageRequest.pageSize.NotNull}")
    private Integer pageSize;
    @NotNull(message = "{PageRequest.pageIndex.NotNull}")
    private Integer pageIndex;

    public Integer getPageSize() {
        return pageSize;
    }

    public void setPageSize(Integer pageSize) {
        this.pageSize = pageSize;
    }

    public Integer getPageIndex() {
        return pageIndex;
    }

    public void setPageIndex(Integer pageIndex) {
        this.pageIndex = pageIndex;
    }
}

#Entity
public class ClassLessonPageRequest extend PageRequest {
    //...
}
```

How to show different message depend the user's language?

```
java
import com.vipabc.vjr.vbs.common.vo.FieldException;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.BindException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

/**
 * Created by Whoami on 2017/4/8.
 */
@ControllerAdvice
public class RestfulResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleBindException(BindException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        FieldException error = new FieldException(ex.getBindingResult().getAllErrors().get(0).getObjectName(), ex.getBindingResult().getFieldErrors().get(0).getDefaultMessage());
        return super.handleExceptionInternal(ex, error, headers, status, request);
    }
}

#Entity
public class FieldException {
    private String name;
    private String message;

    public FieldException() {
    }

    public FieldException(String name, String message) {
        this.name = name;
        this.message = message;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

**Example:**

* output
```
Response Body
{
  "name": "classLessonPageRequest",
  "message": "pageIndex is not null"
}
Response Code
400
```

Just run it.

Done.