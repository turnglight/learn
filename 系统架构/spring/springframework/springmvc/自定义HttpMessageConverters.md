
~~~java
@Configuration
public class BoatWebMvcConfig implements WebMvcConfigurer {
@Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters){
        // 这一段代码必须加上，需要移除spring自带的httpMessageConverter，否则不生效
        Iterator<HttpMessageConverter<?>> iterator = converters.iterator();
        while(iterator.hasNext()){
            HttpMessageConverter<?> converter = iterator.next();
            if(converter instanceof MappingJackson2HttpMessageConverter){
                //将springboot的jackson的消息转换器移除
                iterator.remove();
            }
        }

        final MappingJackson2HttpMessageConverter jackson2HttpMessageConverter = new MappingJackson2HttpMessageConverter();

        // Long->String
        final ToStringSerializer toStringSerializer = new ToStringSerializer(Long.class);
        // date-> pattern format
//        final DateSerializer dateSerializer = new DateSerializer(false, new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));
        SimpleModule simpleModule = new SimpleModule();
        simpleModule.addSerializer(toStringSerializer);
//        simpleModule.addSerializer(dateSerializer);
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(simpleModule).getSerializerProvider().setNullValueSerializer(new JsonSerializer<Object>() {
            @Override
            public void serialize(Object o, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                jsonGenerator.writeString("");
            }
        });
        jackson2HttpMessageConverter.setObjectMapper(mapper);
        converters.add(jackson2HttpMessageConverter);
    }
}
~~~


