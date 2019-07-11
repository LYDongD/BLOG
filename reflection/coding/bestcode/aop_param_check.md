### AOP应用之参数校验

#### 定义切面

扫描所有添加@Valid注解的参数，内部调用spring validation 框架实现参数校验

```
	//手动校验，注入spring Validator
	@Autowired
	private Validator validator;

//定义切点，拦截所有添加了@ReqeustMapping的方法
@Pointcut("@annotation(org.springframework.web.bind.annotation.RequestMapping)") 
	private void controllerInvocation(){		
	}
	
	@Around("controllerInvocation()")
	public Object aroundController(ProceedingJoinPoint joinPoint) throws Throwable {
		//获取方法中参数的注解，只有添加了@MyValid注解的才进行处理
		MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
		Method method = methodSignature.getMethod();
		Annotation[][] argAnnotations = method.getParameterAnnotations();
		String[] argNames = methodSignature.getParameterNames(); 
		Object[] args = joinPoint.getArgs(); 
		for (int i = 0; i < args.length; i++) {
			//仅处理@MyValid参数
			if (hasValidAnnotations(argAnnotations[i])) {
				Object ret = validateArg(args[i], argNames[i]); 
				//校验不通过，则返回失败提示响应消息
				if(ret != null){
					return ret; 
				}
			}
		}
		
		//校验通过或无需进行校验，统一对响应消息进行序列化
		ObjectMapper mapper = new ObjectMapper();
		//序列化时忽略null属性
		mapper.setSerializationInclusion(Include.NON_NULL);
		mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
		return mapper.writeValueAsString(joinPoint.proceed(args));
	}
	
	//是否包含@MyValid注解
	private boolean hasValidAnnotations(Annotation[] annotations){
		if (annotations.length < 1)  
		      return false;  		   
		    for (Annotation annotation : annotations) {  
		      if (MyValid.class.isInstance(annotation))  
		    	  return true;  
		    }  
		    return false;  
	}
	
	/**
	 * 参数验证，区分列表和普通参数
	 * @param arg
	 * @param argName
	 * @return
	 */
	private Object validateArg(Object arg, String argName) {  
	    BindingResult result = getBindingResult(arg, argName);
	    if(arg==null){
	    	return JsonResult.fail(GlobalResultStatus.PARAM_MISSING);
	    }
	    if(arg instanceof List){	    	
			@SuppressWarnings("rawtypes")
			Iterator it = ((List) arg).iterator();
			while(it.hasNext())
				validator.validate(it.next(), result);
		}else
			validator.validate(arg, result);  
	    if (result.hasErrors()){
	    	List<ObjectError> errors = result.getAllErrors();
	    	Iterator<ObjectError> iterator = errors.iterator();
	    	Map<String, String> errorMap = new HashMap<String, String>();
	    	while(iterator.hasNext()){
	    		errorMap.put(iterator.next().getCode(), "true");
	    	}
	    	if("true".equals(errorMap.get("NotEmpty"))||"true".equals(errorMap.get("NotNull"))){
	    		return JsonResult.fail(GlobalResultStatus.PARAM_MISSING);
	    	}
	    	return JsonResult.fail(GlobalResultStatus.PARAM_ERROR,result.getAllErrors().get(0).getDefaultMessage());
	    }
	    return null;  
	  }  
	
	private BindingResult getBindingResult(Object target, String targetName) {  
	    return new BeanPropertyBindingResult(target, targetName);  
	  } 	


```

---

### jackson序列化与反序列使用方式

[jackson高阶应用](https://www.ibm.com/developerworks/cn/java/jackson-advanced-application/index.html)

---

### 手动校验的两种方式

**方式1**

```
容器注入, 默认使用spring容器的LocalValidatorFactoryBean

@Autowired
private Validator validator

```

**方式2**

```
通过工厂获取

ValidatorFactory vf = Validation.buildDefaultValidatorFactory();
Validator validator = vf.getValidator();

```


---

### 自定义校验注解

**STEP1: 定义注解, 例如枚举值校验注解, 指定注解处理器**

```
@Constraint(validatedBy = EnumValue.Validator.class)
public @interface EnumValue {}

```

**STEP2: 定义注解处理器，可写成内部类的形式**

```
/**
 *  枚举类型标记，用于参数校验注解
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = EnumValue.Validator.class) //指定校验类
public @interface EnumValue {

    String message() default "枚举参数校验失败";

	//分组
    Class<?>[] groups() default {};

	//负载
    Class<? extends Payload>[] payload() default {};

    //枚举类
    Class<? extends Enum<?>> enumClass();

    //验证方法
    String enumMethod();

	//注解处理器
    class Validator implements ConstraintValidator<EnumValue, Object> {

        private Class<? extends Enum<?>> enumClass;
        private String enumMethod;

        @Override
        public void initialize(EnumValue enumValue) {
            enumMethod = enumValue.enumMethod();
            enumClass = enumValue.enumClass();
        }

        @Override
        public boolean isValid(Object value, ConstraintValidatorContext constraintValidatorContext) {

            if (value == null) {
                return true;
            }

            if (enumClass == null || enumMethod == null) {
                return true;
            }

            //待校验参数的类型
            Class<?> valueClass = value.getClass();
            //获取校验方法
            try {
                Method method = enumClass.getMethod(enumMethod, valueClass);

                //方法必须返回boolean
                if (!Boolean.TYPE.equals(method.getReturnType()) && !Boolean.class.equals(method.getReturnType())) {
                    throw new ValidationException("枚举参数的校验方法返回类型不是boolean，需要改成boolean类型");
                }

                if (!Modifier.isStatic(method.getModifiers())) {
                    throw new ValidationException("枚举参数的校验方法类型不是static的，需要改成static");
                }

                //调用校验方法
                Boolean result = (Boolean)method.invoke(null, value);
                return result == null ? false : result;
            }catch (Exception e) {
                throw new ValidationException(e.getMessage());
            }
        }
    }
}

```

**STEP3: 在参数模型内添加注解**

```
public class Rule implements Serializable {
	@EnumValue(enumClass=RuleTypeEnum.class, enumMethod="isValid", message = "规则类型不正确")
    private Integer type;
}
```

**测试**

```
public class RuleControllerTest extends BaseControllerTest{

    @Test
    public void addOrUpdate() throws Exception{
        String responseResult = mockMvc.perform(MockMvcRequestBuilders.post("/rule/addOrUpdate/v2.0")
                .param("ruleName", "测试参数校验规则1")
                .param("type", "3"))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andReturn().getResponse().getContentAsString();

        logger.info("请求结果:{}",responseResult);

    }
```

**结果**

```
{"msg":"参数错误","code":100010202,"data":"规则类型不正确"}
;
```
