#### 一、介绍
  
  <br/>
  <br>
    

#### 二、Api说明
1. TclRouter  
    为对外提供调用的类，所有的操作包括初始化，具体功能调用等

    ``` java
    //Sdk初始化  
    TclRouter.init();

    //开启日志，在init()方法之前调用  
    TclRouter.openLog();

    //开启调试，在init()方法之前调用，因为有缓存，在调试的时候，如果不开启这个设置，涉及到组件有变更，会不能实时更新  
    TclRouter.openDebug();

    //根据组件的接口获取组件，适用于组件端提供了一种实现，不需要用路径区分  
    //当找不到实现类或不需要实现时，会得到一个方法函数都是空实现的实现类  
    TclRouter.getInstance().getComponent(TclComponentAService.class);

    //根据组件的接口和路径获取组件，适用于组件端提供了多种实现，需要用路径区分  
    //当找不到实现类或不需要实现时，会得到一个方法函数都是空实现的实现类  
    TclRouter.getInstance().getComponent(TclComponentAService.class,"/componentA/serviceImpl");

    //设置不需要实现的组件-设置单个  
    TclRouter.getInstance().setUnableComponent(TclComponentAService.class);

    //设置不需要实现的组件-设置多个  
    TclRouter.getInstance().setUnableComponents(TclComponentAService.class,TclComponentBService.class);

    //设置不需要实现的组件-用列表形式设置多个  
    TclRouter.getInstance().setUnableComponentList(componentList);

    //清空设置了不需要实现的组件  
    TclRouter.getInstance().clearUnableComponentList();
    ```
2. _TclRouter  
    为内部功能实现类，TclRouter提供的方法的实现，都由_TclRouter来实现

3. Warehouse  
    为路由数据的储存类

4. ITclComponent  
    为定义组件的接口类，所有组件都要继承这个类

    方法：init()，当获取组件时会调用

5. @interface Route  
    路由注解，用于组件接口的实现类

    path:定义组件实现类的路径，至少需要有两级，/xx/xx；必须要赋值

6. @interface Autowired  
    自动获取组件接口的实现类的注解，需要和注入方法TclRouter.getInstance().inject(this)一起使用

    path:定义组件实现类的路径，当组件实现类只有一个路径时，可以不赋值，会自动找到对应的实现类

7. @interface DefaultValue  
    默认值注解，用于interface接口类的方法上，当没有实现类时，可以对于以下几种返回类型的方法设置默认返回值  
    String  
    8种基本数据类型(byte、short、int、long、float、double、char、boolean)  
    8种基本数据类型的包装类(Byte、Short、Integer、Long、Float、Double、Character、Boolean)  

    
    例子：
    ``` java
    public interface TclComponentAService extends ITclComponent {
        void method1();
    
        @DefaultValue(stringValue = "value1")
        String getValue1();
    
        IClassA getClassA();
    
        AbsClassC getClassC();
    
        @DefaultValue(intValue = -1)
        int getIntValue() ;
        
        @DefaultValue(intValue = -1)
        Integer getIntegerValue() ;        
    
        @DefaultValue(byteValue = 1)
        byte getByteValue();
    
        @DefaultValue(doubleValue = 12.00d)
        double getDoubleValue();
    
        @DefaultValue(booleanValue = false)
        boolean getBooleanValue();
        
    }
    ```


#### 三、依赖和配置说明
1. 添加maven仓库

    ``` gradle
    allprojects {
        repositories {
            google()
            jcenter()
            maven {
                url 'http://10.92.246.65:8081/nexus/repository/maven-releases/'
            }

        }
    }
    ```

2. 添加依赖和配置
    ``` gradle
    //这个用于注解的自动生成代码，
    //与下方的annotationProcessor 'com.tcl.tclrouter:tclrouter-compiler: x.x.x'依赖一起，
    //组件的Api部分不需要这个配置
    android {
        defaultConfig {
            ...
            javaCompileOptions {
                annotationProcessorOptions {
                    arguments = [TCLROUTER_MODULE_NAME: project.getName()]
                }
            }
        }
    }

    dependencies {
        implementation 'com.tcl.tclrouter:tclrouter-api:x.x.x'

        //这个用于注解的自动生成代码，与上方的配置一起，组件的Api部分不需要这个依赖
        annotationProcessor 'com.tcl.tclrouter:tclrouter-compiler:x.x.x'
        ...
    }
    ```

#### 四、组件端
分为2个module，1个是Api，1个是组件的实现  
1. Api部分、
    定义组件对外接口，并继承ITclComponent，对于有返回值的自定义类，都必须是接口interface或抽象类  
    ``` java
    /**
     * 定义组件对外接口，并继承ITclComponent
     * 对于有返回值为自定义类的方法，为接口类或者抽象类
     */
    public interface TclComponentAService extends ITclComponent {
        void method1();

        @DefaultValue(stringValue = "value1")
        String getValue1();

        IClassA getClassA();

        AbsClassC getClassC();

        @DefaultValue(intValue = -1)
        int getIntValue() ;

        @DefaultValue(booleanValue = true)
        boolean getBooleanValue();
        ...
    }
    ```

2. 组件实现部分
    ``` java
     //这里的路径需要注意的是至少需要有两级
     @Route(path = "/componentA/serviceImpl")
     public class TclComponentAServiceImpl implements TclComponentAService {
         ...
     }
    ```

#### 五、组件调用端

1. 初始化组件SDK，在Application初始化

    ``` java
        if (isDebug()) {//openLog和openDebug需要在init之前
             TclRouter.openLog();
             TclRouter.openDebug();//因为有缓存，在调试的时候，如果不开启这个设置，涉及到组件有变更，会不能实时更新
        }
    ```

2. 依赖组件

    ``` gradle
    dependencies {
        //组件和组件Api根据具体需求，选择依赖其中1个
          
        //组件
        implementation 'com.tcl.tclrouter.demo:componentA:0.0.1'

        //组件的Api
        //implementation 'com.tcl.tclrouter.demo:componentA-api:0.0.2'
        ...
    }
    ```

3. 设置不想要实现的组件  运行时禁用

    ``` java
        //方式一：设置单个
        TclRouter.getInstance().setUnableComponent(TclComponentAService.class);

        //方式二：设置多个
        TclRouter.getInstance().setUnableComponents(TclComponentAService.class,TclComponentBService.class);

        //方式三：用列表形式设置多个
        List<Class> componentList = new ArrayList<>();
        componentList.add(TclComponentAService.class);
        componentList.add(TclComponentBService.class);
        TclRouter.getInstance().setUnableComponentList(componentList);
    ```


3. 获取组件  
    当不需要组件实现时，会得到一个方法函数都是空实现的实现类；  
    对于返回值为interface的自定义类方法，也会得到一个方法函数都是空实现的实现类  
    对于返回值为String或8大基本数据类型及其包装类的方法，可以通过@DefaultValue设置默认值，不如不设置，则为Sdk设置的默认值
    ``` java
        //方式一：当组件只有一种实现时，不指定路径，会默认找到该实现
        //需要先调用TclRouter.getInstance().inject(this)
        @Autowired
        public TclComponentAService tclComponentAService1;

        //方式二：当组件有多种实现时，需要指定路径path
        //需要先调用TclRouter.getInstance().inject(this)
        @Autowired(path = "/componentA/serviceImpl")
        public TclComponentAService tclComponentAService2;

        //方式三：当组件只有一种实现时，不指定路径，会默认找到该实现
        TclComponentAService tclComponentAService3 = TclRouter.getInstance().getComponent(
                TclComponentAService.class);

        //方式四：当组件有多种实现时，需要指定路径path
        TclComponentAService tclComponentAService4 = TclRouter.getInstance().getComponent(
                TclComponentAService.class,"/componentA/serviceImpl");
    ```

