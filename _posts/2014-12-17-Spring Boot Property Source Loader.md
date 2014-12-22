---
layout: post
category : howto
tagline: "Decrypt properties on the fly .."
tags : [spring]
---
{% include JB/setup %}

![propertySourceLoader.png]({{ site.url }}/assets/images/propertySourceLoader.png)

```java
    
package hello;

import org.springframework.boot.env.PropertySourceLoader;
import org.springframework.core.PriorityOrdered;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PropertiesLoaderUtils;

import java.io.IOException;
import java.util.Properties;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class EncryptedPropertySourceLoader implements PropertySourceLoader, PriorityOrdered {

    private static Pattern encryptedPropertyPattern = Pattern.compile("^ENC\\((?<content>.*?)\\)$");

    public EncryptedPropertySourceLoader(){
        //TODO: this could be taken from an environment variable
    }

    @Override
    public String[] getFileExtensions(){
        return new String[]{"properties"};
    }

    @Override
    public PropertySource<?> load(final String name, final Resource resource, final String profile) throws IOException {
        if (profile == null){
            final Properties props = PropertiesLoaderUtils.loadProperties(resource);

            if (!props.isEmpty()){
                return new PropertySource(name, props){
                    @Override
                    public Object getProperty(String s) {
                        if(props.containsKey(s)) {
                            Object value = props.getProperty(s);
                            if(value instanceof String){
                                String strValue = (String) value;
                                Matcher matcher =  encryptedPropertyPattern.matcher(strValue);
                                boolean isEncrypted = matcher.matches();
                                if(isEncrypted){
                                    String content = matcher.group("content");
                                    return "Decrypted from props" + content;
                                }
                            }
                            return value;
                        }
                        return null;
                    }
                };
            }
        }
        return null;
    }

    @Override
    public int getOrder() {
        return HIGHEST_PRECEDENCE;
    }
}
```
