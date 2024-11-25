
![](https://img2024.cnblogs.com/blog/468667/202411/468667-20241124174433177-1037167232.gif)
【引言】


在本篇文章中，我们将探讨如何在鸿蒙NEXT平台上实现二维码的生成与识别功能。通过使用ArkUI组件库和相关的媒体库，我们将创建一个简单的应用程序，用户可以生成二维码并扫描识别。


【环境准备】


• 操作系统：Windows 10


• 开发工具：DevEco Studio NEXT Beta1 Build Version: 5\.0\.3\.806


• 目标设备：华为Mate60 Pro


• 开发语言：ArkTS


• 框架：ArkUI


• API版本：API 12


• 权限：ohos.permission.WRITE\_IMAGEVIDEO（为实现将图片保存至相册功能）


【项目介绍】


## 1\. 项目结构


我们首先定义一个名为`QrCodeGeneratorAndScanner`的组件，使用`@Component`装饰器进行标记。该组件包含多个状态变量和方法，用于处理二维码的生成、识别和剪贴板操作。


## 2\. 组件状态


组件的状态包括：


* `buttonOptions`: 定义分段按钮的选项，用于切换生成和识别二维码的功能。
* `inputText`: 用户输入的文本，用于生成二维码。
* `scanResult`: 扫描结果文本。
* `scanResultObject`: 存储扫描结果的对象。


## 3\. 用户界面构建


在`build`方法中，我们使用`Column`和`Row`布局来构建用户界面。主要包含以下部分：


* **分段按钮**：用户可以选择生成二维码或识别二维码。
* **输入区域**：用户可以输入文本并生成二维码。
* **二维码显示**：根据输入文本生成二维码。
* **扫描区域**：用户可以通过相机扫描二维码或从图库选择图片进行识别。


## 4\. 二维码生成


二维码生成使用`QRCode`组件，输入文本通过`this.inputText`传递。用户输入后，二维码会实时更新。


## 5\. 二维码识别


二维码识别功能通过`scanBarcode`模块实现。用户可以点击“扫一扫”按钮，启动相机进行扫描，或选择图库中的图片进行识别。识别结果将显示在界面上，并提供复制功能。


## 6\. 剪贴板操作


用户可以将扫描结果复制到剪贴板，使用`pasteboard`模块实现。点击“复制”按钮后，扫描结果将被复制，用户会收到提示。


【完整代码】


填写权限使用声明字符串：src/main/resources/base/element/string.json



[?](https://github.com)

| 1234567891011121314151617181920 | `{``"string"``: [``{``"name"``:` `"module_desc"``,``"value"``:` `"module description"``},``{``"name"``:` `"EntryAbility_desc"``,``"value"``:` `"description"``},``{``"name"``:` `"EntryAbility_label"``,``"value"``:` `"label"``},``{``"name"``:` `"WRITE_IMAGEVIDEO_info"``,``"value"``:` `"保存功能需要该权限"``}``]``}` |
| --- | --- |



配置权限：src/main/module.json5



[?](https://github.com):[楚门加速器p](https://tianchuang88.com)

| 123456789101112 | `{``"module"``: {``"requestPermissions"``: [``{``"name"``:` `'ohos.permission.WRITE_IMAGEVIDEO'``,``"reason"``:` `"$string:WRITE_IMAGEVIDEO_info"``,``"usedScene"``: {` `}``}``],``//... ...` |
| --- | --- |



示例代码：src/main/ets/pages/Index.ets



[?](https://github.com)

| 123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127128129130131132133134135136137138139140141142143144145146147148149150151152153154155156157158159160161162163164165166167168169170171172173174175176177178179180181182183184185186187188189190191192193194195196197198199200201202203204205206207208209210211212213214215216217218219220221222223224225226227228229230231232233234235236237238239240241242243244245246247248249250251252253254255256257258259260261262263264265266267268269270271272273274275276277278279280281282283284285286287288289290291292293294295296297298299300301302303304305306307308309310311312313314315316317318319320321322323324325326327328329330331332333334335336337338339340341342343344345346347348349350351352353354355356357358359360361362363364365366367 | `import` `{``componentSnapshot,` `// 组件快照``promptAction,` `// 提示操作``SegmentButton,` `// 分段按钮``SegmentButtonItemTuple,` `// 分段按钮项元组``SegmentButtonOptions` `// 分段按钮选项``} from` `'@kit.ArkUI'``;` `// 引入 ArkUI 组件库` `import` `{ photoAccessHelper } from` `'@kit.MediaLibraryKit'``;` `// 引入 MediaLibraryKit 中的照片访问助手``import` `{ common } from` `'@kit.AbilityKit'``;` `// 引入 AbilityKit 中的通用功能``import` `{ fileIo as fs } from` `'@kit.CoreFileKit'``;` `// 引入 CoreFileKit 中的文件 I/O 模块``import` `{ image } from` `'@kit.ImageKit'``;` `// 引入 ImageKit 中的图像处理模块``import` `{ BusinessError, pasteboard } from` `'@kit.BasicServicesKit'``;` `// 引入 BasicServicesKit 中的业务错误和剪贴板操作``import` `{ hilog } from` `'@kit.PerformanceAnalysisKit'``;` `// 引入 PerformanceAnalysisKit 中的性能分析模块``import` `{ detectBarcode, scanBarcode } from` `'@kit.ScanKit'``;` `// 引入 ScanKit 中的条形码识别模块` `@Entry``// 入口标记``@Component``// 组件标记``struct QrCodeGeneratorAndScanner {` `// 定义二维码生成与识别组件``@State` `private` `buttonOptions: SegmentButtonOptions = SegmentButtonOptions.capsule({``// 定义分段按钮选项``buttons: [{ text:` `'生成二维码'` `}, { text:` `'识别二维码'` `}] as SegmentButtonItemTuple,` `// 按钮文本``multiply:` `false``,` `// 不允许多选``fontColor: Color.White,` `// 字体颜色为白色``selectedFontColor: Color.White,` `// 选中字体颜色为白色``selectedBackgroundColor: Color.Orange,` `// 选中背景颜色为橙色``backgroundColor:` `"#d5d5d5"``,` `// 背景颜色``backgroundBlurStyle: BlurStyle.BACKGROUND_THICK` `// 背景模糊样式``})``@State` `private` `sampleText: string =` `'hello world'``;` `// 示例文本``@State` `private` `inputText: string =` `""``;` `// 输入文本``@State` `private` `scanResult: string =` `""``;` `// 扫描结果``@State @Watch(``'selectIndexChanged'``) selectIndex: number = 0` `// 选择索引``@State @Watch(``'selectedIndexesChanged'``) selectedIndexes: number[] = [0];` `// 选中索引数组``private` `qrCodeId: string =` `"qrCodeId"` `// 二维码 ID``@State` `private` `scanResultObject: scanBarcode.ScanResult = ({} as scanBarcode.ScanResult)` `// 扫描结果对象``@State` `private` `textColor: string =` `"#2e2e2e"``;` `// 文本颜色``@State` `private` `shadowColor: string =` `"#d5d5d5"``;` `// 阴影颜色``@State` `private` `basePadding: number = 30;` `// 基础内边距` `selectedIndexesChanged() {` `// 选中索引改变事件``console.info(```this``.selectedIndexes[0]:${``this``.selectedIndexes[0]}`)``this``.selectIndex =` `this``.selectedIndexes[0]``}` `selectIndexChanged() {` `// 选择索引改变事件``console.info(`selectIndex:${``this``.selectIndex}`)``this``.selectedIndexes[0] =` `this``.selectIndex``}` `private` `copyToClipboard(text: string): void {` `// 复制文本到剪贴板``const pasteboardData = pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, text);` `// 创建剪贴板数据``const systemPasteboard = pasteboard.getSystemPasteboard();` `// 获取系统剪贴板``systemPasteboard.setData(pasteboardData);` `// 设置数据``promptAction.showToast({ message:` `'已复制'` `});` `// 弹出提示消息``}` `build() {` `// 构建界面``Column() {` `// 列布局``SegmentButton({` `// 分段按钮``options:` `this``.buttonOptions,` `// 选项``selectedIndexes:` `this``.selectedIndexes` `// 选中索引``}).width(``'400lpx'``).margin({ top: 20 })` `// 设置宽度和外边距``Tabs({ index:` `this``.selectIndex }) {` `// 选项卡``TabContent() {` `// 选项卡内容``Scroll() {` `// 滚动视图``Column() {` `// 列布局``Column() {` `// 列布局``Row() {` `// 行布局``Text(``'示例'``)` `// 文本``.fontColor(``"#5871ce"``)` `// 字体颜色``.fontSize(16)` `// 字体大小``.padding(`${``this``.basePadding / 2}lpx`)` `// 内边距``.backgroundColor(``"#f2f1fd"``)` `// 背景颜色``.borderRadius(5)` `// 边框圆角``.clickEffect({ level: ClickEffectLevel.LIGHT, scale: 0.8 })` `// 点击效果``.onClick(() => {` `// 点击事件``this``.inputText =` `this``.sampleText;` `// 设置输入文本为示例文本``});``Blank();` `// 空白占位``Text(``'清空'``)` `// 清空按钮``.fontColor(``"#e48742"``)` `// 字体颜色``.fontSize(16)` `// 字体大小``.padding(`${``this``.basePadding / 2}lpx`)` `// 内边距``.clickEffect({ level: ClickEffectLevel.LIGHT, scale: 0.8 })` `// 点击效果``.backgroundColor(``"#ffefe6"``)` `// 背景颜色``.borderRadius(5)` `// 边框圆角``.onClick(() => {` `// 点击事件``this``.inputText =` `""``;` `// 清空输入文本``});``}``.height(45)` `// 设置高度``.justifyContent(FlexAlign.SpaceBetween)` `// 主轴对齐方式``.width(``'100%'``);` `// 设置宽度` `Divider();` `// 分隔线``TextArea({ text: $$``this``.inputText, placeholder: `请输入内容` })` `// 文本输入框``.width(`${650 -` `this``.basePadding * 2}lpx`)` `// 设置宽度``.height(100)` `// 设置高度``.fontSize(16)` `// 字体大小``.caretColor(``this``.textColor)` `// 光标颜色``.fontColor(``this``.textColor)` `// 字体颜色``.margin({ top: `${``this``.basePadding}lpx` })` `// 外边距``.padding(0)` `// 内边距``.backgroundColor(Color.Transparent)` `// 背景颜色``.borderRadius(0)` `// 边框圆角``.textAlign(TextAlign.JUSTIFY);` `// 文本对齐方式``}``.alignItems(HorizontalAlign.Start)` `// 交叉轴对齐方式``.width(``'650lpx'``)` `// 设置宽度``.padding(`${``this``.basePadding}lpx`)` `// 内边距``.borderRadius(10)` `// 边框圆角``.backgroundColor(Color.White)` `// 背景颜色``.shadow({` `// 阴影``radius: 10,` `// 阴影半径``color:` `this``.shadowColor,` `// 阴影颜色``offsetX: 0,` `// X 轴偏移``offsetY: 0` `// Y 轴偏移``});` `Row() {` `// 行布局``QRCode(``this``.inputText)` `// 二维码组件``.width(``'300lpx'``)` `// 设置宽度``.aspectRatio(1)` `// 设置宽高比``.id(``this``.qrCodeId)` `// 设置 ID``SaveButton()` `// 保存按钮``.onClick(async (_event: ClickEvent, result: SaveButtonOnClickResult) => {` `// 点击事件``if` `(result === SaveButtonOnClickResult.SUCCESS) {` `// 如果保存成功``const context: common.UIAbilityContext = getContext(``this``) as common.UIAbilityContext;` `// 获取上下文``let` `helper = photoAccessHelper.getPhotoAccessHelper(context);` `// 获取照片访问助手``try` `{` `// 尝试``let` `uri = await helper.createAsset(photoAccessHelper.PhotoType.IMAGE,` `'jpg'``);` `// 创建图片资源``let` `file = await fs.open(uri, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);` `// 打开文件``componentSnapshot.get(``this``.qrCodeId).then((pixelMap) => {` `// 获取二维码快照``let` `packOpts: image.PackingOption = { format:` `'image/png'``, quality: 100 }` `// 打包选项``const imagePacker: image.ImagePacker = image.createImagePacker();` `// 创建图像打包器``return` `imagePacker.packToFile(pixelMap, file.fd, packOpts).finally(() => {` `// 打包并保存文件``imagePacker.release();` `// 释放打包器``fs.close(file.fd);` `// 关闭文件``promptAction.showToast({` `// 弹出提示消息``message:` `'图片已保存至相册'``,` `// 提示内容``duration: 2000` `// 持续时间``});``});``})``}` `catch` `(error) {` `// 捕获错误``const err: BusinessError = error as BusinessError;` `// 转换为业务错误``console.error(`Failed to save photo. Code is ${err.code}, message is ${err.message}`);` `// 打印错误信息``}` `}` `else` `{` `// 如果保存失败``promptAction.showToast({` `// 弹出提示消息``message:` `'设置权限失败！'``,` `// 提示内容``duration: 2000` `// 持续时间``});``}``})``}``.visibility(``this``.inputText ? Visibility.Visible : Visibility.Hidden)` `// 根据输入文本设置可见性``.justifyContent(FlexAlign.SpaceBetween)` `// 主轴对齐方式``.width(``'650lpx'``)` `// 设置宽度``.padding(`${``this``.basePadding}lpx`)` `// 内边距``.margin({ top: `${``this``.basePadding}lpx` })` `// 外边距``.borderRadius(10)` `// 边框圆角``.backgroundColor(Color.White)` `// 背景颜色``.shadow({` `// 阴影``radius: 10,` `// 阴影半径``color:` `this``.shadowColor,` `// 阴影颜色``offsetX: 0,` `// X 轴偏移``offsetY: 0` `// Y 轴偏移``});``}.padding({ top: 20, bottom: 20 })` `// 设置内边距``.width(``'100%'``)` `// 设置宽度``}.scrollBar(BarState.Off)` `// 禁用滚动条``.align(Alignment.Top)` `// 顶部对齐``.height(``'100%'``)` `// 设置高度``}``TabContent() {` `// 第二个选项卡内容``Scroll() {` `// 滚动视图``Column() {` `// 列布局``Row() {` `// 行布局``Text(``'扫一扫'``)` `// 扫一扫文本``.fontSize(20)` `// 字体大小``.textAlign(TextAlign.Center)` `// 文本居中对齐``.fontColor(``"#5871ce"``)` `// 字体颜色``.backgroundColor(``"#f2f1fd"``)` `// 背景颜色``.clickEffect({ scale: 0.8, level: ClickEffectLevel.LIGHT })` `// 点击效果``.borderRadius(10)` `// 边框圆角``.height(``'250lpx'``)` `// 设置高度``.layoutWeight(1)` `// 布局权重``.onClick(() => {` `// 点击事件``if` `(canIUse(``'SystemCapability.Multimedia.Scan.ScanBarcode'``)) {` `// 检查是否支持扫描``try` `{` `// 尝试``scanBarcode.startScanForResult(getContext(``this``), {` `// 开始扫描``enableMultiMode:` `true``,` `// 启用多模式``enableAlbum:` `true` `// 启用相册选择``},``(error: BusinessError, result: scanBarcode.ScanResult) => {` `// 扫描结果回调``if` `(error) {` `// 如果发生错误``hilog.error(0x0001,` `'[Scan CPSample]'``,` `// 记录错误日志```Failed to get ScanResult by callback` `with` `options. Code: ${error.code}, message: ${error.message}`);``return``;` `// 退出``}``hilog.info(0x0001,` `'[Scan CPSample]'``,` `// 记录成功日志```Succeeded` `in` `getting ScanResult by callback` `with` `options, result is ${JSON.stringify(result)}`);``this``.scanResultObject = result;` `// 设置扫描结果对象``this``.scanResult = result.originalValue ? result.originalValue :` `'无法识别'``;` `// 设置扫描结果文本``});``}` `catch` `(error) {` `// 捕获错误``hilog.error(0x0001,` `'[Scan CPSample]'``,` `// 记录错误日志```Failed to start the scanning service. Code:${error.code}, message: ${error.message}`);``}``}` `else` `{` `// 如果不支持扫描``promptAction.showToast({ message:` `'当前设备不支持二维码扫描'` `});` `// 弹出提示消息``}``});` `Line().width(`${``this``.basePadding}lpx`).aspectRatio(1);` `// 分隔线` `Text(``'图库选'``)` `// 图库选择文本``.fontSize(20)` `// 字体大小``.textAlign(TextAlign.Center)` `// 文本居中对齐``.fontColor(``"#e48742"``)` `// 字体颜色``.backgroundColor(``"#ffefe6"``)` `// 背景颜色``.borderRadius(10)` `// 边框圆角``.clickEffect({ scale: 0.8, level: ClickEffectLevel.LIGHT })` `// 点击效果``.height(``'250lpx'``)` `// 设置高度``.layoutWeight(1)` `// 布局权重``.onClick(() => {` `// 点击事件``if` `(canIUse(``'SystemCapability.Multimedia.Scan.ScanBarcode'``)) {` `// 检查是否支持扫描``let` `photoOption =` `new` `photoAccessHelper.PhotoSelectOptions();` `// 创建照片选择选项``photoOption.MIMEType = photoAccessHelper.PhotoViewMIMETypes.IMAGE_TYPE;` `// 设置 MIME 类型``photoOption.maxSelectNumber = 1;` `// 设置最大选择数量``let` `photoPicker =` `new` `photoAccessHelper.PhotoViewPicker();` `// 创建照片选择器``photoPicker.select(photoOption).then((result) => {` `// 选择照片``let` `inputImage: detectBarcode.InputImage = { uri: result.photoUris[0] };` `// 获取选中的图片 URI``try` `{` `// 尝试``detectBarcode.decode(inputImage,` `// 解码条形码``(error: BusinessError, result: Array) => {` `// 解码结果回调``if` `(error && error.code) {` `// 如果发生错误``hilog.error(0x0001,` `'[Scan Sample]'``,` `// 记录错误日志```Failed to get ScanResult by callback. Code: ${error.code}, message: ${error.message}`);``return``;` `// 退出``}``hilog.info(0x0001,` `'[Scan Sample]'``,` `// 记录成功日志```Succeeded` `in` `getting ScanResult by callback, result is ${JSON.stringify(result,` `null``,` `'\u00A0\u00A0'``)}`);``if` `(result.length > 0) {` `// 如果有结果``this``.scanResultObject = result[0];` `// 设置扫描结果对象``this``.scanResult = result[0].originalValue ? result[0].originalValue :` `'无法识别'``;` `// 设置扫描结果文本``}` `else` `{` `// 如果没有结果``this``.scanResult =` `'不存在二维码'``;` `// 设置结果文本``}``});``}` `catch` `(error) {` `// 捕获错误``hilog.error(0x0001,` `'[Scan Sample]'``,` `// 记录错误日志```Failed to detect Barcode. Code: ${error.code}, message: ${error.message}`);``}``});``}` `else` `{` `// 如果不支持扫描``promptAction.showToast({ message:` `'当前设备不支持二维码扫描'` `});` `// 弹出提示消息``}``});``}``.justifyContent(FlexAlign.SpaceEvenly)` `// 主轴对齐方式``.width(``'650lpx'``)` `// 设置宽度``.padding(`${``this``.basePadding}lpx`)` `// 内边距``.borderRadius(10)` `// 边框圆角``.backgroundColor(Color.White)` `// 背景颜色``.shadow({` `// 阴影``radius: 10,` `// 阴影半径``color:` `this``.shadowColor,` `// 阴影颜色``offsetX: 0,` `// X 轴偏移``offsetY: 0` `// Y 轴偏移``});` `Column() {` `// 列布局``Row() {` `// 行布局``Text(`解析结果：\n${``this``.scanResult}`)` `// 显示解析结果文本``.fontColor(``this``.textColor)` `// 设置字体颜色``.fontSize(18)` `// 设置字体大小``.layoutWeight(1)` `// 设置布局权重``Text(``'复制'``)` `// 复制按钮文本``.fontColor(Color.White)` `// 设置字体颜色为白色``.fontSize(16)` `// 设置字体大小``.padding(`${``this``.basePadding / 2}lpx`)` `// 设置内边距``.backgroundColor(``"#0052d9"``)` `// 设置背景颜色``.borderRadius(5)` `// 设置边框圆角``.clickEffect({ level: ClickEffectLevel.LIGHT, scale: 0.8 })` `// 设置点击效果``.onClick(() => {` `// 点击事件``this``.copyToClipboard(``this``.scanResult);` `// 复制扫描结果到剪贴板``});``}.constraintSize({ minHeight: 45 })` `// 设置最小高度约束``.justifyContent(FlexAlign.SpaceBetween)` `// 设置主轴对齐方式``.width(``'100%'``);` `// 设置宽度为100%` `}``.visibility(``this``.scanResult ? Visibility.Visible : Visibility.Hidden)` `// 根据扫描结果设置可见性``.alignItems(HorizontalAlign.Start)` `// 设置交叉轴对齐方式``.width(``'650lpx'``)` `// 设置宽度``.padding(`${``this``.basePadding}lpx`)` `// 设置内边距``.margin({ top: `${``this``.basePadding}lpx` })` `// 设置外边距``.borderRadius(10)` `// 设置边框圆角``.backgroundColor(Color.White)` `// 设置背景颜色为白色``.shadow({` `// 设置阴影``radius: 10,` `// 设置阴影半径``color:` `this``.shadowColor,` `// 设置阴影颜色``offsetX: 0,` `// 设置X轴偏移``offsetY: 0` `// 设置Y轴偏移``});` `Column() {` `// 列布局``Row() {` `// 行布局``Text(`完整结果：`).fontColor(``this``.textColor).fontSize(18).layoutWeight(1)` `// 显示完整结果文本``}.constraintSize({ minHeight: 45 })` `// 设置最小高度约束``.justifyContent(FlexAlign.SpaceBetween)` `// 设置主轴对齐方式``.width(``'100%'``);` `// 设置宽度为100%` `Divider().margin({ top: 2, bottom: 15 });` `// 添加分隔线并设置外边距` `Row() {` `// 行布局``Text(`${JSON.stringify(``this``.scanResultObject,` `null``,` `'\u00A0\u00A0'``)}`)` `// 显示完整扫描结果``.fontColor(``this``.textColor)` `// 设置字体颜色``.fontSize(18)` `// 设置字体大小``.layoutWeight(1)` `// 设置布局权重``.copyOption(CopyOptions.LocalDevice);` `// 设置复制选项``}.constraintSize({ minHeight: 45 })` `// 设置最小高度约束``.justifyContent(FlexAlign.SpaceBetween)` `// 设置主轴对齐方式``.width(``'100%'``);` `// 设置宽度为100%` `}``.visibility(``this``.scanResult ? Visibility.Visible : Visibility.Hidden)` `// 根据扫描结果设置可见性``.alignItems(HorizontalAlign.Start)` `// 设置交叉轴对齐方式``.width(``'650lpx'``)` `// 设置宽度``.padding(`${``this``.basePadding}lpx`)` `// 设置内边距``.margin({ top: `${``this``.basePadding}lpx` })` `// 设置外边距``.borderRadius(10)` `// 设置边框圆角``.backgroundColor(Color.White)` `// 设置背景颜色为白色``.shadow({` `// 设置阴影``radius: 10,` `// 设置阴影半径``color:` `this``.shadowColor,` `// 设置阴影颜色``offsetX: 0,` `// 设置X轴偏移``offsetY: 0` `// 设置Y轴偏移``});` `}``.padding({ top: 20, bottom: 20 })` `// 设置内边距``.width(``'100%'``)` `// 设置宽度为100%``}.scrollBar(BarState.Off)` `// 禁用滚动条``.align(Alignment.Top)` `// 顶部对齐``.height(``'100%'``)` `// 设置高度为100%``}``}``.barHeight(0)` `// 设置选项卡条高度为0``.tabIndex(``this``.selectIndex)` `// 设置当前选中的索引``.width(``'100%'``)` `// 设置宽度为100%``.layoutWeight(1)` `// 设置布局权重``.onChange((index: number) => {` `// 选项卡变化事件``this``.selectIndex = index;` `// 更新选择的索引``});``}``.height(``'100%'``)` `// 设置高度为100%``.width(``'100%'``)` `// 设置宽度为100%``.backgroundColor(``"#f4f8fb"``);` `// 设置背景颜色``}``}` |
| --- | --- |



　　


