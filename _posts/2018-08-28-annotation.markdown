---
layout:         post
title:          @Transaction的一些问题
subtitle:       遇见可遇不可求的美好
card-image:     https://ww1.sinaimg.cn/mw690/906cb9dbgw1fayoxw9jh0j20b407e3zn.jpg
date:           2018-08-27 19:00:00
tags:           life
post-card-type: image
---
# annotation(@Target@Retention@Inherited@Documented)详解 #
 
## 注解：深入理解JAVA注解 ##

　　要深入学习注解，我们就必须能定义自己的注解，并使用注解，在定义自己的注解之前，我们就必须要了解Java为我们提供的元注解和相关定义注解的语法。
   
## 元注解（meta-annotation）：
    作用：负责注释其他注解，被用来提供对其他annotation类型进行说明元注解：
    @Retention
    @Inherited
    @Documented
    @Target    
1. @Target<br>
          
         作用：@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。
         枚举类：
               public enum ElementType {
                 /** 用于描述类、接口(包括注解类型) 或enum声明 */
                 TYPE,

                 /** 字段声明（包括枚举常量） */
                FIELD,

                /** 方法声明(Method declaration) */
                 METHOD,

                /** 正式的参数声明 */
                PARAMETER,

                /** 构造函数声明 */
                CONSTRUCTOR,

                /** 局部变量声明 */
                LOCAL_VARIABLE,

                /** 注释类型声明 */
                ANNOTATION_TYPE,

                /** 包声明 */
                PACKAGE,

                /**
                 * 类型参数声明
                *
                * @since 1.8
                */
                TYPE_PARAMETER,

                /**
                * 使用的类型
                *
                * @since 1.8
                */
                TYPE_USE
               }
           说明：注解只能在ElementType设定的范围内使用，否则将会编译报错。例如：范围只包含ElementType.METHOD ，则表明该注解只能使用在类的方法上，超出使用范围将编译异常。
    
2. @Retention<br>       
         
         作用：定义了改annotation被保存的时间长短，配置annotation生命周期。
         枚举类:RetentionPolicy:
                public enum RetentionPolicy {
                /**
                 * 注解仅保存在于源码中，在class字节码文件中不包含
                 * （仅仅出现在源代码中，被编译器丢弃）
                 * Annotations are to be discarded by the compiler.
                 */
                SOURCE,

                /**
                 * 默认使用，注解在class字码文件中存在，但运行时无法获取
                 * （被编译在class文件中，编译在class文件中但是会被虚拟机忽略）
                 * Annotations are to be recorded in the class file by the compiler
                 * but need not be retained by the VM at run time.  This is the default
                 * behavior.
                 */
                CLASS,

                /**
                 * 不仅在class字节文件中存在，在运行时可以通过反射获取到
                 * (在class被装在时进行被读取，不影响class的运行，annotation和class在使用时是分离的)
                 * Annotations are to be recorded in the class file by the compiler and
                 * retained by the VM at run time, so they may be read reflectively.
                 *
                 * @see java.lang.reflect.AnnotatedElement
                 */
                RUNTIME
                }
          实例：
               @Target(ElementType.FIELD)
               @Retention(RetentionPolicy.RUNTIME)
               public @interface Column {
                     public String name() default "fieldName";
                     public String setFuncName() default "setField";
                     public String getFuncName() default "getField"; 
                     public boolean defaultDBValue() default false;
               }
         Retention设置的是RUNTIME，注解处理器就可以通过反射获取改注解的属性值，进行一些逻辑处理

   
3.  @Inherited<br>  
        
         作用：标记注解，来申明被标记的类是能被继承的，如果一个被标记annotationl用于一个class，则这个annotation可以被用于该类的子类。
              如果@Inherited标记在使用RetentionPolicy.RUNTIME的annotation上，则反射会增强这种继承性，使用reflect去查询是，反射会检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。
         实例：
              @Target(ElementType.TYPE)
              @Retention(RetentionPolicy.RUNTIME)
              @Inherited
              public @interface Atest {
                public  String name () default "";
              }

              @Target(ElementType.TYPE)
              @Retention(RetentionPolicy.RUNTIME)
              public @interface Btest {
                public String name() default "";
              }
           
              @Atest
              public class Super {
                private int superx;
                public int supery;
                public Super() {
                }
                private int superX(){
                    return 0;
                }   
                public int superY(){
                return 0;
                }
              }

              @Btest
              public class Sub extends  Super {
                private int subx;
                public int suby;
                private Sub(){
                }
                public Sub(int i){
                }
                private int subX(){
                    return 0;
                }
                public int subY(){
                  return 0;
                 }
              }

              public static void main(String[] args) {
                 Class<Sub> clazz = Sub.class;

                System.out.println("============================Field===========================");
                System.out.println(Arrays.toString(clazz.getFields()));
                System.out.println(Arrays.toString(clazz.getDeclaredFields()));  //all + 自身
                System.out.println("============================Method===========================");
                System.out.println(Arrays.toString(clazz.getMethods()));   //public + 继承
                //all + 自身
                System.out.println(Arrays.toString(clazz.getDeclaredMethods()));
                System.out.println("============================Constructor===========================");
                System.out.println(Arrays.toString(clazz.getConstructors()));
                System.out.println(Arrays.toString(clazz.getDeclaredConstructors()));
                System.out.println("============================AnnotatedElement===========================");
                //注解DBTable2是否存在于元素上
                System.out.println(clazz.isAnnotationPresent(Btest.class));
                //如果存在该元素的指定类型的注释DBTable2，则返回这些注释，否则返回 null。
                 System.out.println(clazz.getAnnotation(Btest.class));
                //继承
                System.out.println(Arrays.toString(clazz.getAnnotations()));
                System.out.println(Arrays.toString(clazz.getDeclaredAnnotations()));  ////自身

              }
          运行结果：
              ============================Field===========================
                [public int com.tuhu.data.security.model.Sub.suby, public int com.tuhu.data.security.model.Super.supery]
                [private int com.tuhu.data.security.model.Sub.subx, public int com.tuhu.data.security.model.Sub.suby]
              ============================Method===========================
                [public int com.tuhu.data.security.model.Sub.subY(), public int com.tuhu.data.security.model.Super.superY(), public final void java.lang.Object.wait() throws java.lang.InterruptedException, public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException, public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException, public boolean java.lang.Object.equals(java.lang.Object), public java.lang.String java.lang.Object.toString(), public native int java.lang.Object.hashCode(), public final native java.lang.Class java.lang.Object.getClass(), public final native void java.lang.Object.notify(), public final native void java.lang.Object.notifyAll()]
                [public int com.tuhu.data.security.model.Sub.subY(), private int com.tuhu.data.security.model.Sub.subX()]
              ============================Constructor===========================
                [public com.tuhu.data.security.model.Sub(int)]
                [private com.tuhu.data.security.model.Sub(), public com.tuhu.data.security.model.Sub(int)]
              ============================AnnotatedElement===========================
                true
                @com.tuhu.data.security.annotation.Btest(name=)
                [@com.tuhu.data.security.annotation.Atest(name=), @com.tuhu.data.security.annotation.Btest(name=)]
                [@com.tuhu.data.security.annotation.Btest(name=)]
          
           getFields()获得某个类的所有的公共（public）的字段，包括父类。 
           getDeclaredFields()获得某个类的所有申明的字段，即包括public、private和proteced， 
           但是不包括父类的申明字段。 同样类似的还有getConstructors()和getDeclaredConstructors()， 
           getMethods()和getDeclaredMethods()。 

           因此：Field的打印好理解，因为sub是super类的子类，会继承super的类 
           同样method和constructor的打印也是如此。 

           clazz.getAnnotations()可以打印出当前类的注解和父类的注解 
           clazz.getDeclaredAnnotations()只会打印出当前类的注解 

           如果注解ATable把@Inherit去掉。无法获取到@Atest的注解， 
           也就是说注解和普通类的区别是如果一个子类想获取到父类上的注解信息， 
           那么必须在父类上使用的注解上面 加上@Inherit关键字
  
4.  @Documented<br> 
     
          作用：表明制作javadoc时，是否将注解信息加入文档。如果注解在声明时使用了@Documented，则在制作javadoc时注解信息会加入javadoc。
                @Documented
                @Retention(value=RUNTIME)
                @Target(value=ANNOTATION_TYPE)//说明该注解只能在声明注解时使用
                public @interface Documented {
                }
          


  

