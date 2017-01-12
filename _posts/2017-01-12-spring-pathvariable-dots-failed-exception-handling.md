---
title: "Spring, @PathVariable, dots, and failed exception handling"
categories: spring
---

If you use Spring Boot to set up an endpoint using `@RequestMapping` with a `@PathVariable` at the end of the request URI, you might run into several quirks if this path variable happens to include a dot. For example, consider the following application:

```java
@RequestMapping(value = "/api/resource/{id}", method = RequestMethod.GET)
@ResponseBody
public Resource getResource(@PathVariable("id") final String id) throws ResourceNotFoundException {
  final Resource resource = resourceService.findById(id);
  if (resource == null) {
    throw new ResourceNotFoundException();
  }
  return resource;
}
```

This mapping returns a `Resource` object in the response body, which is serialized to JSON using Jackson2. Let's also assume that we handle the `ResourceNotFoundException` using an `@ExceptionHandler`.

```java
@ExceptionHandler(ResourceNotFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
@ResponseBody
public ErrorResponse handleResourceNotFoundException(final ResourceNotFoundException exception) {
  return new ErrorResponse();
}
```

The `ErrorResponse` object is similarly serialized to JSON.

If you perform a GET request to `/api/resource/this.is.my.id` you may notice several things. For one, the `id` parameter is equal to `"this.is.my"`. The last dot and everything after were truncated! Secondly, if the resource `this.is.my` does not exist, a 500 is returned, not the 404 with the expected JSON object. What's going on?

A quick search on stack overflow will reveal a fix for the first problem: a regex pattern on the path variable.

```java
@RequestMapping(value = "/api/resource/{id:.+}", method = RequestMethod.GET)
```

This hack works, but it is far from ideal, as we would have to use it on every endpoint. How can we fix this globally? Spring thinks the characters after the last dot are a file extension, and removes them for you. Since 4.0.1, we can configure this ourselves using the [`PathMatchConfigurer`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/PathMatchConfigurer.html). We can turn off suffix pattern matching entirely, or only allow of for extensions that we explicitely register ourselves. Let's turn it off entirely:

```java
@Configuration
public static class WebConfig extends WebMvcConfigurerAdapter {
  
  @Override
  public void configurePathMatch(final PathMatcherConfigurer configurer) {
    configurer.setUseSuffixPatternMatch(false);
  }
}
```

That's much better. However, keep in mind that a method mapped to `/users` will no longer match to `/users.*`. In my opinion, that behaviour is unnecessary anyway.

We're still stuck with the second problem; requesting `script.js` returns 500. A bit of digging reveals the `handleResourceNotFoundException` is being called, and an `ErrorResponse` object is returned, but a `HttpMediaTypeNotAcceptableException` is being thrown somewhere in the Spring framework. This indicates Spring is unable to generate a response that is acceptable by the client. However, the client didn't specify the response format. What is going on?

It turns out that Spring is overriding the response format based on the "file format" of the path. In this case, given `script.js`, Spring assumes it needs to return a javascript file. It cannot convert the `ErrorResponse` object to the `application/javascript` content type, so it throws an exception.

We can fix this using the [`ContentNegotiatorConfigurer`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/config/annotation/ContentNegotiationConfigurer.html). In the same `WebConfig` class, we implement the following override:

```java
@Override
public void configureContentNegotiation(final ContentNegotiationConfigurer configurer) {
  configurer.favorPathExtension(false);
}
```

Viola, with two configuration options the mapping works as expected.
