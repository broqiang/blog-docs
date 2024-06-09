+++
title = "flutter flavor 配置"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2024-06-10T01:53:53
updated_at = 2024-06-10T01:53:53
description = "flutter flavor 配置"
tags = ["flutter"]
+++

# flutter flavor 配置

[参考](https://dwirandyh.medium.com/create-build-flavor-in-flutter-application-ios-android-fb35a81a9fac)

## Flutter 中配置

### 创建 `lib/flavors/flavor_config.dart` 文件

> 文件内容只作为参考，按照实际的用到的去修改即可

```dart
class FlavorConfig {
  /// App 名字
  final String appName;

  /// Api 地址
  final String apiUrl;

  /// 当前环境
  final Flavor flavor;

  static FlavorConfig shared = FlavorConfig.create();

  factory FlavorConfig.create({
    String appName = "",
    String apiUrl = "",
    Flavor flavor = Flavor.dev,
  }) {
    return shared = FlavorConfig(appName: appName, apiUrl: apiUrl, flavor: flavor);
  }

  FlavorConfig({required this.appName, required this.apiUrl, this.flavor = Flavor.dev});
}
```

### 修改原来的 `lib/main.dart` 文件

为了防止文件名称混淆，这里直接将原来的 `lib/main.dart` 文件放在 `lib/flavors` 目录下，并且改名字为 `mainCommon.dart`

```dart
// 只修改了这里，其他内容还是该怎么样就怎么样
void mainCommon() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      /// 需要使用的时候就可以直接这样调用 
      body: Center(
        child: Column(
          children: [
            Text(FlavorConfig.shared.flavor.name),
            Text(FlavorConfig.shared.appName),
            Text(FlavorConfig.shared.apiUrl),
          ],
        ),
      ),
    );
  }
}
```

### 为每一个环境创建一个入口文件

- 创建 Dev `lib/flavors/main_dev.dart` 

```dart
import 'flavor_config.dart';
import 'main_common.dart';

void main() {
  FlavorConfig.create(
    appName: 'app dev',
    apiUrl: 'http://127.0.0.1:8081',
    flavor: Flavor.dev,
  );

  mainCommon();
}
```

- Prod `lib/flavors/main_prod.dart`

```dart
import 'flavor_config.dart';
import 'main_common.dart';

void main() {
  FlavorConfig.create(
    appName: 'app prod',
    apiUrl: 'https://yourapiserver.com',
    flavor: Flavor.prod,
  );

  mainCommon();
}
```

## 配置 iOS 端 Flavor

用 xcode 打开文件， vscode 上是右键点击个目录下的 `ios` 目录， 然后选择 `Open in Xcode` 选项， 
注意：`下面操作都是 xcode 中的`。

### 复制 Target

打开 `ios/Runner.xcworkspace` ， 然后点击左侧的 `Runner` Target， 选择 `Duplicate` ，复制一份新的，
修改名字为 `RunnerDev`， 原来的 `Runner` 保留，作为 prod 环境。

![flutter_flavor_2024-06-09-21-42-23](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-21-42-23.png)

### 重命名 Scheme

点击最顶部的 `Runner` 选择 `Manage Schemes`， 然后将 `Runner copy` 重命名为 `dev`。

![flutter_flavor_2024-06-09-21-46-43](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-21-46-43.png)

将默认的 `Runner` 重命名为 `prod`， 现在就有两个 scheme 了， `dev` 和 `prod`.

![flutter_flavor_2024-06-09-21-48-13](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-21-48-13.png)

### 添加 Build Mode 配置

点击左侧 PROJECT 下的 Runner， 然后将现在的 `Debug, Release，Profile` 3个文件全都复制一份（点击列表下面+号复制），
然后改名为`Debug-dev, Release-dev, Profile-dev`， 作为 dev 环境；
再将原来的三个文件改名为 `Debug-prod, Release-prod, Profile-prod`，作为 prod 环境。

![flutter_flavor_2024-06-09-22-06-59](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-22-06-59.png)

完成后是这样的

![flutter_flavor_2024-06-09-22-07-15](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-22-07-15.png)

### 为 Dev 环境配置 App ID

prod 就用原来的就好，如果有需要再修改,可以再原来的 identifier 后面加上 `.dev` 来给开发环境使用

点中左侧的 `RunnerDev` 然后选中 `Build Settings`， 然后在上面的 `Filter` 处搜索 `bundle id`， 将搜索出来的 `Product Bundle Identifier` 的值添加 `.dev`。

![flutter_flavor_2024-06-09-22-17-16](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-22-17-16.png)

### 修改 App Icon

准备好图标文件，如果没有可以在 [https://www.appicon.co](https://www.appicon.co) 制作

选中左侧目录的 `Assets.xcassets`， 然后将准备好的图标复制进去(如果是 [https://www.appicon.co](https://www.appicon.co) 生成的图标，
将 `AppIcon.appiconset` 整个目录拉过来即可)，改名为 AppIconDev。

找到 RunnerDev， 选中 Build Settings， 然后在上面的 `Filter` 处搜索 `primary App Icon`， 将值改成 AppIconDev。

![flutter_flavor_2024-06-09-23-05-49](https://image.broqiang.com/vscode/flutter_flavor_2024-06-09-23-05-49.png)

### 修改 App 名字

在 指定的 Runner Target 中，修改 `Display Name` 为想要的名称即可。

### iOS 完成

到现在， iOS 的配置已经完成， 可以通过下面命令，分别测试下 dev 和 prod 环境。

```bash
# dev 环境
flutter run -t lib/flavors/main_dev.dart --flavor dev

# prod 环境
flutter run -t lib/flavors/main_prod.dart --flavor prod
```

## 配置 Android 端 Flavor

### build.gradle 中添加 Flavor 配置

打开 `android/app/build.gradle`， 在 `android{}` 中添加如下配置 

```gradle
android {
  defaultConfig {
         ...
      }
  ...
  flavorDimensions "default"
  productFlavors {
      prod {
          dimension "default"
          resValue "string", "app_name", "YourAppName"
      }
      dev {
          dimension "default"
          applicationIdSuffix ".dev"
          resValue "string", "app_name", "YourAppNameDev"
          versionNameSuffix ".dev"
      }
  }
```

### 修改 App 名称

在 `android/app/src` 下创建两个文件（目录不存在，也创建） `dev/res/values/strings.xml` 和 `prod/res/values/strings.xml`

写入下面内容

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">YourAppNameDev</string>
</resources>
```

修改 `android/app/src/main/AndroidManifest.xml` 中的 `android:label` 为 `@string/app_name`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:label="@string/app_name"
        ......
```

### 完成

测试下打包

执行上面的 run 命令，这次选择设备选择 Android

## 编写 MakeFile

执行的命令有点长， 写个 MakeFile 来简化下

在项目根目录下创建文件 `Makefile`, 内容如下

```makefile
PHONY: run_dev run_dev_ios run_dev_android run_prod build_dev_apk build_prod_apk

FLAVOR_DEV=lib/flavors/main_dev.dart
FLAVOR_PROD=lib/flavors/main_dev.dart

run_dev:
	flutter run -t ${FLAVOR_DEV} --flavor dev 

# -d 后面跟的事设备的 id， 通过 flutter devices 查询， 根据实际的加入
run_dev_ios: 
	flutter run -t ${FLAVOR_DEV} --flavor dev -d 59b2a30e8b1edcc60809265090431cd1d9debeeb

# -d 后面跟的事设备的 id， 通过 flutter devices 查询， 根据实际的加入
run_dev_android:
	flutter run -t ${FLAVOR_DEV} --flavor dev -d rgvorccehepfpb5h

run_prod:
	flutter run -t ${FLAVOR_PROD} --flavor prod 

build_dev_apk:
	flutter build apk -t ${FLAVOR_DEV} --flavor dev

build_prod_apk:
	flutter build apk -t ${FLAVOR_PROD} --flavor prod
```

使用的时候，直接在项目根目录下执行 make 命令即可

```bash
# 运行 dev 环境
make run_dev

# 打包 dev 环境 的 apk 包
make build_dev_apk
```

## 编写 VsCode launch.json

在项目根目录下创建文件 `.vscode/launch.json`， 内容如下

写入下面内容

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "tsappDev",
            "request": "launch",
            "type": "dart",
            "program": "lib/flavors/main_dev.dart",
            "args": [
                "--flavor", 
                "dev",
            ]
        },
        {
            "name": "tsappProd",
            "request": "launch",
            "type": "dart",
            "program": "lib/flavors/main_prod.dart",
            "args": [
                "--flavor", 
                "prod",
            ]
        },
    ]
}
```

至此 Flavor 配置完成， 可以愉快的在各种环境中玩耍。