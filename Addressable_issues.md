# Addressable 踩坑记录

## 转向Addressable
首先放源码链接，[addressable source code](https://github.com/needle-mirror/com.unity.addressables).

Addressable(以下简称AA)，是Unity最新推出的资源管理方案，它与之前的AssetBundle(以下简称AB)不是替代关系，
而是建立在AssetBundle提供的基础功能上面的一套更加封装和易用的系统。

在之前使用AB的时候,由于AB提供的接口非常的基础和底层,这虽然有很高的自由度，但更常见的情况就是资源系统的
的每一处细节都要开发人员自己去实现，事无巨细,非常麻烦，bug也多。这也导致涌现出了很多资源管理框架。

后来Unity也推出了Addressable，在写下这篇文章的时候，最新版本是1.20.5,大概每两个月左右一个版本，
还处于活跃开发阶段，会修复很多问题或者增加一些功能。

按照Unity的描述 AA可以"Simplify your content management with Addressables"。实际上也是这样，
AA为开发者封装了很多细节，包括依赖管理，引用计数，打包，下载，更新等等。如果没有很特殊需求，
按照AA的教程，很快就能搭起一套可用的完全体资源管理系统。

AA可以在PackageManager中直接安装。建议可以直接转最新的版本，因为所谓的稳定版本其实也不是那么稳定，
我曾在项目中使用稳定版本，结果一个API调用会在真机上产生竞态条件卡死，导致不得不使用魔改过的版本，
六个月后的新版本中Unity的团队才修复了这个问题。

## 几个踩坑的地方  
本文不是一个教程，如果从0开始学习可以移步[官方文档](https://docs.unity3d.com/Packages/com.unity.addressables@0.8/manual/AddressableAssetsGettingStarted.html)

下面是几个在实际使用中遇到的坑点。

* 冗余资源
    - AA也会出现冗余资源的问题，如果将一个资源标记为AA资源，但是没有将它的间接引用标记为AA资源
    那么它的间接引用会作为依赖打进最终的包里。  
    比如很多包含Text组件的预制体都引用了一个字体，但是没有将这个字体标记，那么打包这些预制体的时候，
    每个预制体会打一份相同的冗余字体，这就会造成包膨胀。就字体来说，因为引用的地方很多，
    而且本身字体文件也挺大，有时候膨胀几百MB到1GB都有可能。


* 资源生命周期
    - 如果使用AA来加载资源，那么最好也使用AA提供的API来实例化游戏对象，并且使用AA的API来释放资源和销毁对象。  
    如果使用GameObject.Instantiate来实例化AA加载的资源，那么AA的资源引用计数是不对的，
    可能会造成资源被提前卸载等问题。当然也可以自己做引用计数，自己控制资源卸载，不过相对麻烦一些。
 

* 主动更新与非主动更新
    - AA默认会在初始化的时候去检查资源更新，并且下载Catalog文件。有时候我们并不想要这样的功能，
    而是想在需要下载更新的时候再去检查更新。那么就要打开AddressableAssetSetting中的
    "Only update catalogs manually"选项，这样的话只有手动调用更新API才会去更新Catalog。  

    ```C#

        Addressables.CheckForCatalogUpdates()  

    ```

    - 除了自己控制更新时机，可能有时候还要计算下载量，但是AA并没有提供这样的API，不排除将来会有，
    但是在这篇文章写下的时候还没有。要实现计算下载量，需要自己对比资源列表，并且自己计算下载量。
    其核心是根据远程的Catalog文件，构造一个ResourceLocationMap。然后利用这个ResourceLocationMap跟
    本地的资源文件做对比来计算大小。这个远程Catalog文件，可以通过比较hash去下载到本地缓存。

    ```C#
        private ResourceLocationMap GetCatalogMap(string catalogPath) {
            try {
                var txt = File.ReadAllText(catalogPath);
                // 构造最新资源的Catalog的实例
                var h = JsonUtility.FromJson<ContentCatalogData>(txt);
                var locator = h.CreateLocator();
                return locator;
            } catch (Exception e) {
                Debug.LogError(e.Message);
                return null;
            }
        }

        private long CalcDownLoadSize() {
            var locator = GetCatalogMap(pathToRemoteCatalogFile);
            if (locator == null) {
                return 0;
            }
            long size = 0;
            foreach (var kv in locator.Locations) {
                var key = kv.Key as string;
                if (key == null || !key.EndsWith(".bundle")) {
                    continue;
                }

                foreach (var location in kv.Value) {
                    var sizeData = location.Data as ILocationSizeData;
                    if (sizeData != null) {
                        // 与本地的ResourceManager对比下载量
                        size += sizeData.ComputeSize(location, Addressables.ResourceManager);
                    }
                }
            }

            return size;
        }

    ```

* 随包资源更新
    - AA给开发者提供的抽象是随时加载随时使用，不用预先下载，在需要时再异步获取。这样的设计在有些情况下是很好的。
    比如小体量的游戏，最终包去除掉资源，可以非常小，最小能有个十几MB。但是很多时候，是需要预先将资源随包分发的。
    虽然AA提供了一种方式实现这种功能，也就是Update Previous build ，但是那种更新方式相当复杂，而且限制很多，
    要拨动很多开关才行。  
    不过幸好AA预留了一个转换地址的函数，可以利用这个函数去将请求远程资源的地址映射到本地。
    ```C#
        Addressables.ResourceManager.InternalIdTransformFunc
    ```     
    - 然后就是将本地的资源放到StreamingAssets下面随包。然后在初始化的时候通过上面那个函数判断读取本地资源。

    ```C#
        private string Init() {
            Addressables.ResourceManager.InternalIdTransformFunc = TransFunc;
        }

        private string TransFunc(IResourceLocation location) {
            var path = location.InternalId;
            if (location.Data is AssetBundleRequestOptions) {
                // 尝试读取本地资源
                ...
                return path;
            }

            // 没有读取到本地资源，返回原始地址
            return location.InternalId; 
        }
    ```     

## ... 待续