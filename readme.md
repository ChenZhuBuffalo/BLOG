
# 基于LEGO模型对Spring IoC 应用的理解 
> 永远不要在你的程序内直接生成程序所依赖的对象，而是将其作为一个外部实体进行描述

####  一些相关概念
**Spring:** 一种轻量级的java框架
**Spring IoC(Inversion of Control):** 一种软件设计原则：框架或者外部实体应该代替开发者去控制程序的运行
**DI(Dependency Injection):**  实现控制反转的一种模式

我们先看几个需求场景：


### 场景一：一辆 LEGO REVOR的JAVA  实现

```java
package Vehicles;  
  
import Accessories.GeneralAccessories;  
import Frame.CATFrame;  
import Motor.TOYOTAMotor;  
import Tires.MichelinTires;  
  
public class RoverCar {  
    public void makeFrame(){  
        CATFrame CATFrame = new CATFrame();  
        CATFrame.fix();  
    }  
  
    public void makeMotor(){  
        TOYOTAMotor TOYOTAMotor = new TOYOTAMotor();  
        TOYOTAMotor.run();  
    }  
  
    public void makeTires (){  
        MichelinTires michelinTires = new MichelinTires();  
        michelinTires.rotate();  
    }  
  
    public void makeAccessories(){  
        GeneralAccessories generalAccessories = new GeneralAccessories();  
        generalAccessories.assemble();  
    }  
}
```

```java
  
public class MichelinTires {  
    public void rotate() {  
        System.out.println("rotation");  
    }  
}


  
public class TOYOTAMotor {  
    public void run() {  
        System.out.println("run");  
    }  
}


  
public class CATFrame {  
    public void fix() {  
        System.out.println("fixation");  
    }  
}


  
public class GeneralAccessories {  
    public void assemble(){  
        System.out.println("assembled");  
    }  
}
```

这时我们已经通过java搭建了一个简单的LEGO小车，它拥有类似Tire, Motor,Frame等各种配件，可以实现基本的功能。我们只需要在`RoverCar` 中创建这些配件，并调用相应的方法，就可以让这辆小车运行。但此时需求有了改变：

### 场景二：需要更换配件使车辆满足某些需求，如越障
> 在 `RoverCar` 中直接更换配件Class, 如将`TOYOTAMotor` 更换为 `GeneralMotor`

> 将配件Class更换为自定义配件，在车辆对象中使用

```java
package Vehicles;  
  
import Accessories.CustomizedAccessories;  
import Frame.CustomizedFrame;  
import Motor.CustomizedMotor;  
import Tires.CustomizedTires;  
  
public class CustomizedLEGOVehicle {  
  
    public void makeTires(){  
        CustomizedTires customizedTires = new CustomizedTires("Michelin");  
        customizedTires.rotate();  
    }  
  
    public void makeMotor(){  
        CustomizedMotor customizedMotor = new CustomizedMotor("TOYOTA");  
        customizedMotor.run();  
    }  
  
    public void makeFrame() {  
        CustomizedFrame customizedFrame = new CustomizedFrame("CAT");  
        customizedFrame.fixed();  
    }  
  
    public void makeAccessories(){  
        CustomizedAccessories customizedAccessories = new CustomizedAccessories("General");  
        customizedAccessories.assemble();  
    }  
  
}
```
```java
package Frame;  
  
public class CustomizedFrame {  
    private String frameName;  
  
    public CustomizedFrame(String frameName) {  
        this.frameName = frameName;  
    }  
  
    public String getFrameName() {  
        return frameName;  
    }  
  
    public void setFrameName(String frameName) {  
        this.frameName = frameName;  
    }  
  
    public void fixed() {  
        System.out.println("fixation");  
    }  
}
```
（其他配件用同样的写法）

上述方案存在的问题：
1. 每更换一次配件，都需要重新新建一个单独的配件对象，如米其林轮胎和固特异轮胎
2. 对上述问题进行改进后，使用自定义轮胎，不需要新建单独的配件对象，但也需要在方法中对其进行初始化
3. 无论使用那种方法，都是因为需求的改变，需要在更换配件时，联系乐高公司生产新的所需的配件，时间和人力成本较高。如果其他模型想使用相同的配件，需要再次联系乐高公司重新生产

#### 用Spring IoC 来处理这个问题

**Beans 就像它的名字一样，可以理解为咖啡盒子中的咖啡豆，是独立且可以被随时取用的**

##### Step 1
使用注释`@Compoment`将每个配件都作为一个Bean创建，比如`Motor`
（此处使用`@Component`而非`@Bean` 是因为前者可以针对整个对象而非单个方法）
```java
@Component  
public class CustomizedMotor {  
    private String motorName;  
  
    public CustomizedMotor(String motorName) {  
        this.motorName = motorName;  
    }  
  
    public String getMotorName() {  
        return motorName;  
    }  
  
    public CustomizedMotor() {  
    }  
  
    public void setMotorName(String motorName) {  
        this.motorName = motorName;  
    }  
  
    public void run(){  
        System.out.println("run");  
    }  
}
```

###### Step 2
创建`VehicleService` 对象，在其中将各个配件使用`Autowired` 注入，并实现方法（此时注入的是在应用运行时已在容器中启动好的Beans,不需要在此对象中重新创建）
```java
@Component  
public class VehicleService {  
    @Autowired  
    private CustomizedFrame customizedFrame;  
    @Autowired  
    private CustomizedMotor customizedMotor;  
    @Autowired  
    private CustomizedAccessories customizedAccessories;  
    @Autowired  
    private CustomizedTires customizedTires;  
  
    public void fix( String name) {  
        customizedFrame.setFrameName(name);  
        customizedFrame.fixed();  
    }  
  
    public void assemble(String name) {  
        customizedAccessories.setAccessoriesName(name);  
        customizedAccessories.assemble();  
    }  
  
    public void run(String name) {  
        customizedMotor.setMotorName(name);  
        customizedMotor.run();  
    }  
  
    public void rotate(String name) {  
        customizedTires.setTireName(name);  
        customizedTires.rotate();  
    }  
  
}
```

###### Step 3
将`VehicleService` 注入`Vehicle` 对象，并为其设置`get` 方法
```java
@Component  
public class Vehicle {  
    private final VehicleService vehicleService;  
  
    @Autowired  
    public Vehicle(VehicleService vehicleService) {  
        this.vehicleService = vehicleService;  
    }  
  
    public VehicleService getVehicleService() {return vehicleService;}  
}
```

###### Step 4
创建`ProjectConfig` 配置类，使用注释`@Configuration` 和`ComponentScan` 告知容器需要在运行时启动的Beans（这一步即告知容器需要装入那些Beans）
```java
@Configuration  
@ComponentScan(basePackages = "com.example.springioc_lego")  
@ComponentScan(basePackageClasses = {com.example.springioc_lego.Vehicle.class})  
public class ProjectConfig {  
  
}
```

###### Step 5
在启动类中调用相应的方法并运行
```java
@SpringBootApplication  
public class SpringIoCLegoApplication {  
  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ProjectConfig.class);  
        Vehicle vehicle = context.getBean(Vehicle.class);  
        vehicle.getVehicleService().assemble("CAT");  
        vehicle.getVehicleService().run("TOYOTA");  
        vehicle.getVehicleService().fix("General");  
        vehicle.getVehicleService().rotate("GoodYear");  
    }  
  
}

```

##### 通俗描述实现过程
1. 乐高公司收集用户对模型需求的描述，根据描述提前准备好生产配件所需的模具（对应创建Beans）
2. 用户组装模型时，乐高公司可直接使用准备好的模具进行生产。当用户需要改装时，则直接从生产好的配件中挑选，组装(对应将Beans注入)
3. 当其他模型需要相同的配件时，可以直接再次取用，且根据需求通过依赖注入自行调整各个对象之间的依赖关系，无需让公司重复生产配件，故对模型的控制权由用户专为乐高公司本身

>基于 Spring IoC原则，使用DI模式，在此应用中，所被依赖的各个配件不再在应用运行时被创建，而是由容器提前创建好，在需要时可以被直接使用。实现了各个对象之间的松耦合关系。

