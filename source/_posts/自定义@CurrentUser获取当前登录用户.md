title: 自定义@CurrentUser获取当前登录用户
date: 2016-08-01 20:23:43
categories: java

---

# 定义 @CurrenUser注解

	/**
	 * 在Controller的方法参数中使用此注解，该方法在映射时会注入当前登录的User对象
	 */
	@Target(ElementType.PARAMETER)//可用在方法的参数上
	@Retention(RetentionPolicy.RUNTIME)//运行时有效
	public @interface CurrentUser {
	}

# 添加测试方法
在 userApi.java 中添加临时测试用的，测试完记得删掉。

	@GetMapping("/test")
	@LoginRequired
	public Object testCurrentUser(@CurrentUser User user) {
	    return user;
	}

不要忘了添加 @LoginRequired 这个注解（上节添加的），要获取当前登录用户嘛，肯定得要求用户登录。重启项目访问 /api/user/test测试下。

![](\img\3907956-5d831a4f4501a5b8.png)

记得填写 token（访问登录接口获取）。全是null，很遗憾现在还不能返回我们想要的用户信息。

# 添加参数解析器

要想 @CurrentUser 起作用，需要编写一个配套解析器，做法是实现 spring 提供的 HandlerMethodArgumentResolver 接口。
新增 CurrentUserMethodArgumentResolver.java

	/**
	 *  增加方法注入，将含有 @CurrentUser 注解的方法参数注入当前登录用户
	 */
	public class CurrentUserMethodArgumentResolver implements HandlerMethodArgumentResolver {
	    @Override
	    public boolean supportsParameter(MethodParameter parameter) {
	        return parameter.getParameterType().isAssignableFrom(User.class)
	                && parameter.hasParameterAnnotation(CurrentUser.class);
	    }
	    @Override
	    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
	        User user = (User) webRequest.getAttribute("currentUser", RequestAttributes.SCOPE_REQUEST);
	        if (user != null) {
	            return user;
	        }
	        throw new MissingServletRequestPartException("currentUser");
	    }
	}

User user = (User) webRequest.getAttribute("currentUser", RequestAttributes.SCOPE_REQUEST) 这一句是从 request 作用域中取出名为 currentUser 的属性。currentUser 是什么呢？在上节编写的登录拦截器中，最后有这么一句 request.setAttribute("currentUser", user)，所以 currentUser 是 token 验证通过之后查询到的当前用户。

# 配置参数解析器
在 WebMvcConfigurer.java 中 Override addArgumentResolvers 方法

	@Override
	public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
	    argumentResolvers.add(currentUserMethodArgumentResolver());
	    super.addArgumentResolvers(argumentResolvers);
	}
	@Bean
	public CurrentUserMethodArgumentResolver currentUserMethodArgumentResolver() {
	    return new CurrentUserMethodArgumentResolver();
	}


# 再次测试
我是用 “张三” 这个用户登录获取的 token，正确地返回了张三的信息。

![](\img\3907956-1dda70c8a9342adf.png)