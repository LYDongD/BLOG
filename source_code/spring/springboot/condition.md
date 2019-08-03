## 条件

### @Profile注解的实现原理

@Profile注解依赖@Conditional(ProfileCondition.class)

```
class ProfileCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //1 读取@Profile元信息
		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
		if (attrs != null) {
            //2 读取value属性值，与环境比较判断是否匹配当前环境
			for (Object value : attrs.get("value")) {
				if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
					return true;
				}
			}
			return false;
		}

        //3 如果没有指定任何属性，则认为匹配当前环境
		return true;
	}

}

```

> 如何存储注解元信息

这里是有MultiValueMap存储注解元信息，其中：

key: 注解的属性
value: 属性的值，可以是任意对象，包括数组

MultiValueMap接口支持多值value操作，比如直接添加一个列表或字典等

```
public interface MultiValueMap<K, V> extends Map<K, List<V>> {
    @Nullable
    V getFirst(K var1);

    void add(K var1, @Nullable V var2);

    void addAll(K var1, List<? extends V> var2);

    void addAll(MultiValueMap<K, V> var1);

    void set(K var1, @Nullable V var2);

    void setAll(Map<K, V> var1);

    Map<K, V> toSingleValueMap();
}

```

> 如何判断环境是否匹配

1 通过特定方式激活环境

```
spring.profiles.active 或 @ActiveProfile的形式

```

2 使用@Profile指定spring组件适用的环境

* 可以添加在@Component包括@Configuration注释的类上
* 可以添加在@Bean注释的方法上
* 作为其他注解的元注解

@Profile的value支持简单名称或复合表达式，例如"p1 & p2"

Profiles.of((String[]) value) 支持对expression的解析

```

static Profiles of(String... profiles) {
	return ProfilesParser.parse(profiles);
}

通过ProfilesParser进行解析：

private static Profiles parseExpression(String expression) {
	Assert.hasText(expression, () -> "Invalid profile expression [" + expression + "]: must contain text");
    //expression支持关系符: (), & | 和 !, 可见，expression可以配置环境的组合
	StringTokenizer tokens = new StringTokenizer(expression, "()&|!", true);
	return parseTokens(expression, tokens);
}


```

3 环境匹配，通过Profiles和谓词实现

输入谓词，对捕获了Profile(注解指定的环境)的Profiles进行校验，并返回校验结果

```
	@Override
	public boolean acceptsProfiles(Profiles profiles) {
		Assert.notNull(profiles, "Profiles must not be null");
		return profiles.matches(this::isProfileActive);
	}

```

这里的Profiles，是简单的，或根据逻辑关系符(&|!)等组合成的组合Profiles生成的数组(ProfilesParser) 包装器ProfilesParser

简单的Profiles:

```
//该方法返回的Profiles捕获了参数profie
private static Profiles equals(String profile) {
	return activeProfile -> activeProfile.test(profile);
}

```


谓词 activeProfile实现:

```
protected boolean isProfileActive(String profile) {
	validateProfile(profile);
	Set<String> currentActiveProfiles = doGetActiveProfiles();
	return (currentActiveProfiles.contains(profile) ||
			(currentActiveProfiles.isEmpty() && doGetDefaultProfiles().contains(profile)));
}

```

多个Profiles通过and,or或not等关系符的包装实现组合型的Profiles

```
//要求所有profile执行结果都返回true才返回true
private static Profiles and(Profiles... profiles) {
		return activeProfile -> Arrays.stream(profiles).allMatch(isMatch(activeProfile));
}

//要求任意一个profile执行结果返回true即返回true
private static Profiles or(Profiles... profiles) {
	return activeProfile -> Arrays.stream(profiles).anyMatch(isMatch(activeProfile));
}

其中isMatch

private static Predicate<Profiles> isMatch(Predicate<String> activeProfile) {
		return profiles -> profiles.matches(activeProfile);
}

```

装饰器ParsedProfiles实现Profile接口，统一管理Profiles[]

```
	static Profiles parse(String... expressions) {
		Assert.notEmpty(expressions, "Must specify at least one profile");
		Profiles[] parsed = new Profiles[expressions.length];
		for (int i = 0; i < expressions.length; i++) {
			parsed[i] = parseExpression(expressions[i]);
		}
		return new ParsedProfiles(expressions, parsed);
	}

```

统一匹配：

```
  @Override
  public boolean matches(Predicate<String> activeProfiles) {
  	for (Profiles candidate : this.parsed) {
  		if (candidate.matches(activeProfiles)) {
  			return true;
  		}
  	}
  	return false;
  }

```
