---
title: Spring @Autowire注入Map和List
date: 2019-10-17 22:55:23
tags:
    - Java
    - 框架
    - Spring
---
今天要实现一个需求，我们要同时使用阿里云和白山云的对象存储，在上传或者删除一些文件时，两个对象存储都要做同样的操作。由于是不同的平台，配置、SDK什么的都不一样，因此我写了一个接口让它们去实现，然后我再在service调用bean的方法去操作文件。本来还想用@Autowire和@Qualifier分别注入这两个bean再塞到一个list去调用它们，然后突然发现@Autowired能直接注入Map或List，把实现该接口的所有的bean塞到一个集合中，实在是太棒太灵活了！
![太棒了](https://img-blog.csdnimg.cn/20201114184730276.jpg)

那么就不多BB了，直接上代码。
### 接口

```
public interface Animal {
    void sayHello();
}
```

### 实现类

```
@Component
public class Cat implements Animal {
    @Override
    public void sayHello() {
        System.out.println("Hello, this is " + this.getClass().getSimpleName());
    }
}
```

```
@Component
public class Dog implements Animal {
    @Override
    public void sayHello() {
        System.out.println("Hello, this is " + this.getClass().getSimpleName());
    }
}
```

### 自动注入
可以注入Map或者List，指定泛型是那个接口类型即可。下面是测试的代码

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class AnimalTest {
    @Autowired
    private List<Animal> animalList;
    @Autowired
    private Map<String, Animal> animalMap;

    @Test
    public void contextLoads() {
        for (Animal animal : animalList) {
            animal.sayHello();
        }

        Set<String> keys = animalMap.keySet();
        for (String key : keys) {
            animalMap.get(key).sayHello();
        }

    }
}
```

### 测试
断点看了一下变量，可以看到在Map里面，key就是bean的名字。
![测试断点](https://img-blog.csdnimg.cn/2020111418481998.jpg)

下面是运行结果。
![运行结果](https://img-blog.csdnimg.cn/2020111418481978.jpg)