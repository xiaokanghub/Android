# Android
Android 加固应用Hook方式-Frida
```
Java.perform(function () {
    var application = Java.use('android.app.Application');

    application.attach.overload('android.content.Context').implementation = function(context){

        var result = this.attach(context);
        var classloader = context.getClassLoader();
        Java.classFactory.loader = classloader;

        var yeyoulogin = Java.classFactory.use('com.zcm.主窗口');
        console.log("yeyoulogin:"+ yeyoulogin);


        yeyoulogin.按钮_用户登录$被单击.implementation = function(arg){
            console.log("retval:"+ this.返回值);
        }

    }
});
```
列出加载的类
```
Java.enumerateLoadedClasses(
  {
  "onMatch": function(className){ 
        console.log(className) 
    },
  "onComplete":function(){}
  }
)
```

# Hook 动态加载类
## 获取构造函数的参数
```
Java.perform(function(){
        //创建一个DexClassLoader的wapper
        var dexclassLoader = Java.use("dalvik.system.DexClassLoader");
        //hook 它的构造函数$init，我们将它的四个参数打印出来看看。
        dexclassLoader.$init.implementation = function(dexPath,optimizedDirectory,librarySearchPath,parent){
            console.log("dexPath:"+dexPath);
            console.log("optimizedDirectory:"+optimizedDirectory);
            console.log("librarySearchPath:"+librarySearchPath);
            console.log("parent:"+parent);
            //不破换它原本的逻辑，我们调用它原本的构造函数。
          this.$init(dexPath,optimizedDirectory,librarySearchPath,parent);
        }
        console.log("down!");

});
```
## 获取动态加载的类
```
Java.perform(function(){
        var dexclassLoader = Java.use("dalvik.system.DexClassLoader");
        var hookClass = undefined;
        var ClassUse = Java.use("java.lang.Class");

        dexclassLoader.loadClass.overload('java.lang.String').implementation = function(name){
           //定义一个String变量，指定我们需要的类
            var hookname = "cn.chaitin.geektan.crackme.MainActivityPatch";
            //直接调用第二个重载方法，跟原本的逻辑相同。
            var result = this.loadClass(name,false);
            //如果loadClass的name参数和我们想要hook的类名相同
            if(name === hookname){
                //则拿到它的值
                hookClass = result;
                //打印hookClass变量的值
                console.log(hookClass);
                send(hookClass);
                return result;
            }
            return result;
        }
});
```
## 通过Java.cast处理泛型方法(JAVA中Class<?>表示泛型)，在调用动态加载方法
```
Java.perform(function(){
        var hookClass = undefined;
        var ClassUse = Java.use("java.lang.Class");
        var dexclassLoader = Java.use("dalvik.system.DexClassLoader");
        var constructorclass = Java.use("java.lang.reflect.Constructor");
        var objectclass= Java.use("java.lang.Object");
        dexclassLoader.loadClass.overload('java.lang.String').implementation = function(name){
            var hookname = "cn.chaitin.geektan.crackme.MainActivityPatch";
            var result = this.loadClass(name,false);

            if(name == hookname){
                var hookClass = result;
                console.log("------------------------------CAST--------------------------------")
                //类型转换
                var hookClassCast = Java.cast(hookClass,ClassUse);
                //调用getMethods()获取类下的所有方法
                var methods = hookClassCast.getMethods();
                console.log(methods);
                console.log("-----------------------------NOT CAST-----------------------------")
                //未进行类型转换，看看能否调用getMethods()方法
                var methodtest = hookClass.getMethods();
                console.log(methodtest);
                console.log("---------------------OVER------------------------")
                return result;

            }
            return result;
        }


});
```
### 利用getDeclaredConstructor()获取具有指定参数列表构造函数的Constructor 并实例化
```
Java.perform(function(){
        var hookClass = undefined;
        var ClassUse = Java.use("java.lang.Class");
        var objectclass= Java.use("java.lang.Object");
        var dexclassLoader = Java.use("dalvik.system.DexClassLoader");
        var orininclass = Java.use("cn.chaitin.geektan.crackme.MainActivity");
        var Integerclass = Java.use("java.lang.Integer");
        //实例化MainActivity对象
        var mainAc = orininclass.$new();


        dexclassLoader.loadClass.overload('java.lang.String').implementation = function(name){
            var hookname = "cn.chaitin.geektan.crackme.MainActivityPatch";
            var result = this.loadClass(name,false);

            if(name == hookname){
                var hookClass = result;
                var hookClassCast = Java.cast(hookClass,ClassUse);
                console.log("-----------------------------BEGIN-------------------------------------");
                //获取构造器
                var ConstructorParam =Java.array('Ljava.lang.Object;',[objectclass.class]);
                var Constructor = hookClassCast.getDeclaredConstructor(ConstructorParam);
                console.log("Constructor:"+Constructor);
                console.log("orinin:"+mainAc);
                //实例化，newInstance的参数也是Ljava.lang.Object;
                var instance = Constructor.newInstance([mainAc]);
                console.log("patchAc:"+instance);
                send(instance);
console.log("--------------------------------------------------------------------");
                return result;

            }
            return result;
        }
});
```
### 利用getDeclaredMethods()，获取本类中的所有方法
```
Java.perform(function(){
        var hookClass = undefined;
        var ClassUse = Java.use("java.lang.Class");
        var objectclass= Java.use("java.lang.Object");
        var dexclassLoader = Java.use("dalvik.system.DexClassLoader");
        var orininclass = Java.use("cn.chaitin.geektan.crackme.MainActivity");
        var Integerclass = Java.use("java.lang.Integer");
        //实例化MainActivity对象
        var mainAc = orininclass.$new();


        dexclassLoader.loadClass.overload('java.lang.String').implementation = function(name){
            var hookname = "cn.chaitin.geektan.crackme.MainActivityPatch";
            var result = this.loadClass(name,false);

            if(name == hookname){
                var hookClass = result;
                var hookClassCast = Java.cast(hookClass,ClassUse);
                console.log("-----------------------------BEGIN-------------------------------------");
                //获取构造器
                var ConstructorParam =Java.array('Ljava.lang.Object;',[objectclass.class]);
                var Constructor = hookClassCast.getDeclaredConstructor(ConstructorParam);
                console.log("Constructor:"+Constructor);
                console.log("orinin:"+mainAc);
                //实例化，newInstance的参数也是Ljava.lang.Object;
                var instance = Constructor.newInstance([mainAc]);
                console.log("MainActivityPatchInstance:"+instance);
                send(instance);
                console.log("----------------------------Methods---------------------------------");
                var func = hookClassCast.getDeclaredMethods();
                console.log(func);
                console.log("--------------------------Need Method---------------------------------");
                console.log(func[0]);
                var f = func[0];
                console.log("---------------------------- OVER---------------------------------");
                return result;

            }
            return result;
        }
});
```
## 调用Method.invoke()去执行方法(invoke方法的第一个参数是执行这个方法的对象实例，第二个参数是带入的实际值数组，返回值是Object，也既是该方法执行后的返回值)
```
f.invoke(instance,Array);
```

## read-std-string
```
/*
 * Note: Only compatible with libc++, though libstdc++'s std::string is a lot simpler.
 */

function readStdString (str) {
  const isTiny = (str.readU8() & 1) === 0;
  if (isTiny) {
    return str.add(1).readUtf8String();
  }

  return str.add(2 * Process.pointerSize).readPointer().readUtf8String();
}
```

## whereisnative
```
Java.perform(function() {

    var SystemDef = Java.use('java.lang.System');

    var RuntimeDef = Java.use('java.lang.Runtime');

    var exceptionClass = Java.use('java.lang.Exception');

    var SystemLoad_1 = SystemDef.load.overload('java.lang.String');

    var SystemLoad_2 = SystemDef.loadLibrary.overload('java.lang.String');

    var RuntimeLoad_1 = RuntimeDef.load.overload('java.lang.String');

    var RuntimeLoad_2 = RuntimeDef.loadLibrary.overload('java.lang.String');

    var ThreadDef = Java.use('java.lang.Thread');

    var ThreadObj = ThreadDef.$new();

    SystemLoad_1.implementation = function(library) {
        send("Loading dynamic library => " + library);
        stackTrace();
        return SystemLoad_1.call(this, library);
    }

    SystemLoad_2.implementation = function(library) {
        send("Loading dynamic library => " + library);
        stackTrace();
        SystemLoad_2.call(this, library);
        return;
    }

    RuntimeLoad_1.implementation = function(library) {
        send("Loading dynamic library => " + library);
        stackTrace();
        RuntimeLoad_1.call(this, library);
        return;
    }

    RuntimeLoad_2.implementation = function(library) {
        send("Loading dynamic library => " + library);
        stackTrace();
        RuntimeLoad_2.call(this, library);
        return;
    }

    function stackTrace() {
        var stack = ThreadObj.currentThread().getStackTrace();
        for (var i = 0; i < stack.length; i++) {
            send(i + " => " + stack[i].toString());
        }
        send("--------------------------------------------------------------------------");
    }

})；
```


