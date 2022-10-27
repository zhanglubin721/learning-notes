# Spring 请求方法的调用原理（Controller）和请求参数的获取的原理

## 1、请求映射原理

- 所有的请求都会经过`DispatcherServlet`这个类，先了解它的继承树

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224452787-2050869535.png)

本质还是httpServlet

- 原理图

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224506255-1224198216.png)

- 测试

  request请求携带的参数

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224539213-21592543.png)

 从requestMapping中寻找请求方法

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224549620-1409621928.png)

```undefined
就可以获取到请求的方法
```

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224555697-2008855586.png)

 拿到这个方法后最终会调用DispatchServlet

```java
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

 然后调用AbstractHandlerMethodAdapter的handleInternal()方法，被RequestMappingHandlerAdapter实现

```java
@Override
protected ModelAndView handleInternal(HttpServletRequest request,
                                      HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
 
    ModelAndView mav;
    checkRequest(request);
 
    // Execute invokeHandlerMethod in synchronized block if required.
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            // No HttpSession available -> no mutex necessary
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }
    }
    else {
        // No synchronization on session demanded at all...
        mav = invokeHandlerMethod(request, response, handlerMethod);
    }
```

 主要调用invokeHandlerMethod(request, response, handlerMethod);方法

```java
invocableMethod.invokeAndHandle(webRequest, mavContainer);
```

 再调用invokeAndHandle()方法

```java
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
 
 
@Nullable
public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
                               Object... providedArgs) throws Exception {
 
    Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
    if (logger.isTraceEnabled()) {
        logger.trace("Arguments: " + Arrays.toString(args));
    }
    return doInvoke(args);
}
```

 在通过doInvoke(args);实现方法的调用

```java
@Nullable
protected Object doInvoke(Object... args) throws Exception {
    Method method = getBridgedMethod();
    try {
        if (KotlinDetector.isSuspendingFunction(method)) {
            return CoroutinesUtils.invokeSuspendingFunction(method, getBean(), args);
        }
        return method.invoke(getBean(), args);
    }
    .......
```

`method.invoke(getBean(), args);`这就是通过反射实现方法的调用

## 2、请求参数注解处理原理

原理图

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224735993-1251882532.png)

1. 先得到请求的request

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224747933-1268883404.png)

1. 在获取可以处理请求的方法的Mapping映射器

   ```java
    DispatcherServlet中的
        doDispatch方法
           // Determine handler for the current request.
           mappedHandler = getHandler(processedRequest);
   ```

2. 判断每个参数带有注解是哪个，是否存在相应的解析器

   - 寻找合适的处理适配器

     ```java
      DispatcherServlet中的
          doDispatch方法		
             // Determine handler adapter for the current request.
             HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
     ```

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224758145-1157509296.png)

- 寻找可以处理相应注解的处理器

```java
第一步
DispatcherServlet中的
    doDispatch方法  
        // Actually invoke the handler.
        mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 
第二步
RequestMappingHandlerAdapter中的
    handleInternal方法
    	// No synchronization on session demanded at all...
			mav = invokeHandlerMethod(request, response, handlerMethod);
	invokeHandlerMethod方法
        invocableMethod.invokeAndHandle(webRequest, mavContainer);
 
第三步
ServletInvocableHandlerMethod中的
    invokeAndHandle方法
    	Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
 
第四步
InvocableHandlerMethod类中的
	invokeForRequest方法
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);//获取请求的参数
 
第五步
InvocableHandlerMethod类中的
    getMethodArgumentValues方法
            if (!this.resolvers.supportsParameter(parameter)) {//寻找处理相关注解的处理器,并保存到缓存中 supportsParameter(parameter)从这里进入的
                throw new IllegalStateException(formatArgumentError(parameter, 								"No suitable resolver"));
            }
        	try {
            args[i] = this.resolvers.resolveArgument(parameter, 	mavContainer, 					request, this.dataBinderFactory);
        }
 
第六步
HandlerMethodArgumentResolverComposite类中的
    	@Nullable
        private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter 														parameter) {
            HandlerMethodArgumentResolver result = 	
                this.argumentResolverCache.get(parameter);//从缓存中获取，开始肯定没有
            if (result == null) {
                //增强for循环中选择合适的处理器  27个
                for (HandlerMethodArgumentResolver resolver :
                     this.argumentResolvers) {
                    if (resolver.supportsParameter(parameter)) {
                        result = resolver;
                        //保存到缓存中
                        this.argumentResolverCache.put(parameter, result);
                        break;
                    }
                }
            }
            return result;
        }
```

![img](https://img2022.cnblogs.com/blog/2745283/202206/2745283-20220615224839048-859498609.png)

- 通过注解处理器获取参数

  ```java
  第一步
  InvocableHandlerMethod类中的
      getMethodArgumentValues方法
   
          try {
              args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory); //真正开始获取参数          
          }
   
  第二步
  HandlerMethodArgumentResolverComposite类中的
      resolveArgument方法中
      	//获取参数对应注解的处理器
      	HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
  	getArgumentResolver方法中
      //从缓存中获取，由于开始的之前往缓存中存入了，所以现在可以直接拿，如果没有下面还有可以循环寻找
          HandlerMethodArgumentResolver result = 		    		
          				this.argumentResolverCache.get(parameter);
      
      
  AbstractNamedValueMethodArgumentResolver类中的
      resolveArgument方法
      	Object resolvedName = 
      		resolveEmbeddedValuesAndExpressions(namedValueInfo.name);//获取参数名
  		Object arg = resolveName(resolvedName.toString(), 
                                   nestedParameter, webRequest);//获取参数值
   
  //具体怎么获取
  PathVariableMethodArgumentResolver类中的
      resolveName方法
      protected Object resolveName(String name, MethodParameter parameter, NativeWebRequest request) throws Exception {
      Map<String, String> uriTemplateVars = (Map<String, String>) request.getAttribute(//直接从request请求域中获取  
          HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE, RequestAttributes.SCOPE_REQUEST);
      return (uriTemplateVars != null ? uriTemplateVars.get(name) : null);//在通过参数名称确定是哪一个
  }
  ```

- 获取参数完成后，会调用InvocableHandlerMethod来中的doInvoke方法继续反射调用方法

  ```java
  InvocableHandlerMethod类中的
      doInvoke方法
      	return method.invoke(getBean(), args);
  ```