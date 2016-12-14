---
layout: post
title:  "Spring mvc restful api unite exception handler."
date:   2016-12-14 15:17:00 +0800
categories: [spring, codefights]
---

When we use spring mvc restful api, we needn't return the error msg up to controller and response to client, We just need to throw a exception, Just all.

**Usage**

* For 404, throw new NotFoundException(), three choice.

```java
1. throw new NotFoundException(ErrorCode.NOT_FOUND)
2. throw new NotFoundException(ErrorCode.NOT_FOUND, "NOT_FOUND")
3. throw new NotFoundException(ErrorCode.NOT_FOUND, "NOT_FOUND", ImmutableMap.of("NOT_FOUND", "Sorry, Not found!"))
```
* For 400, throw new BadRequestException().

```java
1. throw new BadRequestException(ErrorCode.CAMPAIGN_DATE_INVALID)
2. throw new BadRequestException(ErrorCode.CAMPAIGN_DATE_INVALID, "CAMPAIGN_DATE_INVALID")
3. throw new BadRequestException(ErrorCode.CAMPAIGN_DATE_INVALID, "CAMPAIGN DATE INVALID", ImmutableMap.of("CAMPAIGN_DATE_INVALID", "Sorry, Campaign date invalid!"))
```
* You can add UnauthorizedException with HttpStatus.UNAUTHORIZED、 ForbiddenException with HttpStatus.FORBIDDEN, something like that.

**Solution:**

This is base request exception class for exception detect.

```java
import org.springframework.http.HttpStatus;

import java.util.Map;

/**
 * Created by Whoami on 2016/12/10.
 */
public abstract class RequestException extends RuntimeException {

    private HttpStatus httpStatus;
    private Integer errorCode;
    private Map<String, Object> errorMsgs;

    public RequestException(HttpStatus httpStatus, Integer errorCode) {
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public RequestException(HttpStatus httpStatus, Integer errorCode, String message) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
    }

    public RequestException(HttpStatus httpStatus, Integer errorCode, String message, Map<String, Object> errorMsgs) {
        super(message);
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
        this.errorMsgs = errorMsgs;
    }

    public RequestException(HttpStatus httpStatus, Integer errorCode, Map<String, Object> errorMsgs) {
        this.httpStatus = httpStatus;
        this.errorCode = errorCode;
        this.errorMsgs = errorMsgs;
    }

    public HttpStatus getHttpStatus() {
        return httpStatus;
    }

    public void setHttpStatus(HttpStatus httpStatus) {
        this.httpStatus = httpStatus;
    }

    public Integer getErrorCode() {
        return errorCode;
    }

    public void setErrorCode(Integer errorCode) {
        this.errorCode = errorCode;
    }

    public Map<String, Object> getErrorMsgs() {
        return errorMsgs;
    }

    public void setErrorMsgs(Map<String, Object> errorMsgs) {
        this.errorMsgs = errorMsgs;
    }
}
```

This is the NotFoundException class.

```java
import org.springframework.http.HttpStatus;

import java.util.Map;

public class NotFoundException extends RequestException {

    public NotFoundException(Integer errorCode, String message) {
        super(HttpStatus.NOT_FOUND, errorCode, message);
    }

    public NotFoundException(Integer errorCode, Map<String, Object> errorMsgs) {
        super(HttpStatus.NOT_FOUND, errorCode, errorMsgs);
    }

    public NotFoundException(Integer errorCode, String message, Map<String, Object> errorMsgs) {
        super(HttpStatus.NOT_FOUND, errorCode, message, errorMsgs);
    }
}
```

This is the BadRequestException class.

```java
import org.springframework.http.HttpStatus;

import java.util.Map;

public class BadRequestException extends RequestException {

    public BadRequestException(Integer errorCode, String message) {
        super(HttpStatus.BAD_REQUEST, errorCode, message);
    }

    public BadRequestException(Integer errorCode, Map<String, Object> errorMsgs) {
        super(HttpStatus.BAD_REQUEST, errorCode, errorMsgs);
    }

    public BadRequestException(Integer errorCode, String message, Map<String, Object> errorMsgs) {
        super(HttpStatus.BAD_REQUEST, errorCode, message, errorMsgs);
    }
}
```

the ErrorEntity class

```java
import java.util.HashMap;
import java.util.Map;

public class JsonInfo {

    private boolean success = true;
    
    private String message = "";
    
    private int error_code;
    
    private Map<String,Object> results = new HashMap<String,Object>();
    
    private JsonInfo(boolean _success){
        this.success = _success;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
    
    public Map<String, Object> getResults() {
        return results;
    }

    public JsonInfo setResults(Map<String, Object> results) {
        this.results = results;
        return this;
    }

    public static JsonInfo fail(){
        return new JsonInfo(false);
    }
    
    public static JsonInfo success(){
        return new JsonInfo(true);
    }
    
    public JsonInfo message(String message){
        setMessage(message);
        return this;
    }
    
    public JsonInfo addResult(String key,Object value){
        results.put(key, value);
        return this;
    }

    public int getError_code() {
        return error_code;
    }

    public JsonInfo setError_code(int error_code) {
        this.error_code = error_code;
        return this;
    }

    public JsonInfo errorCode(int i) {
        setError_code(i);
        return this;
    }

}
```

The leading role is coming.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

@ControllerAdvice
public class RestResponseEntityExceptionHandler extends
        ResponseEntityExceptionHandler {
    private Logger logger = LoggerFactory.getLogger(getClass());

    @ExceptionHandler(Except

        ion.class)
    protected ResponseEntity<Object> handleExceptions(Exception ex,
                                                      WebRequest request) {

        HttpStatus status = HttpStatus.OK;
        HttpHeaders headers = new HttpHeaders();
        String bodyOfResponse = "";

        //some object with error info. change use you own error entity.
        JsonInfo jsonInfo = JsonInfo.fail();
        if (ex instanceof RequestException) {
            status = ((RequestException) ex).getHttpStatus();
            jsonInfo.setError_code(((RequestException) ex).getErrorCode());
            jsonInfo.setMessage(ex.getMessage());
            jsonInfo.setResults(((RequestException) ex).getErrorMsgs());
        } else {
            status = HttpStatus.INTERNAL_SERVER_ERROR;
            logger.error(ex.getMessage(), ex);
            jsonInfo.setMessage(ex.getMessage());
        }
        //some tools you can serialize object to json.
        bodyOfResponse = JsonHelper.toJsonString(jsonInfo);

        headers.add("Content-Type", "application/json;charset=UTF-8");

        return handleExceptionInternal(ex, bodyOfResponse, headers, status, request);
    }
}
```

**Example:**

* you can throw it anywhere when you need response it to client.

```java
@RequestMapping(value = "/test/exception", method = RequestMethod.GET)
    public void test() {
        throw new BadRequestException(ErrorCode.SIGN_ERROR, "SIGN_ERROR", ImmutableMap.of("SIGN_ERROR", "签名错误!"));
    }
```

**Output:**

```json
{
  "success": false,
  "message": "SIGN_ERROR",
  "error_code": 34002,
  "results": {
    "SIGN_ERROR": "签名错误!"
  }
}
```
Done!