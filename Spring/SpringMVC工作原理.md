### SpringMVC 简单介绍

SpringMVC 框架是以请求为驱动，围绕 Servlet 设计，将请求发给控制器，然后通过模型对象，分派器来展示请求结果视图。其中核心类是 DispatcherServlet，它是一个 Servlet，顶层是实现的Servlet接口。

### SpringMVC 工作原理（重要）

DispatcherServlet的执行链

![image](http://wx4.sinaimg.cn/large/007iUdjSgy1fybb3gg1ptj311s0f8t94.jpg)

**流程说明（重要）：**

（1）客户端（浏览器）发送请求，直接请求到 DispatcherServlet。

（2）DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。

（3）解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。

（4）HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑。

（5）处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。

（6）ViewResolver 会根据逻辑 View 查找实际的 View。

（7）DispaterServlet 把返回的 Model 传给 View（视图渲染）。

（8）把 View 返回给请求者（浏览器）

### SpringMVC 重要组件说明

**1、前端控制器DispatcherServlet（不需要工程师开发）,由框架提供（重要）**

作用：**Spring MVC 的入口函数。接收请求，响应结果，相当于转发器，中央处理器。有了 DispatcherServlet 减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，DispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性。**

>## 什么是DispatcherServlet？
>
>通过上面讲到的前端控制器模式，我们可以很轻易的知道`DispatcherServlet`是基于`Spring`的`Web`应用程序的中心点。它需要传入请求，并在处理程序映射，拦截器，控制器和视图解析器的帮助下，生成对客户端的响应。所以，我们可以分析这个类的细节，并总结出一些核心要点。
>
>下面是处理一个请求时`DispatcherServlet`执行的步骤：
>
>#### 1. 策略初始化
>
>`DispatcherServlet`是一个位于**org.springframework.web.servlet**包中的类，并扩展了同一个包中的抽象类`FrameworkServlet`。它包含一些解析器的私有静态字段(用于本地化，视图，异常或上传文件)，`映射处理器:handlerMapping`和`处理适配器:handlerAdapter`(进入这个类的第一眼就能看到的)。`DispatcherServlet`非常重要的一个核心点就是是初始化策略的方法(**protected void initStrategies（ApplicationContext context）**)。在调用`onRefresh`方法时调用此方法。最后一次调用是在`FrameworkServlet`中通过`initServletBean`和`initWebApplicationContext`方法进行的(`initServletBean`方法中调用`initWebApplicationContext`，后者调用`onRefresh(wac)`)。`initServletBean`通过所提供的这些策略生成我们所需要的应用程序上下文。其中每个策略都会产生一类在`DispatcherServlet`中用来处理传入请求的对象。
>
>```java
>/**
> * This implementation calls {@link #initStrategies}.
> */
>@Override
>protected void onRefresh(ApplicationContext context) {
>	initStrategies(context);
>}
>
>/**
> * Initialize the strategy objects that this servlet uses.
> * <p>May be overridden in subclasses in order to initialize further strategy objects.
> */
>protected void initStrategies(ApplicationContext context) {
>	initMultipartResolver(context);
>	initLocaleResolver(context);
>	initThemeResolver(context);
>	initHandlerMappings(context);
>	initHandlerAdapters(context);
>	initHandlerExceptionResolvers(context);
>	initRequestToViewNameTranslator(context);
>	initViewResolvers(context);
>	initFlashMapManager(context);
>}
>```
>
>#### 2.请求预处理
>
>`FrameworkServlet`抽象类扩展了同一个包下的`HttpServletBean`，`HttpServletBean`扩展了**javax.servlet.http.HttpServlet**。点开这个类源码可以看到，`HttpServlet`是一个抽象类，其方法定义主要用来处理每种类型的`HTTP`请求：`doGet（GET请求）`，`doPost（POST）`，`doPut（PUT）`，`doDelete（DELETE）`，`doTrace（TRACE）`，`doHead（HEAD）`，`doOptions（OPTIONS）`。`FrameworkServlet`通过将每个传入的请求调度到**processRequest(HttpServletRequest request，HttpServletResponse response)**来覆盖它们。`processRequest`是一个`protected`和`final`的方法，它构造出`LocaleContext`和`ServletRequestAttributes`对象，两者都可以在`initContextHolders(request, localeContext, requestAttributes)`之后访问。
>
>
>
>#### 3.请求处理
>
>由上面所看到的，在`processRequest`的代码中，调用**initContextHolders**方法后，调用**protected void doService(HttpServletRequest request，HttpServletResponse response)**。doService将一些附加参数放入request（如Flash映射:`request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap())`，上下文信息:`request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext())`等）中，并调用**protected void doDispatch(HttpServletRequest request，HttpServletResponse response)**。
>
>`doDispatch`方法最重要的部分是处理(`handler`)的检索。`doDispatch`调用`getHandler()`方法来分析处理后的请求并返回`HandlerExecutionChain`实例。此实例包含`handler mapping` 和``interceptors(拦截器)`。`DispatcherServlet`做的另一件事是应用预处理程序拦截器（*applyPreHandle()*）。如果至少有一个返回`false`，则请求处理停止。否则，`servlet`使用与`handler adapter`适配(其实理解成这也是个`handler`就对了)相应的`handler mapping`来生成视图对象。
>
>HandlerMapping   继承RequestMappingHandlerMapping类 ，读取自定义的注解然后匹配看是否有映射关系
>
>- **protected void registerHandlerMethod(Object handler，Method method，RequestMappingInfo mapping)**:
>- **protected boolean isHandler(Class beanType)**: 检查bean是否符合给定处理程序的条件。
>- **protected RequestMappingInfo getMappingForMethod(Method method，Class handlerType)**: 为给定的Method实例提供映射的方法，该方法表示处理的方法（例如，使用`@RequestMapping`注解的`controller`的方法上所对应的`URL`）。
>- **protected HandlerMethod handleNoMatch(Set requestMappingInfos, String lookupPath, HttpServletRequest request)** : 在给定的`HttpServletRequest`对象找不到匹配的处理方法时被调用。
>- **protected void handleMatch(RequestMappingInfo info, String lookupPath, HttpServletRequest request)** : 当为给定的`HttpServletRequest`对象找到匹配的处理方法时调用。
>
>**HandlerAdapter**接口，它只有3种方法：
>
>- **boolean supports(Object handler)**:检查传入参数的对象是否可以由此适配器处理
>- **ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)** : 将请求翻译成视图。
>- **long getLastModified(HttpServletRequest request, Object handler)**:返回给定`HttpServletRequest`的最后修改日期，以毫秒为单位。
>
>#### 4.视图解析
>
>获取`ModelAndView`实例以查看呈现后，`doDispatch`方法调用**private void applyDefaultViewName(HttpServletRequest request，ModelAndView mv)**。默认视图名称根据定义的bean名称，即`viewNameTranslator`。默认情况下，它的实现是**org.springframework.web.servlet.RequestToViewNameTranslator**。这个默认实现只是简单的将URL转换为视图名称，例如(直接从`RequestToViewNameTranslator`获取):http:// localhost:8080/admin/index.html将生成视图admin / index。
>
>
>
>#### 5.处理调度请求 - 视图渲染
>
>现在，`servlet`知道应该是哪个视图被渲染。它通过**private void processDispatchResult(HttpServletRequest request，HttpServletResponse response，@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,@Nullable Exception exception)**方法来进行最后一步操作 - 视图渲染。
>
>首先，`processDispatchResult`检查它们是否有参数传递异常。有一些异常的话，它定义了一个新的视图，专门用来定位错误页面。如果没有任何异常，该方法将检查`ModelAndView实例`，如果它不为`null`，则调用`render`方法。
>
>渲染方法`protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception`。跳进此方法内部，根据定义的视图策略，它会查找得到一个`View类`实例。它将负责显示响应。如果没有找到`View`，则会抛出一个`ServletException异常`。有的话，`DispatcherServlet`会调用其`render`方法来显示结果。
>
>其实可以说成是后置拦截器(进入拦截器前置拦截处理->controller处理->出拦截器之前的此拦截器的后置处理)，也就是在请求处理的最后一个步骤中被调用。

**2、处理器映射器HandlerMapping(不需要工程师开发),由框架提供**

作用：根据请求的url查找Handler。HandlerMapping负责根据用户请求找到Handler即处理器（Controller），SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

**3、处理器适配器HandlerAdapter**

作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler
通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**4、处理器Handler(需要工程师开发)**

注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler
Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。
由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

**5、视图解析器View resolver(不需要工程师开发),由框架提供**

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。
一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**6、视图View(需要工程师开发)**

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

**注意：处理器Handler（也就是我们平常说的Controller控制器）以及视图层view都是需要我们自己手动开发的。其他的一些组件比如：前端控制器DispatcherServlet、处理器映射器HandlerMapping、处理器适配器HandlerAdapter等等都是框架提供给我们的，不需要自己手动开发。**

### DispatcherServlet详细解析

首先看下源码：

```java
package org.springframework.web.servlet;
 
@SuppressWarnings("serial")
public class DispatcherServlet extends FrameworkServlet {
 
	public static final String MULTIPART_RESOLVER_BEAN_NAME = "multipartResolver";
	public static final String LOCALE_RESOLVER_BEAN_NAME = "localeResolver";
	public static final String THEME_RESOLVER_BEAN_NAME = "themeResolver";
	public static final String HANDLER_MAPPING_BEAN_NAME = "handlerMapping";
	public static final String HANDLER_ADAPTER_BEAN_NAME = "handlerAdapter";
	public static final String HANDLER_EXCEPTION_RESOLVER_BEAN_NAME = "handlerExceptionResolver";
	public static final String REQUEST_TO_VIEW_NAME_TRANSLATOR_BEAN_NAME = "viewNameTranslator";
	public static final String VIEW_RESOLVER_BEAN_NAME = "viewResolver";
	public static final String FLASH_MAP_MANAGER_BEAN_NAME = "flashMapManager";
	public static final String WEB_APPLICATION_CONTEXT_ATTRIBUTE = DispatcherServlet.class.getName() + ".CONTEXT";
	public static final String LOCALE_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".LOCALE_RESOLVER";
	public static final String THEME_RESOLVER_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_RESOLVER";
	public static final String THEME_SOURCE_ATTRIBUTE = DispatcherServlet.class.getName() + ".THEME_SOURCE";
	public static final String INPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".INPUT_FLASH_MAP";
	public static final String OUTPUT_FLASH_MAP_ATTRIBUTE = DispatcherServlet.class.getName() + ".OUTPUT_FLASH_MAP";
	public static final String FLASH_MAP_MANAGER_ATTRIBUTE = DispatcherServlet.class.getName() + ".FLASH_MAP_MANAGER";
	public static final String EXCEPTION_ATTRIBUTE = DispatcherServlet.class.getName() + ".EXCEPTION";
	public static final String PAGE_NOT_FOUND_LOG_CATEGORY = "org.springframework.web.servlet.PageNotFound";
	private static final String DEFAULT_STRATEGIES_PATH = "DispatcherServlet.properties";
	protected static final Log pageNotFoundLogger = LogFactory.getLog(PAGE_NOT_FOUND_LOG_CATEGORY);
	private static final Properties defaultStrategies;
	static {
		try {
			ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, DispatcherServlet.class);
			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Could not load 'DispatcherServlet.properties': " + ex.getMessage());
		}
	}
 
	/** Detect all HandlerMappings or just expect "handlerMapping" bean? */
	private boolean detectAllHandlerMappings = true;
 
	/** Detect all HandlerAdapters or just expect "handlerAdapter" bean? */
	private boolean detectAllHandlerAdapters = true;
 
	/** Detect all HandlerExceptionResolvers or just expect "handlerExceptionResolver" bean? */
	private boolean detectAllHandlerExceptionResolvers = true;
 
	/** Detect all ViewResolvers or just expect "viewResolver" bean? */
	private boolean detectAllViewResolvers = true;
 
	/** Throw a NoHandlerFoundException if no Handler was found to process this request? **/
	private boolean throwExceptionIfNoHandlerFound = false;
 
	/** Perform cleanup of request attributes after include request? */
	private boolean cleanupAfterInclude = true;
 
	/** MultipartResolver used by this servlet */
	private MultipartResolver multipartResolver;
 
	/** LocaleResolver used by this servlet */
	private LocaleResolver localeResolver;
 
	/** ThemeResolver used by this servlet */
	private ThemeResolver themeResolver;
 
	/** List of HandlerMappings used by this servlet */
	private List<HandlerMapping> handlerMappings;
 
	/** List of HandlerAdapters used by this servlet */
	private List<HandlerAdapter> handlerAdapters;
 
	/** List of HandlerExceptionResolvers used by this servlet */
	private List<HandlerExceptionResolver> handlerExceptionResolvers;
 
	/** RequestToViewNameTranslator used by this servlet */
	private RequestToViewNameTranslator viewNameTranslator;
 
	private FlashMapManager flashMapManager;
 
	/** List of ViewResolvers used by this servlet */
	private List<ViewResolver> viewResolvers;
 
	public DispatcherServlet() {
		super();
	}
 
	public DispatcherServlet(WebApplicationContext webApplicationContext) {
		super(webApplicationContext);
	}
	@Override
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}
 
	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
}

```

DispatcherServlet类中的属性beans：

- HandlerMapping：用于handlers映射请求和一系列的对于拦截器的前处理和后处理，大部分用@Controller注解。
- HandlerAdapter：帮助DispatcherServlet处理映射请求处理程序的适配器，而不用考虑实际调用的是 哪个处理程序。- - - 
- 

在Web MVC框架中，每个DispatcherServlet都拥自己的WebApplicationContext，它继承了ApplicationContext。WebApplicationContext包含了其上下文和Servlet实例之间共享的所有的基础框架beans。

**HandlerMapping**

- DefaultAnnotationHandlerMapping类通过注解把URL映射到Controller类。

**HandlerAdapter**

AnnotationMethodHandlerAdapter：通过注解，把请求URL映射到Controller类的方法上。











从技术上讲，前端控制器模式由一个捕获所有传入请求的类组成。之后，分析每个请求以知道哪个控制器以及哪个方法应该来处理该请求。

## DispatcherServlet的执行链

前端控制器模式有自己的执行链。这意味着它有自己的逻辑来处理请求并将视图返回给客户端：

1. 请求由客户端发送。它到达作为Spring的默认前端控制器的`DispatcherServlet`类。

2. `DispatcherServlet`使用请求处理程序映射来发现将分析请求的控制器(controller

   )。接口**org.springframework.web.servlet.HandlerMapping**的实现返回一个包含**org.springframework.web.servlet.HandlerExecutionChain**类的实例。此实例包含可在控制器调用之前或之后调用的处理程序拦截器数组。你可以在Spring中有关于拦截器的文章中了解更多的信息。如果在所有定义的处理程序映射中找不到`HandlerExecutionChain`，这意味着Spring无法将URL与对应的控制器进行匹配。这样的话会抛出一个错误。

3. 现在系统进行拦截器预处理并调用由映射处理器找到的相应的controller(其实就是在找到的controller之前进行一波拦截处理)。在controller处理请求后，`DispatcherServlet`开始拦截器的后置处理。在此步骤结束时，它从controller接收ModelAndView实例(整个过程其实就是 `request请求`->`进入interceptors`->`controller`->`从interceptors出来`->`ModelAndView接收`)。

4. DispatcherServlet现在将使用的该视图的名称发送到视图解析器。这个解析器将决定前台的展现内容。接着，它将此视图返回给DispatcherServlet，其实也就是一个“视图生成后可调用”的拦截器。

5. 最后一个操作是视图的渲染并作为对客户端request请求的响应。

DispatcherServlet 一个调度器servlet。它是一个处理所有传入请求并将视图呈现给用户的类。在重写之前，你应该熟悉执行链，handler mapping 或handler adapter等概念。请记住，第一步要做的是定义在调度过程中我们要调用的所有元素。handler mapping 是将传入请求(也就是它的URL)映射到适当的controller。最后提到的元素，一个handler适配器，就是一个对象，它将通过其内包装的handler mapping将请求发送到controller。此调度产生的结果是ModelAndView类的一个实例，后面被用于生成和渲染视图。

### 策略初始化

**protected void onRefresh(ApplicationContext context)**  调用**initStrategies(context);** 初始化策略对象

**protected void initStrategies(ApplicationContext context)**调用**initLocaleResolver(context);**

**initLocaleResolver(context);** 捕获异常，采用默认策略  调用  **getDefaultStrategy**

### 请求预处理

FrameworkServlet继承HttpServletBean       HttpServletBean继承HttpServlet

`FrameworkServlet`通过将每个传入的请求调度到**processRequest(HttpServletRequest request，HttpServletResponse response)**来覆盖它们。`processRequest`是一个`protected`和`final`的方法，它构造出`LocaleContext`和`ServletRequestAttributes`对象，两者都可以在`initContextHolders(request, localeContext, requestAttributes)`之后访问



**protected final void doGet(HttpServletRequest request, HttpServletResponse response)**调用**processRequest(request, response);** 

**protected final void processRequest(HttpServletRequest request, HttpServletResponse response)**调用**private void initContextHolders(HttpServletRequest request,
​			@Nullable LocaleContext localeContext, @Nullable RequestAttributes requestAttributes)** 

### 请求处理

调用**doService(request, response);**         doService将一些附加参数放入request（映射，上下文信息）



`doDispatch`方法最重要的部分是处理(`handler`)的检索。`doDispatch`调用`getHandler()`方法来分析处理后的请求并返回`HandlerExecutionChain`实例。此实例包含`handler mapping` 和``interceptors(拦截器)`。`DispatcherServlet`做的另一件事是应用预处理程序拦截器（*applyPreHandle()*）。如果至少有一个返回`false`，则请求处理停止。否则，`servlet`使用与 `handler adapter`适配(其实理解成这也是个`handler`就对了)相应的`handler mapping`来生成视图对象。



DispatcherServlet

```java
@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		if (logger.isDebugEnabled()) {
			String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
			logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
					" processing " + request.getMethod() + " request for [" + getRequestUri(request) + "]");
		}

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
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

		// Make framework objects available to handlers and view objects.
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
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}
```



```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
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
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}				
```

HandlerAdapter取可处理request的Handler，调用的相应的Handler

**mv = ha.handle(processedRequest, response, mappedHandler.getHandler()); **  调用controller

**applyDefaultViewName(processedRequest, mv);**视图解析

**processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);**视图渲染



## handle

handler mappings 处理程序映射   request 匹配 controller

handler adapter  处理器适配器   从handler mappings 获取映射的controller和方法并调用必须实现handlerAdaper





### Interceptor

过滤器只能在servlet容器下使用。Spring容器不一定运行在web环境中，在这种情况下过滤器就不好使了，而拦截器依然可以在Spring容器中调用。









@ModelAttribute注解用于将动态请求参数`转换`为Java注解中指定的对象 ,Spring会尝试将所有请求参数匹配到Article类的字段中

必须将`BindingResult`实例直接放在经过验证的对象之后

[modelarrtibute](https://blog.csdn.net/hejingyuan6/article/details/49995987)









