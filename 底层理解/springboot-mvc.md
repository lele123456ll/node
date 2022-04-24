## MVC配置

1. 首页配置

    1. 注意, 所有的静态资源都需要使用thymeleaf接管
    2. @{}

2. 页面国际化

    1. 需要配置i18n文件
    2. 我们如果需要在项目中进行按钮自动切换, 需要自定义组件LocalResolver
    3. 最终需要将其配置到spring容器中, 加@Bean
    4. #{}

    ```java
    public class MyLocalResolver implements LocaleResolver {
    
        @Override
        public Locale resolveLocale(HttpServletRequest request) {
            // 语言参数
            String lang = request.getParameter("l");
            Locale locale = Locale.getDefault(); // 默认
    
            // lang不为空, 携带了国际化参数
            if (!StringUtils.isEmpty(lang)) {
                String[] split = lang.split("_");
                // 国家地区
                locale = new Locale(split[0], split[1]);
            }
    
            return locale;
    
        }
    
        @Override
        public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
    
        }
    }
    ```

    