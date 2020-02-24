本文主要讲解 SpringMVC 执行过程，并针对相关源码进行解析。

首先，让我们从 Spring MVC 的四大组件:**前端控制器（DispatcherServlet）、处理器映射器（HandlerMapping）、处理器适配器（HandlerAdapter）以及视图解析器（ViewResolver）** 的角度来看一下 Spring MVC 对用户请求的处理过程，过程如下图所示:

![](https://img-blog.csdnimg.cn/20200221001135320.png)

# SpringMVC 执行过程

1. 用户请求发送到**前端控制器 DispatcherServlet**。
2. 前端控制器 DispatcherServlet 接收到请求后，DispatcherServlet 会使用 HandlerMapping 来处理，**HandlerMapping 会查找到具体进行处理请求的 Handler 对象**。
3. HandlerMapping 找到对应的 Handler 之后，并不是返回一个 Handler 原始对象，而是一个 Handler 执行链（HandlerExecutionChain），在这个执行链中包括了拦截器和处理请求的 Handler。HandlerMapping 返回一个执行链给 DispatcherServlet。
4. DispatcherServlet 接收到执行链之后，会**调用 Handler 适配器去执行 Handler**。
5. Handler 适配器执行完成 Handler（也就是 Controller）之后会得到一个 ModelAndView，并返回给 DispatcherServlet。
6. DispatcherServlet 接收到 HandlerAdapter 返回的 ModelAndView 之后，会根据其中的视图名调用 ViewResolver。
7. **ViewResolver 根据逻辑视图名解析成一个真正的 View 视图**，并返回给 DispatcherServlet。
8. DispatcherServlet 接收到视图之后，会根据上面的 ModelAndView 中的 model 来进行视图中数据的填充，也就是所谓的**视图渲染**。
9. 渲染完成之后，DispatcherServlet 就可以将结果返回给用户了。

在了解了大概的执行过程后，让我们一起去从源码角度去深入探索（SpringMVC 版本为 5.2.3）：

我们先创建一个 Controller 以便进行 debug，内容如下：

```
@Controller
public class SpringMvcController {
    @RequestMapping("testSpringMvc")
    public String testSpringMvc(Map<String, String> map) {
        map.put("note", "在看转发二连");
        return "success";
    }
}
```

然后再创建一个 html 文件，采用 Thymeleaf 模版引擎，文件内容如下：

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>在看转发二连</title>
</head>
<body>
<h1 th:text="${note}"></h1>
</body>
</html>
```

好了，然后启动程序，让我们访问 `http://localhost:8080/testSpringMvc`，来一步一步探索 SpringMVC 的执行过程：

# 源码解析

首先当我们访问页面的时候，将会把请求发送到**前端控制器 DispatcherServlet**，DispatcherServlet 是一个 Servlet，我们知道在 Servlet 在处理一个请求的时候会交给 service 方法进行处理，这里也不例外，DispatcherServlet 继承了 FrameworkServlet，首先进入 FrameworkServlet 的 **service** 方法：

```
protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // 请求方法
    HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
    // 若方法为 PATCH 方法或为空则单独处理
    if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
        processRequest(request, response);
    }
    else {// 其他的请求类型的方法经由父类，也就是 HttpServlet 处理
        super.service(request, response);
    }
}
```

HttpServlet 中会根据请求类型的不同分别调用 doGet 或者 doPost 等方法，FrameworkServlet 中已经重写了这些方法，在这些方法中会调用 processRequest 进行处理，在 processRequest 中会调用 **doService** 方法，这个 doService 方法就是在 DispatcherServlet 中实现的。下面就看下 DispatcherServlet 中的 doService 方法的实现。

## DispatcherServlet 收到请求

DispatcherServlet 中的 doService方法：

```
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);
    // 给 request 中的属性做一份快照，以便能够恢复原始属性
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }
    // 如果没有配置本地化或者主题的处理器之类的，SpringMVC 会使用默认的配置文件，即 DispatcherServlet.properties
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    try {
        // 开始真正的处理
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // 恢复原始属性快照
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
    }
}
```

接下来 DispatcherServlet 开始真正的处理，让我们来看下 **doDispatch** 方法，首先会获取当前请求的 **Handler 执行链**，然后找到合适的 **HandlerAdapter**（此处为 RequestMappingHandlerAdapter），接着调用 RequestMappingHandlerAdapter 的 **handle** 方法，如下为 doDispatch 方法：

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        try {
            // 先检查是不是 Multipart 类型的，比如上传等；如果是 Multipart 类型的，则转换为 MultipartHttpServletRequest 类型
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // 获取当前请求的 Handler 执行链
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // 获取当前请求的 Handler 适配器
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // 对于 header 中 last-modified 的处理
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }

            // 遍历所有定义的 interceptor，执行 preHandle 方法
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // 实际调用 Handler 的地方
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }
            // 处理成默认视图名，也就是添加前缀和后缀等
            applyDefaultViewName(processedRequest, mv);
            // 拦截器postHandle方法进行处理
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 处理最后的结果，渲染之类的都在这里
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

## 查找对应的 Handler 对象

让我们去探索下是如何获取当前请求的 Handler 执行链，对应着这句代码 `mappedHandler = getHandler(processedRequest);`，看下 **DispatcherServlet 具体的 getHandler** 方法，该方法主要是遍历所有的 handlerMappings 进行处理，handlerMappings 是在启动的时候预先注册好的，handlerMappings 包含 RequestMappingHandlerMapping、BeanNameUrlHandlerMapping、RouterFunctionMapping、SimpleUrlHandlerMapping 以及 WelcomePageHandlerMapping，在循环中会调用 **AbstractHandlerMapping 类中的 getHandler 方法**来获取 Handler 执行链，若获取的 Handler 执行链不为 null，则返回当前请求的 Handler 执行链，DispatcherServlet 类的 getHandler 方法如下：

```
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        // 遍历所有的 handlerMappings 进行处理，handlerMappings 是在启动的时候预先注册好的
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```

在循环中，根据 `mapping.getHandler(request);`，继续往下看 **AbstractHandlerMapping 类中的 getHandler 方法**：

```
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    // 根据 request 获取 handler
    Object handler = getHandlerInternal(request);
    if (handler == null) {
        // 如果没有找到就使用默认的 handler
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // 如果 Handler 是 String，表明是一个 bean 名称，需要寻找对应 bean
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = obtainApplicationContext().getBean(handlerName);
    }
    // 封装 Handler 执行链
    return getHandlerExecutionChain(handler, request);
}
```

AbstractHandlerMapping 类中的 getHandler 方法中首先**根据 requrst 获取 handler**，主要是调用了 AbstractHandlerMethodMapping 类中的 **getHandlerInternal** 方法，该方法首先获取 request 中的 url，即 `/testSpringMvc`，用来匹配 handler 并封装成 HandlerMethod，然后根据 handlerMethod 中的 bean 来实例化 Handler 并返回。

```
protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
    // 获取 request 中的 url，用来匹配 handler
    String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
    request.setAttribute(LOOKUP_PATH, lookupPath);
    this.mappingRegistry.acquireReadLock();
    try {
        // 根据路径寻找 Handler，并封装成 HandlerMethod
        HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
        // 根据 handlerMethod 中的 bean 来实例化 Handler，并添加进 HandlerMethod
        return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
    }
    finally {
        this.mappingRegistry.releaseReadLock();
    }
}
```

接下来，我们看 **lookupHandlerMethod** 的逻辑，主要逻辑委托给了 **mappingRegistry** 这个成员变量来处理：

```
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
    List<Match> matches = new ArrayList<>();
    // 通过 lookupPath 属性中查找。如果找到了，就返回对应的RequestMappingInfo
    List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);
    if (directPathMatches != null) {
        // 如果匹配到了，检查其他属性是否符合要求，如请求方法，参数，header 等
        addMatchingMappings(directPathMatches, matches, request);
    }
    if (matches.isEmpty()) {
        // 没有直接匹配到，则遍历所有的处理方法进行通配符匹配
        addMatchingMappings(this.mappingRegistry.getMappings().keySet(), matches, request);
    }

    if (!matches.isEmpty()) {
        // 如果方法有多个匹配，不同的通配符等，则排序选择出最合适的一个
        Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
        matches.sort(comparator);
        Match bestMatch = matches.get(0);
        // 如果有多个匹配的，会找到第二个最合适的进行比较
        if (matches.size() > 1) {
            if (logger.isTraceEnabled()) {
                logger.trace(matches.size() + " matching mappings: " + matches);
            }
            if (CorsUtils.isPreFlightRequest(request)) {
                return PREFLIGHT_AMBIGUOUS_MATCH;
            }
            Match secondBestMatch = matches.get(1);
            if (comparator.compare(bestMatch, secondBestMatch) == 0) {
                Method m1 = bestMatch.handlerMethod.getMethod();
                Method m2 = secondBestMatch.handlerMethod.getMethod();
                String uri = request.getRequestURI();
                // 不能有相同的最优 Match
                throw new IllegalStateException(
                        "Ambiguous handler methods mapped for '" + uri + "': {" + m1 + ", " + m2 + "}");
            }
        }
        request.setAttribute(BEST_MATCHING_HANDLER_ATTRIBUTE, bestMatch.handlerMethod);
        // 设置 request 参数（RequestMappingHandlerMapping 对其进行了覆写）
        handleMatch(bestMatch.mapping, lookupPath, request);
        // 返回匹配的 url 的处理的方法
        return bestMatch.handlerMethod;
    }
    else {
        // 调用 RequestMappingHandlerMapping 类的 handleNoMatch 方法再匹配一次
        return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);
    }
}
```

通过上面的过程，我们就获取到了 Handler，就开始**封装执行链**了，就是将我们配置的拦截器加入到执行链中去，**getHandlerExecutionChain** 方法如下：

```
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
    // 如果当前 Handler 不是执行链类型，就使用一个新的执行链实例封装起来
    HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ? (HandlerExecutionChain) handler : new HandlerExecutionChain(handler));

    String lookupPath = this.urlPathHelper.getLookupPathForRequest(request, LOOKUP_PATH);
    // 遍历拦截器，找到跟当前 url 对应的，添加进执行链中去
    for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
        if (interceptor instanceof MappedInterceptor) {
            MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
            if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                chain.addInterceptor(mappedInterceptor.getInterceptor());
            }
        }
        else {
            chain.addInterceptor(interceptor);
        }
    }
    return chain;
}
```

到此为止，我们就获取了当前请求的 Handler 执行链，接下来看下是如何获取请求的 **Handler 适配器**，主要依靠 **DispatcherServlet 类的 getHandlerAdapter 方法**，该方法就是遍历所有的 HandlerAdapter，找到和当前 Handler 匹配的就返回，在这里匹配到的为 RequestMappingHandlerAdapter。DispatcherServlet 类的 getHandlerAdapter 方法如下：

```
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    if (this.handlerAdapters != null) {
        // 遍历所有的 HandlerAdapter，找到和当前 Handler 匹配的就返回
        for (HandlerAdapter adapter : this.handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
    }
    throw new ServletException("No adapter for handler [" + handler +
            "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

## HandlerAdapter 执行当前的 Handler


再获取完当前请求的 Handler 适配器后，接着进行**缓存处理**，也就是对 last-modified 的处理，然后调用 applyPreHandle 方法**执行拦截器的 preHandle 方法**，即遍历所有定义的 interceptor，执行 preHandle 方法，然后就到了实际执行 handle 的地方，doDispatch 方法中 handle 方法是执行当前 Handler，我们这里使用的是 RequestMappingHandlerAdapter，首先会进入 **AbstractHandlerMethodAdapter 的 handle 方法**：

```
public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    return handleInternal(request, response, (HandlerMethod) handler);
}
```

在 AbstractHandlerMethodAdapter 的 handle 方法中又调用了 **RequestMappingHandlerAdapter 类的 handleInternal 方法**：

```
protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
    ModelAndView mav;
    checkRequest(request);
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }
    }
    else {
        // 执行方法，封装 ModelAndView
        mav = invokeHandlerMethod(request, response, handlerMethod);
    }
    if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
        if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
            applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
        }
        else {
            prepareResponse(response);
        }
    }
    return mav;
}
```

在执行完 handle 方法后，然后调用 applyDefaultViewName 方法**组装默认视图名称**，将前缀和后缀名都加上，接着调用 applyPostHandle 方法**执行拦截器的 preHandle 方法**，也就是遍历所有定义的 interceptor，执行preHandle 方法。

## 处理最终结果以及渲染

最后调用 **DispatcherServlet 类中的 processDispatchResult 方法**，此方法是**处理最终结果的，包括异常处理、渲染页面和发出完成通知触发拦截器的 afterCompletion() 方法执行等**，processDispatchResult()方法代码如下：

```
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
    boolean errorView = false;
    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }
    if (mv != null && !mv.wasCleared()) {
        // 渲染
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }
    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        return;
    }
    if (mappedHandler != null) {
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```

接下来让我们看下 **DispatcherServlet 类的 render 方法**是如何完成渲染的，DispatcherServlet 类的 render 方法渲染过程如下：

1. 判断 ModelAndView 中 view 是否为 view name，没有获取其实例对象：如果是根据 name，如果是则需要调用 resolveViewName 从视图解析器获取对应的视图(View)对象；否则 ModelAndView 中使用 getview 方法获取 view 对象。
2. 然后调用 View 类的 render 方法。

DispatcherServlet 类的 render 方法如下：

```
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 设置本地化
    Locale locale = (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
    response.setLocale(locale);
    View view;
    String viewName = mv.getViewName();
    if (viewName != null) {
        // 解析视图名，得到视图
        view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
        if (view == null) {
            throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
                    "' in servlet with name '" + getServletName() + "'");
        }
    }
    else {
        view = mv.getView();
        if (view == null) {
            throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
                    "View object in servlet with name '" + getServletName() + "'");
        }
    }

    if (logger.isTraceEnabled()) {
        logger.trace("Rendering view [" + view + "] ");
    }
    try {
        if (mv.getStatus() != null) {
            response.setStatus(mv.getStatus().value());
        }
        // 委托给视图进行渲染
        view.render(mv.getModelInternal(), request, response);
    }
    catch (Exception ex) {
        if (logger.isDebugEnabled()) {
            logger.debug("Error rendering view [" + view + "]", ex);
        }
        throw ex;
    }
}
```

因为我们用的是 Thymeleaf 模版引擎，所以 view.render 找到对应的视图 **ThymeleafView 的 render 方法**进行渲染。

```
public void render(final Map<String, ?> model, final HttpServletRequest request, final HttpServletResponse response) throws Exception {
    renderFragment(this.markupSelectors, model, request, response);
}
```

ThymeleafView 的 render 方法又调用 **renderFragment** 方法进行视图渲染，渲染完成之后，DispatcherServlet 就可以将结果返回给我们了。

也就是 note 对应的 value：**在看转发二连**。

![](https://img-blog.csdnimg.cn/20200221003228283.png)

# 总结

通过本文的源码分析，我相信我们都能够清楚的认识到 SpringMVC 执行流程，进一步加深对 SpringMVC 的理解。

> 参考
> 
> https://dwz.cn/DmzEGQcx
> 
> https://dwz.cn/MSB2GJKT