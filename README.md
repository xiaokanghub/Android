# Android
Android 加固应用Hook方式-Frida
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

列出加载的类

Java.enumerateLoadedClasses(
  {
  "onMatch": function(className){ 
        console.log(className) 
    },
  "onComplete":function(){}
  }
)


