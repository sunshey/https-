我们生活在网络信息时代，可以说我们使用的绝大多数设备特别是手机、电脑都离不开网络请求。抓包是一种可以窥探设备发出的网络请求，并且对网络请求作出拦截和修改的技术。

也许有童鞋一头雾水，我打个比方：我们的大脑是通过神经元之间的电信号来传递信息的，在万物互联的现实世界中，我们可以简单地把把网络请求类比为神经元之间的信息传递，如果把这个“任意修改网络请求”这个能力类比到大脑，那就是我们可以任意控制大脑的思想。所以说，抓包是一项非常强大的技术，它可以实现很多黑科技。

对 Android 手机进行抓包的方法有很多，大体列举一下有下面这么几种：

1.在局域网的电脑上建代理，然后设置手机的 WiFi 通过电脑的代理上网，进而在电脑上抓包。这种方式非常方便，如果是开发场景，它通常是我的首选项。不过麻烦的地方在于，它需要额外一台电脑，不是所有场景都适用。
2.通过 Xposed 插件或者 Frida 来 HOOK 特定的网络请求方法；这种方法针对性较强，需要编写特定的插件对目标应用进行处理，普通人一般无法使用。
3.通过 Linux 内核的 eBPF 功能过滤进行抓包，这种方法功能非常强大，但是对 Android 设备的内核版本有要求，使用起来也较为麻烦，通常都是开发人员使用。
4.在手机上建立 VPN，通过 VPN 进行抓包。这种方法比较通用，适合普通人使用。

随着 https 的普及，我们在抓包的过程中一般还会遇到证书问题；有些应用使用了固定证书，我们可能会发现即使我们在系统上安装了自定义证书，某些 https 请求依然抓不到。由于种种限制的存在，我们通常都需要手机拥有 ROOT 权限来绕过这些限制；而 ROOT 本身，也是一种限制。

今天我来给大家介绍一种相对简单的方法来进行抓包，它无需 ROOT 权限，也不需要借助其他设备，在手机上就可以进行，并且还能一定程度上修改网络请求；虽然它无法解决所有的问题，但是对普通人来说足以应对大部分场景。


首先，我们需要准备四款 App：两仪、HttpCanary、太极和 JustTrustMe。两仪 和 HttpCanary 来配合进行免 ROOT 抓包，太极和 JustTrustMe 来解决固定证书导致部分 https 请求抓不到的问题。PS. 后台回复 抓包 可以获取这四款 App 的下载链接。

在手机上安装好这四款 App 以后，我们打开 HttpCanary，它会提醒我们建立 VPN 服务，此时我们选择“允许”，这个 VPN 的作用就是让所有的网络请求都通过 HttpCanary 过一道，这时候我们就有机会对网络包进行处理了。为了避免请求太多导致分析困难，我们最好设置一下仅针对两仪这个 App 进行抓包，在这个 App 的设置可以找到此选项。

打开 HttpCanary 以后，它会提示我们需要安装自定义的证书；如果没有安装证书，我们是无法抓到 https 的请求的；在 Android 10 以前的设备上，我们可以直接点击安装证书；如果你是更高的系统版本，需要在设置中搜索“安装证书”，然后选择我提供的 HttpCanary.pem 文件进行安装。

安装好证书并且在 HttpCanary 中开启抓包服务以后，我们打开两仪，然后导入需要被抓包的目标应用，最后在两仪中打开此应用即可；此时，我们就可以在 HttpCanary 中看到目标应用发出的网络请求，这时候，一个简单的抓包过程就成功了。

很多情况下我们使用上面的方法就足够了，如果你碰到有些请求抓不到，可以按照接下来的步骤继续下去。我们需要通过一个 Xposed 插件来解除固定证书的限制，恰好，两仪中我们可以通过太极使用 Xposed 插件。

首先，我们在两仪中导入太极 App 和 JustTrustMe。

然后打开太极，点击右下角的悬浮窗选择“安装太极”，安装完毕以后按照提示重启两仪（杀掉 App 再次打开即可）：

![640](https://github.com/sunshey/https-/assets/16149870/b59bfa88-db0c-4718-a195-f1cc35a5dcd8)

在重启完两仪以后，我们再次打开太极，点击右下角的悬浮按钮，在弹出的菜单里面选择“模块管理“，进入模块管理页面后，勾选 “JustTrustMe” 这个 App。

最后，我们回到太极的主界面，点击右下角悬浮按钮，在弹出的菜单里选择“添加应用”，将我们需要抓包的目标应用添加到太极。此时，我们的操作就全部完成了。

有童鞋可能有疑问，怎么一会儿要导入到两仪，完事了还要加入到太极，为什么这么麻烦？简单解释一下就是，两仪的作用就是让你能通过 HttpCanary 抓到两仪内部的网络请求，太极的作用是通过 JustTrustMe 这个插件解除应用内的固定证书限制，从而能抓到之前抓不到的 https 请求。

在太极设置好以后，我们再次打开 HttpCanary 和两仪内部的目标应用，此时我们就会发现那些 https 请求也能获取到了。到这里，其实抓包的教程也已经结束了；不过，我们在手机上抓包，由于随时随地可以进行，所以如果能修改网络请求，能完成的事会更多；因此这里也简单介绍一下怎么在手机上修改抓包的请求和响应。

我们在 HttpCanary 中抓到特定的网络请求以后，长按这个请求，会弹出一个菜单：

![640 (1)](https://github.com/sunshey/https-/assets/16149870/42f124b7-2a58-449b-8102-9214ab4f79b5)


点击“静态注入”，会引导我们创建一个静态注入器，在注入器的编辑界面，我们可以修改这个网络请求的请求和响应，包括请求头和请求体、响应头和响应体：

![640 (2)](https://github.com/sunshey/https-/assets/16149870/e248f06b-cdaa-4ec5-aecc-5620a602a390)


我们编辑相应的字段保存，就设置好了一个静态注入器；下次 App 发起相同的网络请求时，就会以我们修改过的请求发出，在响应回来以后，同样会修改为我们设置好的网络请求
