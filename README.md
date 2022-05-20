# 考试助手

让你考试如鱼得水。

电脑没有git的同学可以直接下载***`压缩包`***里的内容，里面是打包好的代码和考试客户端。

***如果你电脑上的客户端是2.0.1版本则不需要下载客户端。***

## 环境配置

1）项目是NodeJS脚本，需要在本地安装NodeJS环境，没有特殊版本要求，按照操作系统要求即可。

2）抓包采用的是Fiddler，需要安装 Fiddler Classic 版本。

3）考试客户端是 2.0.1。

## 主要文件/文件夹说明

- config          配置文件
- lib             需要用的方法
- examBaby.js     主体文件
- index.js        启动文件

### config 文件夹

- request.txt保存用户登录信息的请求header。当前用fiddler自动保存，编码是utf16le
- questionFile.json 保存考试信息题目列表
- template.html   最后生成页面采用的模板

## 运行前准备 - Fiddler配置

1、Fiddler首先开启HTTPS监控，Rules → Options → `HTTPS`选项卡。

2、在自定义规则脚本内做以下修改，根据实际情况替换`{{替换成NodeJS项目下config路径}}`;路径中的 `\` 需要用双反斜杠(`\\`)代替。

如：`E:\\NodeJS\\config\\request.txt`

打开自定义规则方式：

1、Rules → Customize Rules...；快捷键 CTRL + R；

2、默认排版下，右侧的FiddlerScript选项卡；在使用此方式情况下，可以直接选择`OnBeforeRequest`和`OnBeforeResponse`;

以下代码粘贴到 static function OnBeforeRequest(oSession: Session) 函数内
```
//拦截特定请求
if (oSession.fullUrl.Contains("/api/ecs_oe_student/examRecordPaperStruct/getExamRecordPaperStruct")) {
    //消除保存的请求可能存在乱码的情况
    var fso;
    var file;
    fso = new ActiveXObject("Scripting.FileSystemObject");
    //文件保存路径，可自定义
    file = fso.OpenTextFile("{{替换成NodeJS项目下config路径}}\\request.txt", 2, true, true);
    file.writeLine(''+oSession.oRequest.headers);
    file.writeLine("\n");
    file.close();

}
```

以下代码粘贴到 static function OnBeforeResponse(oSession: Session) 函数内
```
//拦截特定请求
if (oSession.fullUrl.Contains("/api/ecs_oe_student/examRecordPaperStruct/getExamRecordPaperStruct")) {
    oSession.utilDecodeResponse();
    //消除保存的请求可能存在乱码的情况
    var fso;
    var file;

    fso = new ActiveXObject("Scripting.FileSystemObject");
    //文件保存路径，可自定义
    file = fso.OpenTextFile("{{替换成NodeJS项目下config路径}}\\questionFile.json", 2, true, true);
    file.writeLine(''+oSession.GetResponseBodyAsString());
    file.writeLine("\n");
    file.close();
}
```

## 运行前准备 - 安装项目必须组件

在项目目录下，运行 `npm install`

## 运行前准备 - 配置本地站点（非必选）

在 `examBaby.js` 脚本运行最后会将生成的名为`exam.html`的文件并复制到本地站点中，方便其他设备通过浏览器查看。

因此需要将其中的地址修改为本地站点路径。如果本机没有web服务，可以选用任何熟悉的web服务，如：IIS，Apache。并确保可以通过IP可以在局域网内其他设备上打开。

亦或者将`question`目录共享，如，右键文件夹 选择 共享选项卡，之后在 ipad 上 `文件` 中的连接服务器进行查看。

## 运行

1、启动Fiddler开始流量监控；

2、在项目目录下，运行 `node index`即可。成功获取答案后计算机蜂鸣器会发声，跟windows出错声音一致。此时，在项目的`question`目录会生成数字加字母的HTML文件，可以通过切换桌面的方式进行访问；如果本地站点部署正确，亦或者可以直接在其他设备输入地址访问。

在点击开始考试按钮时候即可获取到试题，在练习阶段可以进入试题界面直接交卷。


## 提示

***请先在 `练习部分` 测试通过后再进行正式考试。***

也可以只进行`练习`，正式考试中从练习题中获取答案。

***在考试过程中，切换桌面会被截图上传，可能会影响考试成绩。要慎重。***

在正式考试中，获取的习题与考试题一致。***试卷中的主观题同样没有答案。***

## 免责声明

脚本作者不对此脚本的使用以及由此产生的后果承担任何责任。

## 可能出现的问题

1、1.x版本的客户端可能无法获取到试题。

这个需要需要自行调整Fiddler的拦截策略。作者本地客户端是2.x版本，但是很奇怪，官网的是1.x版本。作者依据的一直都是本地客户端策略。所以这个问题不打算解决。

2、生成的HTML文件可能打开缓慢。

HTML模板使用的是外链CDN（jsdelivr）样式，卡顿可能是外链的样式出了问题。

3、2.x版本客户端无法展开试卷

嗯，作者这里确实是无法展开试卷。
