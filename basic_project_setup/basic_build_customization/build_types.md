# 构建类型

默认情况下，Android Plugin 会自动给项目构建 debug 和 release 版本。两个版本的区别在于能否在安全设备（非 dev）上调试，以及 APK 如何签名。debug 使用通过通用的 `name/password` 对生成的密钥证书进行签名（为了防止在构建过程中出现认证请求）。release 在构建过程中不进行签名，需要自行签名。

这些配置是通过 **<font color='green'>BuildType</font>** 对象来完成的。默认情况下，**<font color='green'>debug</font>** 和 **<font color='green'>release</font>** 实例都会被创建。Android plugin 允许像创建其他 *Build Type* 一样自定义这两种类型。在 **<font color='green'>buildTypes</font>** 的 DSL 容器中进行配置：

``` Groovy
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }

        jnidebug {
            initWith(buildTypes.debug)
            packageNameSuffix ".jnidebug"
            jnidebugBuild true
        }
    }
}
```

上面的代码片段实现了以下功能：

* 配置默认的 **<font color='green'>debug</font>** *Build Type*:
  * 设置包名为 `<app appliationId>.debug`，以便能够在同一个设备上安装 *debug* 和 *release* 版的 apk
* 创建了名为 **<font color='green'>jnidebug</font>** 的新 *BuildType*，并且以 **<font color='green'>debug</font>** 的配置作为默认配置。
* 继续配置 **<font color='green'>jnidebug</font>**，开启了 JNI 组件的 debug 功能，并添加了包名后缀。

可以在 **<font color='green'>buildTypes</font>** 标签下添加新的元素来创建新的 *Build Types* ，并且通过调用 **<font color='green'>initWith()</font>** 或者直接使用闭包来配置它。查看 [DSL Reference][1] 了解 *build type* 中可以配置的属性。

除了用于修改构建属性外，*Build Types* 还能用于添加特定的代码及资源。每个 Build Type 都会有匹配的 *sourceSet*。默认的路径为 `src/<buildtypename>/`，例如，`src/debug/java` 目录的资源只会编译于 debug APK 中。这意味着 *BuildType* 名称不能是 *main* 或者 *androidTest*（因为它们是由 plugin 内部实现的），并且他们都必须是唯一的。

跟其他 sourceSet 设置一样，Build Type 的 source set 路径也可以重新定向：

``` Grovvy
android {
    sourceSets.jnidebug.setRoot('foo/jnidebug')
}
```

另外，每个 *Build Type* 都会创建一个新的 `assemble<BuildTypeName>` 任务。  
例如：**<font color='green'>assembleDebug</font>**。当 **<font color='green'>debug</font>** 和 **<font color='green'>release</font>** 构建类型被预创建的时候，**<font color='green'>assembleDebug</font>** 和 **<font color='green'>assembleRelease</font>** 会被自动创建。同样的，上面提到的 *build.gradle* 代码片段中也会创建 **<font color='green'>assembleJnidebug</font>** task，并且 **<font color='green'>assemble</font>** 会像依赖 **<font color='green'>assembleDebug</font>** 和 **<font color='green'>assembleRelease</font>** 一样依赖于 **<font color='green'>assembleJnidebug</font>**。

> 提示：你可以在终端下输入 `gradle aJ` 去运行 **<font color='green'>assembleJnidebug</font>** task。

使用场景：

* debug 模式下才用到的权限
* 自定义的 debug 实现
* 为 debug 模式使用不同的资源（例如，当一个资源的值由绑定的证书决定）

*BuildType* 的代码和资源会在以下场景中被用到：

* manifest 将被合并到 app 的 manifest
* 代码只是换了一个源文件夹
* 资源将叠加到 main 的资源中，并替换已存在的资源

[1]: http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html