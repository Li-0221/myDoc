# H5 & App

## 打包步骤

1.  将 H5 项目使用命令行打包生成 dist 目录

2.  打开 HBuilder X

    - 文件
    - 新建
    - 项目
    - 5+App
    - 默认模板
    - 创建

    生成目录如下

    ```
    css
    img
    js
    unpackage
    index.html
    manifest.json
    ```

3.  将 dist 目录下的 css img js 覆盖到 HBuilder X 生成的目录

4.  解决打包 App 的 bug

    在 index.html 加入如下代码

    ```javascript

    <script type="text/javascript">
    		document.addEventListener('plusready', function(a) { //等待plus ready后再调用5+ API：
    			// 在这里调用5+ API
    			var first = null;
    			plus.key.addEventListener('backbutton', function() { //监听返回键
    				//首次按键，提示‘再按一次退出应用’
    				if (!first) {
    					first = new Date().getTime(); //获取第一次点击的时间戳
    					// console.log('再按一次退出应用');//用自定义toast提示最好
    					// toast('双击返回键退出应用'); //调用自己写的吐丝提示 函数
    					window.history.go(-1);
    					//plus.nativeUI.toast("再按一次退出应用", {duration:'short'}); //通过H5+ API 调用Android 上的toast 提示框
    					setTimeout(function() {
    						first = null;
    					}, 1000);
    				} else {
    					if (new Date().getTime() - first < 1000) { //获取第二次点击的时间戳, 两次之差 小于 1000ms 说明1s点击了两次,
    						plus.runtime.quit(); //退出应用
    					}
    				}
    			}, false);
    		});
    </script>
    ```

5.  配置 manifest

    - 基础配置填写 AppID、应用名称、版本

    - 图标配置：设置 App 的图标

    - 启动界面配置：App 启动时展示的图片，自定义启动图

    - 权限配置：App 需要获取手机的权限

    配置参考

    ```json
    {
      "@platforms": ["android", "iPhone", "iPad"],
      "id": "H5C5F0825",
      /*应用的标识*/
      "name": "路径规划",
      /*应用名称，程序桌面图标名称*/
      "version": {
        "name": "1.0",
        /*应用版本名称*/
        "code": "100"
      },
      "description": "",
      /*应用描述信息*/
      "icons": {
        "72": "icon.png"
      },
      "launch_path": "index.html",
      /*应用的入口页面，默认为根目录下的index.html；支持网络地址，必须以http://或https://开头*/
      "developer": {
        "name": "",
        /*开发者名称*/
        "email": "",
        /*开发者邮箱地址*/
        "url": "" /*开发者个人主页地址*/
      },
      "permissions": {
        "Accelerometer": {
          "description": "访问加速度感应器"
        },
        "Audio": {
          "description": "访问麦克风"
        },
        "Cache": {
          "description": "管理应用缓存"
        },
        "Camera": {
          "description": "访问摄像头"
        },
        "Console": {
          "description": "跟踪调试输出日志"
        },
        "Device": {
          "description": "访问设备信息"
        },
        "Downloader": {
          "description": "文件下载管理"
        },
        "Events": {
          "description": "应用扩展事件"
        },
        "File": {
          "description": "访问本地文件系统"
        },
        "Gallery": {
          "description": "访问系统相册"
        },
        "Geolocation": {
          "description": "访问位置信息"
        },
        "Invocation": {
          "description": "使用Native.js能力"
        },
        "Orientation": {
          "description": "访问方向感应器"
        },
        "Proximity": {
          "description": "访问距离感应器"
        },
        "Storage": {
          "description": "管理应用本地数据"
        },
        "Uploader": {
          "description": "管理文件上传任务"
        },
        "Runtime": {
          "description": "访问运行期环境"
        },
        "XMLHttpRequest": {
          "description": "跨域网络访问"
        },
        "Zip": {
          "description": "文件压缩与解压缩"
        },
        "Barcode": {
          "description": "管理二维码扫描插件"
        },
        "Speech": {
          "description": "管理语音识别插件"
        },
        "Webview": {
          "description": "窗口管理"
        },
        "NativeUI": {
          "description": "原生UI控件"
        },
        "Navigator": {
          "description": "浏览器信息"
        },
        "NativeObj": {
          "description": "原生对象"
        }
      },
      "plus": {
        "splashscreen": {
          "autoclose": true,
          /*是否自动关闭程序启动界面，true表示应用加载应用入口页面后自动关闭；false则需调plus.navigator.closeSplashscreen()关闭*/
          "waiting": true /*是否在程序启动界面显示等待雪花，true表示显示，false表示不显示。*/
        },
        "popGesture": "close",
        /*设置应用默认侧滑返回关闭Webview窗口，"none"为无侧滑返回功能，"hide"为侧滑隐藏Webview窗口。参考http://ask.dcloud.net.cn/article/102*/
        "runmode": "normal",
        /*应用的首次启动运行模式，可取liberate或normal，liberate模式在第一次启动时将解压应用资源（Android平台File API才可正常访问_www目录）*/
        "signature": "Sk9JTiBVUyBtYWlsdG86aHIyMDEzQGRjbG91ZC5pbw==",
        /*可选，保留给应用签名，暂不使用*/
        "distribute": {
          "apple": {
            "appid": "",
            /*iOS应用标识，苹果开发网站申请的appid，如io.dcloud.HelloH5*/
            "mobileprovision": "",
            /*iOS应用打包配置文件*/
            "password": "",
            /*iOS应用打包个人证书导入密码*/
            "p12": "",
            /*iOS应用打包个人证书，打包配置文件关联的个人证书*/
            "devices": "universal",
            /*iOS应用支持的设备类型，可取值iphone/ipad/universal*/
            "frameworks": [],
            /*调用Native.js调用原生Objective-c API需要引用的FrameWork，如需调用GameCenter，则添加"GameKit.framework"*/
            "idfa": false
          },
          "google": {
            "packagename": "",
            /*Android应用包名，如io.dcloud.HelloH5*/
            "keystore": "",
            /*Android应用打包使用的密钥库文件*/
            "password": "",
            /*Android应用打包使用密钥库中证书的密码*/
            "aliasname": "",
            /*Android应用打包使用密钥库中证书的别名*/
            "permissions": [
              "<uses-feature android:name=\"android.hardware.camera\"/>",
              "<uses-feature android:name=\"android.hardware.camera.autofocus\"/>",
              "<uses-permission android:name=\"android.permission.ACCESS_COARSE_LOCATION\"/>",
              "<uses-permission android:name=\"android.permission.ACCESS_FINE_LOCATION\"/>",
              "<uses-permission android:name=\"android.permission.ACCESS_NETWORK_STATE\"/>",
              "<uses-permission android:name=\"android.permission.ACCESS_WIFI_STATE\"/>",
              "<uses-permission android:name=\"android.permission.CALL_PHONE\"/>",
              "<uses-permission android:name=\"android.permission.CAMERA\"/>",
              "<uses-permission android:name=\"android.permission.CHANGE_NETWORK_STATE\"/>",
              "<uses-permission android:name=\"android.permission.CHANGE_WIFI_STATE\"/>",
              "<uses-permission android:name=\"android.permission.FLASHLIGHT\"/>",
              "<uses-permission android:name=\"android.permission.MODIFY_AUDIO_SETTINGS\"/>",
              "<uses-permission android:name=\"android.permission.MOUNT_UNMOUNT_FILESYSTEMS\"/>",
              "<uses-permission android:name=\"android.permission.READ_LOGS\"/>",
              "<uses-permission android:name=\"android.permission.READ_PHONE_STATE\"/>",
              "<uses-permission android:name=\"android.permission.RECORD_AUDIO\"/>",
              "<uses-permission android:name=\"android.permission.VIBRATE\"/>",
              "<uses-permission android:name=\"android.permission.WAKE_LOCK\"/>",
              "<uses-permission android:name=\"android.permission.WRITE_SETTINGS\"/>"
            ],
            "abiFilters": ["armeabi-v7a", "arm64-v8a", "x86"]
          },
          /*使用Native.js调用原生安卓API需要使用到的系统权限*/
          "orientation": ["portrait-primary"],
          /*应用支持的方向，portrait-primary：竖屏正方向；portrait-secondary：竖屏反方向；landscape-primary：横屏正方向；landscape-secondary：横屏反方向*/
          "icons": {
            "ios": {
              "prerendered": true,
              /*应用图标是否已经高亮处理，在iOS6及以下设备上有效*/
              "auto": "",
              /*应用图标，分辨率：512x512，用于自动生成各种尺寸程序图标*/
              "iphone": {
                "normal": "",
                /*iPhone3/3GS程序图标，分辨率：57x57*/
                "retina": "",
                /*iPhone4程序图标，分辨率：114x114*/
                "retina7": "",
                /*iPhone4S/5/6程序图标，分辨率：120x120*/
                "retina8": "",
                /*iPhone6 Plus程序图标，分辨率：180x180*/
                "spotlight-normal": "",
                /*iPhone3/3GS Spotlight搜索程序图标，分辨率：29x29*/
                "spotlight-retina": "",
                /*iPhone4 Spotlight搜索程序图标，分辨率：58x58*/
                "spotlight-retina7": "",
                /*iPhone4S/5/6 Spotlight搜索程序图标，分辨率：80x80*/
                "settings-normal": "",
                /*iPhone4设置页面程序图标，分辨率：29x29*/
                "settings-retina": "",
                /*iPhone4S/5/6设置页面程序图标，分辨率：58x58*/
                "settings-retina8": "" /*iPhone6Plus设置页面程序图标，分辨率：87x87*/
              },
              "ipad": {
                "normal": "",
                /*iPad普通屏幕程序图标，分辨率：72x72*/
                "retina": "",
                /*iPad高分屏程序图标，分辨率：144x144*/
                "normal7": "",
                /*iPad iOS7程序图标，分辨率：76x76*/
                "retina7": "",
                /*iPad iOS7高分屏程序图标，分辨率：152x152*/
                "spotlight-normal": "",
                /*iPad Spotlight搜索程序图标，分辨率：50x50*/
                "spotlight-retina": "",
                /*iPad高分屏Spotlight搜索程序图标，分辨率：100x100*/
                "spotlight-normal7": "",
                /*iPad iOS7 Spotlight搜索程序图标，分辨率：40x40*/
                "spotlight-retina7": "",
                /*iPad iOS7高分屏Spotlight搜索程序图标，分辨率：80x80*/
                "settings-normal": "",
                /*iPad设置页面程序图标，分辨率：29x29*/
                "settings-retina": "" /*iPad高分屏设置页面程序图标，分辨率：58x58*/
              }
            },
            "android": {
              "mdpi": "",
              /*普通屏程序图标，分辨率：48x48*/
              "ldpi": "",
              /*大屏程序图标，分辨率：48x48*/
              "hdpi": "",
              /*高分屏程序图标，分辨率：72x72*/
              "xhdpi": "",
              /*720P高分屏程序图标，分辨率：96x96*/
              "xxhdpi": "",
              /*1080P 高分屏程序图标，分辨率：144x144*/
              "xxxhdpi": ""
            }
          },
          "splashscreen": {
            "ios": {
              "iphone": {
                "default": "",
                /*iPhone3启动图片选，分辨率：320x480*/
                "retina35": "",
                /*3.5英寸设备(iPhone4)启动图片，分辨率：640x960*/
                "retina40": "",
                /*4.0 英寸设备(iPhone5/iPhone5s)启动图片，分辨率：640x1136*/
                "retina47": "",
                /*4.7 英寸设备(iPhone6)启动图片，分辨率：750x1334*/
                "retina55": "",
                /*5.5 英寸设备(iPhone6 Plus)启动图片，分辨率：1242x2208*/
                "retina55l": "" /*5.5 英寸设备(iPhone6 Plus)横屏启动图片，分辨率：2208x1242*/
              },
              "ipad": {
                "portrait": "",
                /*iPad竖屏启动图片，分辨率：768x1004*/
                "portrait-retina": "",
                /*iPad高分屏竖屏图片，分辨率：1536x2008*/
                "landscape": "",
                /*iPad横屏启动图片，分辨率：1024x748*/
                "landscape-retina": "",
                /*iPad高分屏横屏启动图片，分辨率：2048x1496*/
                "portrait7": "",
                /*iPad iOS7竖屏启动图片，分辨率：768x1024*/
                "portrait-retina7": "",
                /*iPad iOS7高分屏竖屏图片，分辨率：1536x2048*/
                "landscape7": "",
                /*iPad iOS7横屏启动图片，分辨率：1024x768*/
                "landscape-retina7": "" /*iPad iOS7高分屏横屏启动图片，分辨率：2048x1536*/
              }
            },
            "android": {
              "mdpi": "",
              /*普通屏启动图片，分辨率：240x282*/
              "ldpi": "",
              /*大屏启动图片，分辨率：320x442*/
              "hdpi": "",
              /*高分屏启动图片，分辨率：480x762*/
              "xhdpi": "",
              /*720P高分屏启动图片，分辨率：720x1242*/
              "xxhdpi": "" /*1080P高分屏启动图片，分辨率：1080x1882*/
            },
            "androidStyle": "default"
          },
          "plugins": {
            "geolocation": {
              "system": {
                "__platform__": ["ios", "android"]
              }
            },
            "ad": {}
          },
          "ios": {
            "dSYMs": false
          }
        }
      },
      "screenOrientation": ["portrait-primary"]
    }
    ```

6.  App 打包

    - 发行
    - 原生 App-云打包
    - 勾选使用公共测试证书
    - 勾选打正式包
    - 打包

7.  生成 apk

## App 更新

1. 获取 App 版本

   ```typescript
   // 获取当前app的版本号（读取manifest配置的版本号）
   const code = plus.runtime.versionCode;
   ```

2. 使用版本号调用后端接口，接口返回更新 App 的 url

3. 下载新的 apk

   ```typescript
   export function download(newAppUrl) {
     // @ts-ignore
     var dtask = plus.downloader.createDownload(
       newAppUrl,
       {},
       function (d: { filename: any }, status: string | number) {
         //d 为下载的文件对象
         if (status == 200) {
           const path = d.filename;
           // @ts-ignore
           plus.runtime.install(path); // 安装下载的 apk 文件
         } else {
           //下载失败
           alert("安装包下载失败");
           // @ts-ignore
           plus.downloader.clear(); //清除下载任务
         }
       }
     );
     dtask.start(); //执行下载
   }
   ```
