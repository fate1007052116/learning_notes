# Assembly 打包插件使用方法

assembly是maven的打包插件，他的作用是可以帮助我们对jar（只能对jar项目打包，war项目可以直接用tomcat插件打包）项目做打包处理，需要使用Assembly打包插件来对项目做打包处理

因为Spring boot自带的maven打包插件可以顺带打包dubbo，但是普通的SpringFramework不能使用Springboot的打包插件，因此需要用到assembly打包插件

## 使用步骤

* 需要在项目最顶级根目录下创建一个目录，名叫assembly
* 将实例中的bin,conf 目录拷贝到asembly的根目录中
* 删除conf目录中dubbo.properties配置文件中的内容（dubbo.properties是一个空文件）
* 修改pom文件，添加assembly打包插件
* 在assembly目录下添加assembly.xml 配置文件
* 还需要修改start.sh