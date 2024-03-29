# Android
Android 加固应用Hook方式-Frida
```javascript
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
```javascript
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
```javascript
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
```javascript
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
```javascript
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
```javascript
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
```javascript
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
```javascript
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
```javascript
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

## Non-ASCII
```java
 int ֏(int x) {
        return x + 100;
    }
```
甚至有一些不可视, 所以可以先编码打印出来, 再用编码后的字符串去 hook.<\br>
```javascript
Java.perform(
        function x() {

            var targetClass = "com.example.hooktest.MainActivity";

            var hookCls = Java.use(targetClass);
            var methods = hookCls.class.getDeclaredMethods();

            for (var i in methods) {
                console.log(methods[i].toString());
                console.log(encodeURIComponent(methods[i].toString().replace(/^.*?\.([^\s\.\(\)]+)\(.*?$/, "$1")));
            }

            hookCls[decodeURIComponent("%D6%8F")]
                .implementation = function (x) {
                    console.log("original call: fun(" + x + ")");
                    var result = this[decodeURIComponent("%D6%8F")](900);
                    return result;
                }
        }
    )
```

## Hook 数据库
```javascript
var SQLiteDatabase = Java.use('com.tencent.wcdb.database.SQLiteDatabase');
    var Set = Java.use("java.util.Set");
    var ContentValues = Java.use("android.content.ContentValues");
    SQLiteDatabase.insert.implementation = function (arg1,arg2,arg3) {

        this.insert.call(this, arg1, arg2, arg3);
        console.log("[insert] -> arg1:" + arg1 + "\t arg2:" + arg2);
        var values = Java.cast(arg3, ContentValues);
        var sets = Java.cast(values.keySet(), Set);
        
        var arr = sets.toArray().toString().split(",");
        for (var i = 0; i < arr.length; i++){
            console.log("[insert] -> key:" + arr[i] + "\t value:" + values.get(arr[i]));
        }
    };
    
```
## Hook JNI Native GetStringUTFChars  
```javascript
function hook_native_GetStringUTFChars() {
    var env = Java.vm.getEnv();
    var handlePointer = Memory.readPointer(env.handle);
    console.log("env handle: " + handlePointer);
    var GetStringUTFCharsPtr = Memory.readPointer(handlePointer.add(0x2A4));
    console.log("GetStringUTFCharsPtr addr: " + GetStringUTFCharsPtr);
    Interceptor.attach(GetStringUTFCharsPtr, {
        onEnter: function (args) {
            var str = "";
            Java.perform(function () {
                str = Java.cast(args[1], Java.use('java.lang.String'));
            });
            console.log("GetStringUTFChars: " + str);

        }
    });
}
```  
## 主动弹窗
```javascript
Java.perform(function() {
    var System = Java.use('java.lang.System');
    var ActivityThread = Java.use("android.app.ActivityThread");
    var AlertDialogBuilder = Java.use("android.app.AlertDialog$Builder");
    var DialogInterfaceOnClickListener = Java.use('android.content.DialogInterface$OnClickListener');

    Java.use("android.app.Activity").onCreate.overload("android.os.Bundle").implementation = function(savedInstanceState) {
        var currentActivity = this;

        // Get Main Activity
        var application = ActivityThread.currentApplication();
        var launcherIntent = application.getPackageManager().getLaunchIntentForPackage(application.getPackageName());
        var launchActivityInfo = launcherIntent.resolveActivityInfo(application.getPackageManager(), 0);

        // Alert Will Only Execute On Main Package Activity Creation
        console.log(this.getComponentName().getClassName())
        /**
         * non protective application
         * if (launchActivityInfo === this.getComponentName().getClassName()) {
         *     ...
         * }
         */

        if (this.getComponentName().getClassName() === "com.xxx") {

            var alert = AlertDialogBuilder.$new(this);
            var jString = Java.use('java.lang.String');
            var CharSequence = Java.use('java.lang.CharSequence');
            var charSequence = Java.cast(jString.$new("What you want to do now?"), CharSequence);
            var charSequence1 = Java.cast(jString.$new("Dismiss"), CharSequence);
            var charSequence2 = Java.cast(jString.$new("Force Close!"), CharSequence);
            alert.setMessage(charSequence);

            alert.setPositiveButton(charSequence1, Java.registerClass({
                name: 'il.co.realgame.OnClickListenerPositive',
                implements: [DialogInterfaceOnClickListener],
                methods: {
                    getName: function() {
                        return 'OnClickListenerPositive';
                    },
                    onClick: function(dialog, which) {
                        // Dismiss
                        dialog.dismiss();
                    }
                }
            }).$new());

            alert.setNegativeButton(charSequence2, Java.registerClass({
                name: 'il.co.realgame.OnClickListenerNegative',
                implements: [DialogInterfaceOnClickListener],
                methods: {
                    getName: function() {
                        return 'OnClickListenerNegative';
                    },
                    onClick: function(dialog, which) {
                        // Close Application
                        //currentActivity.finish();
                        System.exit(0);
                    }
                }
            }).$new());

            // Create Alert
            alert.create().show();
        }
        return this.onCreate.overload("android.os.Bundle").call(this, savedInstanceState);
    };
});
```
# Hook prettyMethod
```javascript
const STD_STRING_SIZE = 3 * Process.pointerSize;
class StdString {
    constructor() {
        this.handle = Memory.alloc(STD_STRING_SIZE);
    }

    dispose() {
        const [data, isTiny] = this._getData();
        if (!isTiny) {
            Java.api.$delete(data);
        }
    }

    disposeToString() {
        const result = this.toString();
        this.dispose();
        return result;
    }

    toString() {
        const [data] = this._getData();
        return data.readUtf8String();
    }

    _getData() {
        const str = this.handle;
        const isTiny = (str.readU8() & 1) === 0;
        const data = isTiny ? str.add(1) : str.add(2 * Process.pointerSize).readPointer();
        return [data, isTiny];
    }
}

function prettyMethod(method_id, withSignature) {
    const result = new StdString();
    Java.api['art::ArtMethod::PrettyMethod'](result, method_id, withSignature ? 1 : 0);
    return result.disposeToString();
}
```

# bypass frida detection
`
result = pthread_create(v102, 0LL, sub_F74, 0LL);
`
```javascript
function hook_pthread_create(){
    var pt_create_func = Module.findExportByName(null,'pthread_create');
    var detect_frida_loop_addr = null;
    console.log('pt_create_func:',pt_create_func);
 
   Interceptor.attach(pt_create_func,{
       onEnter:function(){
           if(detect_frida_loop_addr == null)
           {
                var base_addr = Module.getBaseAddress('libnative-lib.so');
                if(base_addr != null){
                    detect_frida_loop_addr = base_addr.add(0x0000000000000F74)
                    console.log('this.context.x2: ', detect_frida_loop_addr , this.context.x2);
                    if(this.context.x2.compare(detect_frida_loop_addr) == 0) {
                        hook_anti_frida_replace(this.context.x2);
                    }
                }
 
           }
 
       },
       onLeave : function(retval){
           // console.log('retval',retval);
       }
   })
}
function hook_anti_frida_replace(addr){
    console.log('replace anti_addr :',addr);
    Interceptor.replace(addr,new NativeCallback(function(a1){
        console.log('replace success');
        return;
    },'pointer',[]));
 
}

setImmediate(hook_pthread_create());
```
# find svc
```javascript
let target_code_hex;
let call_number_openat;
let call_number_faccessat;
let arch = Process.arch;
let call_number_sendto;
let call_number_sendmsg;
let call_number_kill;
if ("arm" === arch){
    target_code_hex = "00 00 00 EF";
    call_number_openat = 322;
    call_number_faccessat = 334;
    call_number_sendto = 206;
    call_number_sendmsg = 221;
    call_number_kill = 129;
}else if("arm64" === arch){
    target_code_hex = "01 00 00 D4";
    call_number_openat = 56;
    call_number_faccessat = 48;
    call_number_sendto = 206;
    call_number_sendmsg = 221;
    call_number_kill = 129;

}else {
    console.log("arch not support!")
}

if (arch){
    console.log("\nthe_arch = " + arch);
    // 直接Process.enumerateModules()，可能会因为某些地址不可读造成非法访问
    Process.enumerateRanges('r--').forEach(function (range) {
        if(!range.file || !range.file.path){
            return;
        }
        let path = range.file.path;
        if ((!path.startsWith("/data/app/")) || (!path.endsWith(".so"))){
            return;
        }
        let baseAddress = Module.getBaseAddress(path);
        console.log("\npath = " + path + " , baseAddress = " + baseAddress + " , rangeAddress = " + range.base + " , size = " + range.size);

        Memory.scan(range.base, range.size, target_code_hex, {
            onMatch: function (match){
                let code_address = match;
                let code_address_str = code_address.toString();
                if (code_address_str.endsWith("0") || code_address_str.endsWith("4") || code_address_str.endsWith("8") || code_address_str.endsWith("c")){
                    console.log("--------------------------");
                    let call_number = 0;
                    if ("arm" === arch){
                        // call_number = (code_address.sub(0x4).readS16() - 28672);  // 0x7000
                        call_number = (code_address.sub(0x4).readS32()) & 0xFFF;
                    }else if("arm64" === arch){
                        call_number = (code_address.sub(0x4).readS32() >> 5) & 0xFFFF;
                    }else {
                        console.log("the arch get call_number not support!")
                    }
                    console.log("find svc : address = " + code_address + " , call_number = " + call_number + " , offset = " + code_address.sub(baseAddress));

                    // hook svc __NR_openat
                    if (call_number_openat === call_number){
                        let target_hook_addr = code_address;
                        let target_hook_addr_offset = target_hook_addr.sub(baseAddress);
                        console.log("find svc openat , start inlinehook by frida!")
                        Interceptor.attach(target_hook_addr, {
                            onEnter: function (args){
                                console.log("\nonEnter_" + target_hook_addr_offset + " , __NR_openat , args[1] = " + args[1].readCString());
                                this.new_addr = Memory.allocUtf8String("/proc/self/status11");
                                args[1] = this.new_addr;
                                console.log("onEnter_" + target_hook_addr_offset + " , __NR_openat , args[1] = " + args[1].readCString());
                            }, onLeave: function (retval){
                                console.log("onLeave_" + target_hook_addr_offset + " , __NR_openat , retval = " + retval)
                            }
                        });

                    }
                    // hook svc __NR_faccessat
                    if (call_number_faccessat === call_number){
                        let target_hook_addr = code_address;
                        let target_hook_addr_offset = target_hook_addr.sub(baseAddress);
                        console.log("find svc faccessat , start inlinehook by frida!")
                        Interceptor.attach(target_hook_addr, {
                            onEnter: function (args){
                                console.log("\nonEnter_" + target_hook_addr_offset + " , __NR_faccessat , args[1] = " + args[1].readCString());
                                // this.new_addr = Memory.allocUtf8String("/proc/self/status11");
                                // args[1] = this.new_addr;
                                console.log("onEnter_" + target_hook_addr_offset + " , __NR_faccessat , args[1] = " + args[1].readCString());
                            }, onLeave: function (retval){
                                console.log("onLeave_" + target_hook_addr_offset + " , __NR_faccessat , retval = " + retval)
                            }
                        });

                    }
                }
            }, onComplete: function () {}
        });

    });
}
```
# USB Debugging Bypass
```javascript
function USBDebuggingBypass() {
	var Secure = Java.use('android.provider.Settings$Secure');
	var System = Java.use('android.provider.Settings$System');
	var Global = Java.use('android.provider.Settings$Global');
	
	Secure.getInt.overload('android.content.ContentResolver', 'java.lang.String', 'int').implementation = function(arg1, arg2, arg3) {
		if(arg2.indexOf('adb_enabled') !== -1) {
			console.warn('[!] USB Debugging Check Bypass !');
			return 0;
		} else {
			return this.getInt(arg1, arg2, arg3);
		}
	}
	System.getInt.overload('android.content.ContentResolver', 'java.lang.String', 'int').implementation = function(arg1, arg2, arg3) {
		if(arg2.indexOf('adb_enabled') !== -1) {
			console.warn('[!] USB Debugging Check Bypass !');
			return 0;
		} else {
			return this.getInt(arg1, arg2, arg3);
		}
	}
	Global.getInt.overload('android.content.ContentResolver', 'java.lang.String', 'int').implementation = function(arg1, arg2, arg3) {
		if(arg2.indexOf('adb_enabled') !== -1) {
			console.warn('[!] USB Debugging Check Bypass !');
			return 0;
		} else {
			return this.getInt(arg1, arg2, arg3);
		}
	}

	var Debug = Java.use('android.os.Debug');
	Debug.isDebuggerConnected.implementation = function() {
		console.warn('[*] Debug.isDebuggerConnected() Bypass !');

		return false;
	}
}

```
# hook read & open
```javascript
var TraceSysFD = {};
function prettyLog(str) {
    console.log("---------------------------\n" + str);
}

Interceptor.attach(Module.findExportByName(null, "read"), {
    // fd, buff, len
    onEnter: function (args) {
        var bfr = args[1], sz = args[2].toInt32();
        var path = (TraceSysFD["fd-" + args[0].toInt32()] != null) ? TraceSysFD["fd-" + args[0].toInt32()] : "[unknow path]";
        prettyLog("[Libc::read] Read FD (" + path + "," + bfr + "," + sz + ")\n");
    },
    onLeave: function (ret) {
    }
});
Interceptor.attach(Module.findExportByName(null, "open"), {
    // path, flags, mode
    onEnter: function (args) {
        this.path = args[0].readCString();
    },
    onLeave: function (ret) {
        TraceSysFD["fd-" + ret.toInt32()] = this.path;
        prettyLog("[Libc::open] Open file '" + this.path + "' (fd: " + ret.toInt32() + ")");
    }
});

```

# hook readlink
```javascript
var aaa,bbb,ccc;
Interceptor.attach(Module.findExportByName(null, "readlink"),{
    onEnter: function(args){
        aaa = args[0];
        bbb = args[1];
        ccc = args[2];
        },
    onLeave: function(retval){
        // console.log('\nreadlink(' + 's1="' + aaa.readCString() + '"' + ', s2="' + bbb.readCString() + '"' + ', s3="' + ccc + '"' + ')');
        if(bbb.readCString().indexOf("frida")!==-1 ||
            bbb.readCString().indexOf("gum-js-loop")!==-1||
            bbb.readCString().indexOf("gmain")!==-1 ||
            bbb.readCString().indexOf("tmp")!==-1 ||
            bbb.readCString().indexOf("linjector")!==-1){
            console.log('\nreadlink(' + 's1="' + aaa.readCString() + '"' + ', s2="' + bbb.readCString() + '"' + ', s3="' + ccc + '"' + ')');
            bbb.writeUtf8String("/system/framework/boot.art")
            //console.log("replce with: "+bbb.readCString())
            retval.replace(0x1A)
            //console.log("retval: "+retval)
        }
    }
});

```

# hook strstr
```javascript
var strstr = Module.findExportByName(null, "strstr");
if (null !== strstr) {
    Interceptor.attach(strstr, {
        onEnter: function (args) {
            this.frida = Boolean(0);

            this.haystack = args[0];
            this.needle = args[1];

            if (this.haystack.readCString() !== null && this.needle.readCString() !== null) {
                if (this.haystack.readCString().indexOf("frida") !== -1 ||
                    this.needle.readCString().indexOf("frida") !== -1 ||
                    this.haystack.readCString().indexOf("gum-js-loop") !== -1 ||
                    this.needle.readCString().indexOf("gum-js-loop") !== -1 ||
                    this.haystack.readCString().indexOf("gmain") !== -1 ||
                    this.needle.readCString().indexOf("gmain") !== -1 ||
                    this.haystack.readCString().indexOf("linjector") !== -1 ||
                    this.needle.readCString().indexOf("linjector") !== -1) {
                    this.frida = Boolean(1);
                }
            }
        },
        onLeave: function (retval) {
            if (this.frida) {
                retval.replace(ptr("0x0"));
            }

        }
    })
    console.log("anti anti-frida");
}

```

# hook strcmp
```javascript
var strcmp = Module.findExportByName(null,"strcmp");
console.log("find strcmp:",strcmp);
Interceptor.attach(strcmp, {
    onEnter: function (args) {
            if(ptr(args[1]).readCString().indexOf("frida")>=0){
                // ptr(args[1]).writeUtf8String('fuck u');
                console.log("[*] strcmp (" + ptr(args[0]).readCString() + "," + ptr(args[1]).readCString()+")");
                
                this.ishack = true;
            }
        
    },
    onLeave: function(retval){
        if(this.ishack){
            retval.replace(ptr("0x0"))
            console.log("the ishack's result :",retval);
        }
        
    }
});

```
