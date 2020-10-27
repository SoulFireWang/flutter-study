#苹果手机条码扫描不灵敏？

###问题说明
app的条码扫描模块用到了qr_code_scanner库，
但在执行扫描的时候发现，
1. android机型小米在条码扫描的时候效果较好。
2. iOS机型在条码扫描的时候反应非常迟缓，需要适当的大小，且居中位置才能扫描出来。

###苹果官方关于该问题的说明
https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png

####Why isn't my 1-Dimensional barcode being detected?（为什么条码扫描不出来）

First, check that the barcode is of a type supported by AVMetadataMachineReadableCodeObject.
Check that the barcode is in focus. Implementing tap-to-focus or repeatedly triggering auto-focus is recommended for best results on auto-focus enabled cameras. When using a fixed focus camera some codes might be too small to decode at the appropriate focus distance.
Check that the AVCaptureSessionPreset or AVCaptureDeviceFormat provides sufficient resolution for the density of barcode you're trying to scan.

```
1、检测条码类型是否是AVMetadataMachineReadableCodeObject支持的
2、检测条码是否聚焦。
    对于具有自动对焦功能的相机，采用点击聚焦和重复触发自动聚焦。
    当用固定对焦相机，有些条码对于解码来说太小了
3、检测AVCaptureSessionPreset和AVCaptureDeviceFormat是否能支持目标条码精度
```

####Why is my 1-Dimensional barcode not detected at all locations in the image?（为什么条码不能在任何位置被检测到）

2-dimensional barcode symbologies (Aztec, QR) have easily found marker patterns which allow them to be found efficiently anywhere in a digital image. 1-dimensional barcode symbologies (UPCE, Code39, Code93, Code128, EAN8, EAN13, PDF417*) have no such patterns. This is because they were designed to be read by a laser scanner.

```
1、二维码标致符号使他们能在图片中的任何位置被定位
2、条形码并没有这种定位符号，这是因为它是为镭射枪扫描所设计的。
```

The AVCaptureMetadataOutput 1-dimensional barcode detectors behave in a manner consistent with this design. The detector samples the image using a limited set of scan lines. The AVCaptureMetadataOutput API can detect a barcode only when a decodable portion of the barcode intersects one of these lines.

```
AVCaptureMetadataOutput有一种检测模式支持这种设计。
检测器用了有限的一组扫描基线。
当条码在扫描基线当中的时候才能被AVCaptureMetadataOutput检测到
```

The scanning pattern includes horizontal, vertical, near-horizontal, and near-vertical scan lines around the center of the AVCaptureMetadataOutput rectOfInterest ("Center"). See Table 1. You may wish to provide hints to the user about where to place the barcode within the camera preview for best results.
扫描模式包含横、竖、近横、近竖在AVCaptureMetadataOutput的感应中心区域。
```
你可以通过在屏幕上放置扫描框来获取更好的扫描效果
```

In some circumstances, when fewer symbologies are enabled and on more powerful hardware, the set of scan lines is expanded to cover a larger portion of the region of interest ("Additional").

在有些条件下，能在更大的范围被检测到。(支持预警符号和更好的硬件设备)

###为什么微信的扫码速度那么快?

https://mp.weixin.qq.com/s?__biz=MjM5NjM4MDAxMg==&mid=2655072716&idx=1&sn=ae6f8f713f1e9a049b6c4eee019f131f&scene=38#wechat_redirect1

微信对于扫码的过程进行了深度的优化（没有开源QBar，实现成本比较大）。

###解决方式：
>a、在扫描的时候在屏幕中间增加基线用于校准，这样能提高图片扫码质量。 
>b、在苹果机中，如果去掉二维码模式，苹果的AVFoundation会提升条码的检测范围（更多的扫描基线），从而更快的检测到条码