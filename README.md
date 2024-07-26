# **基于OpenLayers的WebGIS简单功能开发**

**1.上机目的**

了解并掌握将地理空间数据发布为数据服务的方法，掌握开源WebGIS开发技术。学会使用GeoServer进行数据服务发布，使用OpenLayers进行空间查询等WebGIS功能开发。

**2.上机软件环境**

OS：Win10等；

开发环境：VSCode等；

应用软件：OpenLayers、Geoserver等；

**3.上机内容**

（1）使用GeoServer将矢量数据发布为WMS/WFS图层；

（2）进行图层之间的叠加显示，进行图层显示控制；

（3）通过Openlayers调用发布的地图/数据服务，实现空间查询（包括地图查属性、属性查地图）功能（可选一功能实现）。

**4.上机要求**

要求独立完成上机实验，报告撰写格式规范。人机交互顺畅，界面友好。报告内容包括：上机实验目的、软件环境、上机要求、关键代码与步骤、上机小结。

**5.关键代码与步骤**

**（1）地图服务发布**

技术栈使用：Vue3 + ElementPlus（组件库） + Openlayers 9+ Geoserver（地图服务发布）

首先使用Geoserver创建工作空间，然后创建存储仓库，矢量数据源选择Shapefile，DBF字符集要选择GBK，防止中文乱码，选择WMS服务，然后点击发布。

要选择坐标参考系统，然后计算边框、经纬度边框范围。

![image-20240726203820081](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262038150.png)



**（2）前端加载地图**

![image-20240726203858188](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262038209.png)

使用Vue3的onMounted生命周期钩子使地图在组件挂载时调用，加载了OSM地图，自己发布的WMS图层，百度地图矢量注记。

ImageWMS中设置了地图Url，参数和服务类型，保证能够正确加载刚刚发布的地图。

要实现点击WMS图层出现要素属性弹窗，需要调用getFeatureInfoUrl函数，在其中设置点击的经纬度，地图缩放等级，坐标系，图层类型等，使用fetch请求Url，将response转换为json格式，并获取属性信息，加载弹出框，此外设置了鼠标事件，将弹出框设置在点击的位置，并设置了转换动画，移动缩放到点击的位置，见图3。

![image-20240726203916442](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262039463.png)

![image-20240726203928255](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262039307.png)

实现查询图层并高亮显示，使用了TileWMS，通过CQL\_FILTER对属性进行查询，并设置styles。还实现了删除高亮图层的功能，见图5。

![image-20240726203940409](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262039431.png)

![image-20240726203947368](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262039414.png)

还设置了切换地图的功能，有OSM图层，百度矢量地图，百度影像地图，代码实现如下：

![image-20240726203953700](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262039719.png)

![image-20240726203959419](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262039441.png)

通过创建TileLayer对象并通过XYZ对象设置地图的url实现加载，并调整地图的插入顺序，保证自己发布的中国资源枯竭城市图层在其它底图之上，避免被遮挡。

![image-20240726204017492](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262040527.png)

![image-20240726204022054](https://blogma.oss-cn-hangzhou.aliyuncs.com/blog/202407262040079.png)

**6.小结**

通过使用GeoServer发布地图，并使用Openlayers加载地图Url，并通过WMS图层的getMap方法获得图层及属性，并能够查询图层、高亮图层。还设置了跳转动画，完成了一个简单的WebGIS开发示例。其中使用GeoServer发布图层时要注意图层的坐标系，否则会出现数据加载失败的情况，此外由于Openlayers开发网上教程较少，需要阅读官方文档，开发遇到的问题较多，但最后都顺利解决，提升了自己解决问题的能力和开发水平。
