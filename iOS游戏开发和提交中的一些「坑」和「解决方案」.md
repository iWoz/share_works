@(博客文章)
#iOS游戏开发和提交中的一些「坑」和「解决方案」

鉴于在iOS类的游戏开发和提交审核的过程中老是遇到一些坑，为了避免在同一个坑里跌倒两次，故在产生了记录之的想法。经过了几个月，已经积攒了十个坑，现在将其共享出来，以后将会持续更新。

我已经将本文markdown源文件放在了[Github](https://github.com/iWoz/share_works)上，通过Fork和P&R来提交你的「坑」和「解决方案」，帮助我完善之。你也可以通过关注这个项目对这些「坑」保持持续关注。

* **iOS设备出现本地存档丢失**
	* 描述：在苹果设备上，当系统提示存储空间已满时，发现本地的存档会丢失。
	
	* 原因：在默认情况下，本地存档放在了`/Library/Caches`下面，根据[苹果官方的描述](https://developer.apple.com/library/ios/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html#//apple_ref/doc/uid/TP40010672-CH2-SW1)，放在`/Library/Caches`目录下的任意文件将在系统弹出存储空间将满的警告时**被系统清空**。
	
	* 解决方案：将所有数据和热更新文件放在`/Library/Application Support`目录下，此目录下的所有文件在收到空间将满警告时不会被移除。此外，这还避免了放在`Documents`目录下可能会被苹果在审核中干掉的风险。

* **Apple Watch 版本提交失败**
	* 描述：首次向AppStore提交带Apple Watch的版本，提示失败，导致提交无法继续。
	
	* 原因：Apple Watch版本的AppIcon的PNG图带了Alpha通道，故被拒。
	
	* 解决方案：去除Apple Watch版本的所有AppIcon的PNG图的Alpha通道。
	
* **Apple Watch 版本审核被拒**
	* 描述：Apple Watch版本提交后，在苹果审核的阶段被拒了。
	
	* 原因：在iPhone上的Apple Watch的这个应用内，我们的游戏名称显示为codename：`xxx ios`。
	
	* 解决方案：发现在Watchkit Extension的Info.plist里，**Bundle name**为默认的`PRODUCT_ID`，这就是我们的codename，将**Bundle name**修正为游戏名称即可。需要注意的是**Bundle name**不等同于**Bundle display name**，前者用于系统的设置的一些显示名称，后者用于在Launcher的App的名称显示。
	
* **单机游戏内购（IAP）被破解**
	* 描述：根据后台的Counter报告，我们确信我们的单机游戏内购被破解了。
	
	* 原因：一个高度可能的原因是我们把订单编号存在本地的一个缓存文件里，每次去苹果服务器询问订单是否成功时，先去此缓存文件内查找是否有相同的订单编号，若找到则说明订单有重复发。但是一旦玩家删除了这个缓存文件，则可反复利用一个已经支付成功了的订单号来反复刷了。
	
	* 解决方案：最稳妥的解决方案是将订单编号存在服务端，然后对服务端的通信进行加密。我们采用了一种不走服务器的方法：即在首次充值成功时，给金币的缓存文件添加一个标记位（负号），然后查询订单缓存文件时，先去查询此标记位，若找到标记位，则说明之前充值过，订单缓存文件应有内容，如果订单缓存文件内容为空，或找不到有意义的订单编号，则说明玩家作弊，此次充值金币将不会加上（作弊惩罚）。由于缓存文件事先已被AES加密过，所以玩家很难去找到该标记位。

* **Apple Watch OS2 运行时找不到图片**
	* 描述：Apple Watch OS1的版本一行代码没改，但运行起来却提示图片无法找到。
	
	* 原因：WatchKit App 下的`Images.xcassets`里的图只设置了`1x`的图片，`2x`和`3x`没有设置。
	
	* 解决方案：设置好`2x`和`3x`的图片。
	
* **用Application Loader 上传二进制时报错**`ERROR ITMS-90168: "The binary you uploaded was invalid."`
	* 描述：同标题。
	
	* 原因：未知，可能跟Xcode升级到7有关。
	
	* 解决方案：打开命令行，输入以下代码：
```bash
cd ~/.itmstransporter
rm update_check*
mv softwaresupport softwaresupport.bak
cd UploadTokens
rm *.token
```

* **iOS 9以下的设备不支持ReplayKit导致无法启动游戏**
	* 描述：同标题。
	
	* 原因：ReplayKit是iOS9才引入的framework，所以无法在iOS9以下的设备上使用。
	
	* 解决方案：打开Xcode，在target的`Build Phases`下搜索`ReplayKit`，把`ReplayKit.framework`的Status由`Require`改成`Optional`。同理，在遇到低版本iOS系统不支持的情况，比如iOS8以下不支持`CloudKit`，一律将framework的Status由`Require`改成`Optional`即可。
	
* **Xcode 7以上默认不支持http请求**
	* 描述：`Application Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. Temporary exceptions can be configured via your app's Info.plist file.`。
	
	* 原因：Xcode 7以上为了安全考虑**默认只支持https请求**。
	
	* 解决方案：打开Xcode，编辑`Info.plist`或选中target的Info栏，新增字段`App Transport Security Settings`，将其内键`Allow Arbitrary Loads`设置值为`YES`。

* **Xcode 7 ERROR ITMS-90474: “Bundle Invalid...”**
	* 描述：提交二进制文件时报错：`ERROR ITMS-90474: "Bundle Invalid. iPad Multitasking support requires there orientations: 'UIInterfaceOrientationPortrait,UIIinterfaceOrientationPortraitUpsideDown,UIInterfaceOrientationLandscapeLeft,UIInterfaceOrientationLandscapeRight'. Found 'UIInterfaceOrientationPortrait' in bundle.`。
	
	* 原因：由于iOS 9引进了多任务处理，所以iOS应用必须设置其是否要求全屏显示。
	
	* 解决方案：打开Xcode，选中target，在`General`一栏，将`Requires full screen`勾选上。
	
* **Xcode 7 ERROR UnityAds.Bundle ITMS-90535 Unexpected CFBundleExecutable Key **
	* 描述：提交二进制文件时报错：`ERROR UnityAds.Bundle ITMS-90535 Unexpected CFBundleExecutable Key`。
	* 原因：老版本的UnityAds.Bundle里包含`CFBundleExecutable`字段，苹果在Xcode7之后会对其进行验证。
	* 解决方案：Xcode内搜索`UnityAds`，找到UnityAds.Bundle里的Info.plist，删除其`CFBundleExecutable`字段即可，该字段在Xcode中显示的名称为：`Executable file`。


