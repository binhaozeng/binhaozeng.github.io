# 常用方法

> 正常使用需要用到的方法记录



## 类方法等

### 1.[String]字符串方法

#### 1.1 获取某字符的位置及其富文本设置

- 由于`swift`原生的字符串处理方法依然不方便，当前对字符串的部分内容进行处理依然采用`NSString`的方法
- `NSRange`的结构里包含`location`和`length`，可以较为方便获取某个字符的当前位置

```swift
let str = "abcd 1234"
let attrStr = NSMutableAttributedString(string: str)
//获取空格处的位置信息
let midIndex = (str as NSString).range(of: " ").location
let range1 = NSRange(location: 0, length: midIndex)
let range2 = NSRange(location: midIndex, length: str.count - midIndex)

//设置富文本
attrStr.setAttributes([NSAttributedString.Key.font : UIFont.systemFont(ofSize: 28)], range: range1)

//当设置为UILabel的富文本时，未进行设置的富文本按默认格式展示
label.attributedText = attrStr

```

#### 1.2 字符串富文本文字上下方向上的位移

- 当我们设置富文本文字大小不一致时，小号字体文字默认位于下方
- 如果使用的字体格式比较特别甚至会出现小号的字体位于下方偏下的位置，无法和大号字体对齐
- 文本框(`UILabel`)只有水平位置上的对齐，没有文字垂直方向上的调整，故要实现只能借助富文本的方法对其位置进行移动
- `NSAttributedString`的`baselineOffset`方法可以使富文本的文字位置以底端为基准进行移动，可以通过文字大小大致计算并实现底端对齐或者垂直居中对齐

```swift
attrStr.setAttributes([NSAttributedString.Key.baselineOffset: 5], range: range1)
```



### 2.时间及时间戳处理

- 正常的调用`Date()`方法获取的是手机里显示的时间，而非标准时间

#### 2.1 获取时间戳

```swift
//获取当前手机设置的时间，而非标准时间
let now = Date()
let timeInterval = now.timeIntervalSince1970
print("当前的时间戳：\(timeInterval)")

```

#### 2.2 将时间戳转换为具体时间

```swift
let date = Date(timeIntervalSince1970: timeInterval)
let formatter = DateFormatter()
formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
print("时间戳对应的时间：\(formatter.string(from: date))")

```

#### 2.3 将`Date`直接转换为具体时间

```swift
let now = Date()
let formatter = DateFormatter()
formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
print("时间戳对应的时间：\(formatter.string(from: now))")
```





## 界面方法

### 1.状态栏`UIStatusBar`相关设置

#### 1.1 设置状态栏字体颜色

- 状态栏的样式有两种: **`.lightContent`(白色),` .default`(黑色)**

- 要使用该方法设置状态栏，需要先设置`.plist`里面的参数：`View controller-based status bar appearance`值设为`YES`

设置更改时需要调用：

```swift
setNeedsStatusBarAppearanceUpdate()
```

调用该方法后，在以下方法里更改状态栏的样式：

```swift
override var preferredStatusBarStyle: UIStatusBarStyle {
    if <##> {
        return .lightContent
    } else {
        return .default
    }
}
```



### 2.`UINavigation`及`UITabbar`相关方法

#### 2.1 设置`push`进子界面的同时隐藏底部`UITabbar`

- 设置子界面的`hidesBottomBarWhenPushed`来隐藏子界面及其下级界面的`UITabbar`

- 可以在`NavigationController`的基类里覆盖`push`方法

```swift
		override func pushViewController(_ viewController: UIViewController, animated: Bool) {
        if (viewControllers.count > 0) {// 当非根控制器隐藏底部tabbar
            viewController.hidesBottomBarWhenPushed = true;
            
            viewController.navigationItem.leftBarButtonItem = UIBarButtonItem(image: UIImage(named:"back"), style: .plain, target: self, action: #selector(back))
            
        }
        super.pushViewController(viewController, animated: animated)
    }
```



#### 2.2 设置UINavigationBar上按钮的文字颜色

全局设置：

```swift
UINavigationBar.appearance().tintColor = UIColor.orange
```

单独某页面设置：

```swift
navigationItem.rightBarButtonItem?.setTitleTextAttributes([NSAttributedString.Key.foregroundColor:UIColor.primary], for:.normal)
navigationItem.rightBarButtonItem?.setTitleTextAttributes([NSAttributedString.Key.foregroundColor:UIColor.primary], for:.highlighted)
```



#### 2.3 设置UITabBarItem上的文字颜色

```swift
UITabBarItem.appearance().setTitleTextAttributes(
            [NSAttributedString.Key.foregroundColor:UIColor.primary], for:.selected)
```





### 3.滚动视图相关设置

#### 3.1 设置`UITableView`分隔线（下划线）

- 移除多余分隔线

```swift
tableView.tableFooterView = UIView(frame: CGRect.zero)
```

- 将分隔线左右拉伸

```
tableview.separatorInset = UIEdgeInsetsMake(0, 0, 0, 0)
```

- 设置分隔线颜色

```
tableview.separatorColor = UIColor.line_grey
```



#### 3.2 UITableView及UICollectionView的点击事件穿透设置方法

- 该方法可以将滚动视图内部的滚动视图的响应穿透，使所有的响应皆为该滚动视图的
- 被吃掉的响应无法再有点击触碰、滑动等效果
- 故使用该方法时需考虑被穿透的响应是否需要
- 设置`UITableView`的穿透，则`UITableView`中除`UITableViewCell`的响应全部失效；设置`UITableViewCell`的穿透，则cell内部的所有响应全部失效
- 如果子滚动视图存在需要滚动的情况，则应该只判断`UITableView`(其他滚动视图同理)

~~~swift
//点击事件穿透，不响应tableView的点击事件
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    let view = super.hitTest(point, with: event)
    if let bool1 = (view?.superview?.isKind(of: UITableViewCell.self)), let bool2 = view?.isKind(of: UITableView.self), bool1 || bool2 {
				return self
    }
    return view
}
~~~



### 4.UITextView

#### 4.1 UITextView设置超链接，点击可响应及其相关方法

```swift
-------------------------------------Class-------------------------------------

--ViewDidLoad:

//The description for the phone call.
let tipStr = "一串内容。"
let popStr = "内容后面可点击的文字"
let contentStr = tipStr + popStr
let range = NSRange(location: tipStr.count, length: popStr.count)
let contentAttrStr = NSMutableAttributedString(string: contentStr)
contentAttrStr.addAttributes([NSAttributedString.Key.link : "pop://"], range: range)

phoneCallContentTv.isEditable = false
phoneCallContentTv.delegate = self
phoneCallContentTv.attributedText = contentAttrStr
phoneCallContentTv.font = UIFont.systemFont(ofSize: 12)
phoneCallContentTv.textContainerInset = UIEdgeInsets.zero
phoneCallContentTv.tintColor = UIColor.primary
headBoard.addSubview(phoneCallContentTv)


-------------------------------------extension-------------------------------------

extension PhoneCallReminderController:UITextViewDelegate {
    func textView(_ textView: UITextView, shouldInteractWith URL: URL, in characterRange: NSRange) -> Bool {
        if URL.description == "pop://" {
            showDetails()
            return false
        }
        return true
    }
}

```



#### 4.2 禁止UITextView被选中(复制粘贴全选)

```swift
//禁止textView选中
override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
    phoneCallContentTv.resignFirstResponder()
    return super.canPerformAction(action, withSender: sender)
}

```



## 存储方法

### 1.Bundle文件的读取

Bundle文件直接存储在工程项目文件夹内

```swift
//获取直接存放如工程的名为zz.png的图片
let path = Bundle.main.path(forResource: "zz", ofType:"png")
let bImage = UIImage(contentsOfFile: path!)
```



文件存放在工程目录的.Bundle文件内

```swift
let bundlePath = Bundle.main.path(forResource: "MCircleBoard", ofType: "bundle") ?? ""
let bundle = Bundle(path: bundlePath)
let filePath = bundle?.path(forResource: imageName, ofType: "png") ?? ""
let image = UIImage(contentsOfFile: filePath) ?? UIImage()
```



**bundle内图片通用获取方式，无论本工程还是第三方库内的图片都可获取**

```swift
var bundle = Bundle(for: theClass.self)
if let resoucePath = bundle.path(forResource: "bundleName", ofType: "bundle"), let resouceBundle = Bundle(path: resoucePath) {
    bundle = resouceBundle
}
let image = UIImage(named: imageName, in: bundle, compatibleWith: nil) ?? UIImage()
```

- `theClass`：工程项目名称；`bundleName`：对应的bundle的名字



### 2.沙盒文件的写入与获取

- 沙盒即程序自己的目录，即文件的本地化存储
- `swift`暂未开放自己读写文件的操作，要实现沙盒文件的读取与写入必须将文件转换为`oc`的类
- 文件写入时可以直接加后缀也可以不加，不影响取出后的使用（取出时会转换成可用的样式）

#### 2.1 图片的存取

保存图片到沙盒

```swift
let image = UIImage(named: "fox")
//给保存在沙盒的文件命名
let imageName = "theFox"

if let imageData =
currentImage.jpegData(compressionQuality: percent) {
    let path = NSHomeDirectory().appending("/Documents/").appending(imageName)
    
    try? imageData.write(to: URL(fileURLWithPath: fullPath(imageName)), options: Data.WritingOptions.atomic)
    
    print("Image path:\(path)")
}

```

从沙盒中取出图片

```swift
//沙盒中文件的名字
let imageName = "theFox"
let path = NSHomeDirectory().appending("/Documents/").appending(imageName)

if let image = UIImage(contentsOfFile: fullPath(imageName)) {
  	print("获取图片成功")
    //return image
} else {
    print("获取图片失败")
    //return nil
}

```



#### 2.2 字典的存取

保存字典到沙盒

```swift
let dic = Dictionary()
let dicName = "theDic"
let path = NSHomeDirectory().appending("/Documents/").appending(dicName)

let nDic = NSDictionary(dictionary: dic)
nDic.write(to: URL(fileURLWithPath: path),atomically: true)
print("Dictionary path:\(path)")

```

从沙盒中取出字典

```swift
let dicName = "theDic"
let path = NSHomeDirectory().appending("/Documents/").appending(dicName)

if let dic = NSDictionary(contentsOfFile: fullPath(dicName)) {
  	print("获取字典成功")
    //return dic as? Dictionary
} else {
    print("获取字典失败")
    //return nil
}

```



#### 2.3 数组的存取

保存数组到沙盒

```swift
let dic = Dictionary()
let dicName = "theDic"
let path = NSHomeDirectory().appending("/Documents/").appending(dicName)

nArray.write(to: URL(fileURLWithPath: path), atomically: true)
print("Array path:\(path)")

```

从沙盒中取出数组

```swift
let arrayName = "theArray"
let path = NSHomeDirectory().appending("/Documents/").appending(arrayName)

if let array = NSArray(contentsOfFile: fullPath(arrayName)) {
  	print("获取数组成功")
    //return array as Array
} else {
    print("获取数组失败")
    //return nil
}

```



#### 2.4 移除文件

```swift
let pathHead:String = NSHomeDirectory().appending("/Documents/")

let arrayName = "theArray"
let path = NSHomeDirectory().appending("/Documents/").appending(arrayName)

//移除文件
try? FileManager.default.removeItem(atPath: path)

//移除所有文件
func deleteAllFile() {
    let enumerator = FileManager.default.enumerator(atPath: pathHead)
    guard enumerator != nil else {return}
    
    for fileName in enumerator!.allObjects as! [String] {
        deleteFile(fileName: fileName)
    }
}

```



### 3.App间分享文件

- iOS的每个App之间相互独立，如同一个个沙盒，所以无法直接相互分享文件
- App间分享文件必须利用iOS本身的文件共享功能



#### 3.1 接收分享文件

- 需要先再`info.plist`添加新属性`CFBundleDocumentTypes`(实际显示为`Document type`)
- `Document type`是一个数组类型的属性，可以在里面同时注册多个类型，[详细属性列表](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html)如下。
- 设置完成后需要在`AppDelegate`文件添加`func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool`方法，并导入对应代码处理接收到的文件



##### 3.1.1 `CFBundleDocumentTypes`的常用属性

可以直接在plist里添加属性，也可以

- CFBundleTypeName

字符串类型，指定某种类型的别名，也就是用来指代我们规定的类型的别称，一般为了保持唯一性，我们使用UTI来标识。

- CFBundleTypeIconFiles

数组类型，包含指定的png图标的文件名，指定代表某种类型的图标，而图标有具体的尺寸标识：`iPad`、`iPhone and iPod touch`

- LSItemContentTypes

数组类型，包含UTI字符串，指定我们的应用程序所有可以识别的文件类型集合 

- LSHandlerRank 

字符串类型，包含`Owner`、`Default`、`Alternate`、`None`四个可选值，指定对于某种类型的优先权级别，而Launcher Service会根据这个优先级别来排列显示的App的顺序。优先级别从高到低依次是`Owner`、`Alternate`、`Default`。`None`表示不接受这种类型。


可以直接存放进plist，也可以以`Source code`的方式编辑保存：

```
<key>CFBundleDocumentTypes</key>
<array>
    <dict>
        <key>CFBundleTypeName</key>
        <string>PDF</string>
        <key>LSHandlerRank</key>
        <string>Owner</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>com.adobe.pdf</string>
        </array>
    </dict>
    <dict>
        <key>CFBundleTypeName</key>
        <string>Image</string>
        <key>LSHandlerRank</key>
        <string>Alternate</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>public.image</string>
        </array>
    </dict>
</array>

```



##### 3.1.2 接收并处理共享文件

- 需要在`AppDelegate`的`func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool`方法里添加代码，对接收到的文件进行处理保存
- iOS9之前的回调是：`func application(_ application: UIApplication, open url: URL, sourceApplication: String?, annotation: Any) -> Bool`，如有需要则需要根据版本分别做处理

当前保存在`Documents`文件夹

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    
    var path = url.absoluteString
    if path.contains("file:///private") {
        path = path.replacingOccurrences(of: "file:///private", with: "")
    }
    let tmpArray = path.components(separatedBy: "/")
    guard tmpArray.count > 0 else {
        return true
    }
    let fileName = tmpArray.last
    let filePath = NSHomeDirectory().appending("/Documents/").appending(fileName!)
    
    try? FileManager.default.copyItem(atPath: path, toPath: filePath)
    if FileManager.default.fileExists(atPath: filePath) {
        print("The file has been shared successfully.")
    } else {
        print("The file has not been added to the directory.")
    }
    
    return true
}

```





## 功能型方法

### 1.拨打电话

```
let number = String(format: "telprompt://%@", "电话号码")
UIApplication.shared.openURL(URL(string: number)!)
```



### 2.通过类名反射创建类

```swift
guard let nameSpace = Bundle.main.infoDictionary?["CFBundleName"] as? String else { return }
let clsName = String(format: "%@.%@", nameSpace, "FirstViewController")
let cls = (NSClassFromString(clsName) as? UIViewController.Type)!
let vc = cls.init()

present(vc, animated: true, completion: nil)
```



### 3.获取App系统信息

```swift
//获取App版本号，如“1.0.2“
let version = Bundle.main.infoDictionary!["CFBundleShortVersionString"] as? String ?? ""

//获取App名称
let nameSpace = Bundle.main.infoDictionary?["CFBundleName"] as? String

//获取App的限制最低版本号
let version = UIDevice.current.systemVersion

```





## 常用记录

### 1.时钟字体

`DBLCDTempBlack`

用名字的方式创建UILabel





# 实用方法

> 部分涉及编程思路及系统性序列性方法的记录



## 解析方法

### 1.[JSON解析]Model创建及使用(Codable)

#### 1.1 `Model`的创建

```
struct DMModel: Codable {
    var code: TStrInt
    var data: DMData
    
    struct DMData: Codable {
        var stateCode: TStrInt
        var message: String
        var returnData: DMReturnData?
    }
    
    struct DMReturnData: Codable {
        var rankinglist: [DMRankingList]?
    }
    
    struct DMRankingList: Codable {
        var title: String
        var subTitle: String
        var cover: String
        var argName: String
        var argValue: TStrInt
        var rankingType: String
    }
}
```

#### 1.2 对`JSON`进行解析

```
let decoder = JSONDecoder()
let model = try? decoder.decode(DMModel.self, from: json)
```



## 优化

### 1.内存优化

#### 1.1 闭包(`Block`/`Closure`)避免循环引用

- 如果闭包内部含有控制器内的全局参数的时候，为避免循环引用，在闭包内部使用`[unowned self]`来修饰

- 如果内部有使用协议`delegate`，则可使用`[unowned self， weak delegete = self.delegate]`进行修饰

```swift
lazy var closure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate] (index: Int, title: String) -> String in
    // closure body goes here
    print("\(self.name)")
    delegate.callBack(title)
}

```



#### 1.2 检测对象是否发生内存泄漏

如果我们要知道对象x是否发生内存泄漏，我们可以为它创建一个弱引用，并称为leakReferece。如果x从内存中释放，ARC会设leakReference为空。所以如果x泄漏，leakReference就一定不是空。

```swift
var x : SomeObject? = SomeObject()
weak var leakReference = x
x = nil

if leakReference == nil {
  //Not leaking.未发生泄漏
  
} else {
  //Leaking.已发生内存泄漏
  
}

```

当处于单元测试时，也可以直接使用`XCTAssertNil(leakReference, "Leak!!")`，当泄漏的时候直接报错。





## iOS库相关



### 1.上传个人库到cocoapods

#### 1.1 在GitHub创建项目

- 填写项目名

- 填写项目描述
- 设置库公开(Public)
- 添加证书及README文件，证书也可以在项目创建完成后点击`Create new file`并输入`LICENSE`，点击右上角出现的`Choose a license template`选择需要的证书并添加进去

设置完成后直接克隆到本地，并将工程代码复制进该文件夹

#### 1.2 创建podspec文件

##### 1.2.1 通过终端进克隆到本地的文件夹

```
cd xxx
```

##### 1.2.2 创建podspec文件

```
pod spec create xxx
```

- xxx为自己创建的库的名字，如`Alamofire`

##### 1.2.3 配置podspec文件

```ruby
@version = "0.0.1"
Pod::Spec.new do |s|
s.name = "MCircleBoard"
s.version = @version
s.summary = "MCircleBoard"
s.description = "MCircleBoard is a dash board."
s.homepage = "https://github.com/whitesage/MCircleBoard"
s.license = { :type => "MIT", :file => "LICENSE" }
s.author = { "WhiteSage" => "1216831710@qq.com" }
s.platform = :ios
s.ios.deployment_target = "8.0"
s.source = { :git => "https://github.com/whitesage/MCircleBoard.git", :tag => "v#{s.version}" }
s.source_files = "Code/*.swift"
s.resources = ["Code/*.xcassets"]
s.swift_version = "4.0"
s.requires_arc = true
s.framework = "UIKit"

end
```

- 如果有依赖库则增加如：`s.dependency "AFNetworking", '~> 3.0'`
- 比较重要的字段有：`name`、`version`、`summary`、`homepage`、`license`、`author`、`deployment_target`、`source`、`source_files`、`requires_arc`
- `source_files`：最好把要上传的文件放在专门的文件夹下
- 往库里添加资源：添加bundle然后设置属性`resource`\\`resources`\\`resource_bundles`



spec里添加资源：

```
spec.resource = 'Code/MCircleBoardIcon.bundle'
spec.resources = ['Images/*.png', 'Sounds/*']
spec.resource_bundles = {
    'YourBundleName' => ['MapView/Map/Resources/*.png'],
    'OtherResources' => ['MapView/Map/OtherResources/*.png']
  }
```

bundle内图片的调用方式：

```swift
var bundle = Bundle(for: theClass.self)
if let resoucePath = bundle.path(forResource: "bundleName", ofType: "bundle"), let resouceBundle = Bundle(path: resoucePath) {
    bundle = resouceBundle
}
let image = UIImage(named: imageName, in: bundle, compatibleWith: nil) ?? UIImage()
```

- 要设置成为第三方库，获取图片只能通过该方式，而不能像主工程项目里的获取方式
- `theClass`：工程项目名称；`bundleName`：对应的bundle的名字



podspec字段说明：

| 字段              | 作用                                                       |
| ----------------- | ---------------------------------------------------------- |
| name              | 库的名称                                                   |
| version           | 版本号，该号一定要和GitHub上的对应，参见下面的步骤         |
| summary           | 该库的一个简介                                             |
| homepage          | 这个库的主页，对应Github上的地址                           |
| license           | 采用何种license，和创建GitHub时候选择的那个要对应          |
| author            | 作者                                                       |
| deployment_target | 可以使用该库的iOS的最低版本                                |
| source            | 数据源，对应的是你`clone`的时候使用的那个`HTTPS`的链接地址 |
| source_files      | 哪些文件是要上传到pods的                                   |
| requires_arc      | 是否在ARC环境下使用                                        |
| dependency        | 该库还引用了哪些第三方库                                   |



#### 1.3 GitHub上创建发布版本

- podspec文件验证无误后需要先将项目更新到GitHub上
- 更新完毕后再release创建新版本，`Tag version`以v开头，后接版本号，标题`release title`需要填写
- 如果操作过程中有对代码进行了更改，则需要删除release，上传代码并重新创建该release



#### 1.4 注册Trunk账号及上传项目

##### 1.4.1 注册cocoapods账号

> 如果已经注册成功账号，则这步跳过

在终端输入指令：

```ruby
pod trunk register 邮箱地址 '用户名' —description='描述信息'
或
pod trunk register 邮箱地址 '用户名'  --verbose
```

- 用户名后面的信息可不写
- 注册成功后会提示去邮箱验证信息
- 注册的信息最好和GitHub保持一致，方便记忆



验证成功后在终端输入以下命令确认是否注册成功

```ruby
pod trunk me
```

- 出现用户信息则注册成功



##### 1.4.2 验证podspec文件

>  如果确认podspec文件没问题则可以不进行验证
>
>  如果配置得正确，则输入验证命令后会提示：`passed validation.`

podspec文件配置完成之后可以在终端输入指令进行验证

```ruby
pod spec lint XXX.podspec
```

- 如果验证的结果有error则必须按要求进行处理；如果有warning则可以选择处理也可以忽视
- 后面的`XXX.podspec`可省略
- `pod lib lint`只从本地验证你的pod能否通过验证；`pod spec lint`从本地和远程验证你的pod能否通过验证。`pod lib lint`可省略，直接用`pod spec lint`进行验证



##### 1.4.3 上传cocoapods

输入以下指令将代码上传到cocoapods：

```ruby
pod trunk push
```

如果之前验证的结果里有`warning`，则可以用`--allow-warnings`忽略警告：

```ruby
pod trunk push --allow-warnings
```



完成后出现`Tell your friends`字样则证明上传完毕，可以通过`pod search XXX`指令来验证是否上传成功。

如果没搜索出来则可以通过`pod repo update`更新本地的pod库，然后再进行搜索。

如果事先已经下载了旧版的库，可以直接cd到该工程，并`pod update 第三方库名`，哪怕暂时无法搜索出来，`pod update`依然可以有效将其更新到最新版本。

[自己上传的第三方库MCircleBoard](https://github.com/whitesage/MCircleBoard)



[参考链接1](https://rakuyomo.github.io/2017/08/21/28-iOS%E5%BC%80%E5%8F%91%E4%B9%8B%E5%B0%86%E8%87%AA%E5%B7%B1%E7%9A%84%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0CocoaPods/)

[参考链接2](https://www.bbsmax.com/A/pRdBjmA1dn/)





## Git命令

### 1.设置账号及邮箱

#### 1.1 查看Git账号信息

查看用户名

```
git config user.name
```

查看用户邮箱

```
git config user.email
```



#### 1.2 修改用户名及邮箱

修改用户及邮箱

```
git config --global user.name "Your_username"
git config --global user.email "Your_email"
```



