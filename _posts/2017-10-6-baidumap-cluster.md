---
layout: post
title: "百度地图点聚合功能的使用"
categories: Android
tags: 百度地图
author: ZekeTian
---


* content
{:toc}


在本篇博客中将介绍百度地图的点聚合功能的基本使用方法，包括聚合图标的自定义、点击聚合点后分散展开效果以及一些注意事项。







# 1. 点聚合介绍

“点聚合”顾名思义，就是将多个点聚合在一起。之所以需要这个功能，是因为在使用百度地图展示大量的 Marker 时会可能导致 OOM ，同时大量的 Marker 聚集在一起视觉效果不佳。因此，我们可以使用点聚合功能，将多个点聚合在一起，避免在地图上显示密密麻麻的 Marker ，同时结合级别进行抽稀可以有效防止 OOM 的问题发生。

效果预览：

![实现的效果](http://ov8clk5cn.bkt.clouddn.com/17-10-6/94122325.jpg)



# 2. 点聚合功能的导入

**复制源码**

因为百度并没有将点聚合功能整合到 jar 包中，而是以源码的形式展示，因此我们需要下载百度地图的 demo 然后解压，在 java 文件中找到点聚合功能的源码（完整的包名 ：com.baidu.mapapi.clusterutil ），然后复制到自己的项目路径下。建议自己直接在资源管理器中复制，不要在 AS 中复制，因为通过 AS 复制会修改一些东西，导致最后修改起来比较麻烦。



**修改以适应项目环境**

将百度地图官方 demo 中的 layout 下的 text_bubble.xml 复制到自己项目中，然后将 demo 中 `res/values/styles.xml` 中如下的样式复制到自己项目中的 styles.xml 中。

```xml
   <style name="Bubble.TextAppearance.Dark" parent="android:TextAppearance.Medium">
        <item name="android:textSize">14sp</item>
        <item name="android:textColor">#ff7f7f7f</item>
    </style>

    <style name="Bubble.TextAppearance.Light" parent="Bubble.TextAppearance.Dark">
        <item name="android:textColor">#ffeeeeee</item>
    </style>

    <style name="ClusterIcon.TextAppearance" parent="android:TextAppearance">
        <item name="android:textColor">#ffeeeeee</item>
        <item name="android:textSize">16sp</item>
        <item name="android:textStyle">bold</item>
    </style>
```



# 3. 点聚合功能的使用

## 3.1 开始点聚合

- **创建 ClusterItem 的子类**

```java
public class MyMarkerItem implements ClusterItem {
    private final LatLng mPosition;
    private BitmapDescriptor mDescriptor;

    // 将设置 icon 的 BitmapDescriptor 传递进去, 从而自定义每个 Marker 图标
    public MyMarkerItem(LatLng latLng, BitmapDescriptor descriptor) {
        mPosition = latLng;
        mDescriptor = descriptor;
    }

    @Override
    public LatLng getPosition() {
        return mPosition;
    }

    @Override
    public BitmapDescriptor getBitmapDescriptor() {
        return mDescriptor;
    }
}
```

MyMarkerItem 是点聚合时的 Marker ，如果需要自定义点聚合时的每个 Marker ，则需要在构造方法中将 Marker 的 BitmapDescriptor 作为参数传递进去，否则，每个 Marker 的 icon 都通过 getBitmapDescriptor() 进行设置，样式单一。



- **创建点聚合管理器**

```java
// 创建点聚合管理器，mClusterManager 为 ClusterManager<MyMarkerItem> 的对象
mClusterManager = new ClusterManager<>(this, mBaiDuMap);
```



- **对地图事件进行监听**
  - 地图状态变化事件监听

  ```java
  // 通过让 mClusterMannger 来监听地图状态变化事件，可以在地图状态变化时实时地对 Marker 进行点聚合
  mBaiDuMap.setOnMapStatusChangeListener(mClusterManager);
  ```

  如果想要自己对地图状态进行管理的话， 可以重写 ClusterManager 监听地图状态变化开始和结束的方法，如下面代码所示

  ```java
  mClusterManager = new ClusterManager<MyMarkerItem>(this, mBaiDuMap) {
      @Override
      public void onMapStatusChangeFinish(MapStatus mapStatus) {
          super.onMapStatusChangeFinish(mapStatus);
          // 重写 onMapStatusChangeFinish() 方法从而可以在地图状态变化完成时处理自己的逻辑操作
      }
  };
  ```

  也可以重写 onMapStatusChangeStart() 方法（这两个方法在 ClusterManager 中都是空实现）

  

  - 地图点击事件监听

  ```java
  // 通过 mClusterManager 设置聚合点的点击事件
  mClusterManager.setOnClusterClickListener(new MyOnClusterClickListener());
  
  // 通过 mClusterManager 设置聚聚合点展开后的单个 Marker 的点击事件
  mClusterManager.setOnClusterItemClickListener(new MyOnClusterItemClickListener());
  
  // 只有让 mClusterManager 监听地图的 Marker 点击事件才能让 mClusterManager 正常监听 Cluster 
  // 以及 ClusterItem 的点击事件
  mBaiDuMap.setOnMarkerClickListener(mClusterManager);
  ```



- **添加 MyMarkerItem 到地图上**

```java
mBaiDuMap.addOverlay(item1); // 将 Marker 添加到地图上
```

添加完毕后，可以通过调用 `mClusterManager.cluster();` 来手动进行第一次聚合。如果没有让 mClusterManager 监听地图状态变化事件，则需要让 mBaiDuMap 监听地图事件变化，并在变化时调用 cluster() 方法，实时地进行点聚合。





## 3.2 自定义聚合点的 icon 样式 

如果不设置聚合点的 icon 样式，默认为显示聚合点包含的 Marker 数量，如果想将聚合点的 icon 修改成显示被聚合的众多 Marker 中的某一个的 icon 样式，则进行如下操作：

在 **com.baidu.mapapi.clusterutil.clustering.view.DefaultClusterRenderer.java ** 文件中将onBeforeClusterRendered 进行修改

```java
 protected void onBeforeClusterRendered(Cluster<T> cluster, MarkerOptions markerOptions) {
        int bucket = getBucket(cluster);
        BitmapDescriptor descriptor = mIcons.get(bucket);
        Collection<T> itemsCollection = cluster.getItems();

   		// 思路：在聚合点添加到 MapView 之前，从已经聚合的点中选择一个 Marker 出来，将其 icon 设置
   		// 成聚合点的 icon
   		
   
        // 方法一：将通过 getItems() 获得的 collection 强转成 List
        List<T> itemList = (List<T>) cluster.getItems();
        descriptor = itemList.get(0).getBitmapDescriptor();

/*
        // 方法二: 将 items 的集合转换成 ArrayList 然后获得第一个 Marker 的 BitmapDescriptor
        ArrayList<T> arrayList = new ArrayList<>(itemsCollection);
        descriptor = arrayList.get(0).getBitmapDescriptor();*/

   /*
        //
        // 方法三：通过 foreach 的方式访问 collection 中的元素，但是当获取到第一个 Marker 的
        // BitmapDescriptor 之后，再通过 break 退出循环即可

        for (T item:itemsCollection) {
            descriptor = item.getBitmapDescriptor();
            markerOptions.icon(descriptor);
            break;
        }
*/
        // 如果设置失败，则使用默认的样式（即显示聚合点聚合的 Marker 的数量）
        if (descriptor == null) {
            mColoredCircleBackground.getPaint().setColor(getColor(bucket));
            descriptor = BitmapDescriptorFactory.fromBitmap(mIconGenerator.makeIcon(getClusterText(bucket)));
            mIcons.put(bucket, descriptor);
        }
        // TODO: consider adding anchor(.5, .5) (Individual markers will overlap more often)
        markerOptions.icon(descriptor);
}
```



修改前后效果对比：

修改前：

![修改前](http://ov8clk5cn.bkt.clouddn.com/17-10-6/62902310.jpg)

修改后（聚合点的 icon 显示的是聚合点包含的 Marker 中的某一个的 icon 样式）：

![修改后](http://ov8clk5cn.bkt.clouddn.com/17-10-6/50580963.jpg)







## 3.3 修改聚合效果

- **修改开始聚合的点的个数**

在**com.baidu.mapapi.clusterutil.clustering.view.DefaultClusterRenderer.java**文件找到如下代码部分并进行修改

```java
	 /**
     * If cluster size is less than this size, display individual markers.
     */
    private static final int MIN_CLUSTER_SIZE = 2; // 设置成 3 个点开始聚合，小于等于 2 个点的则不聚合
```



- **修改开始聚合时图标之间的距离**

在**com.baidu.mapapi.clusterutil.clustering.algo.NonHierarchicalDistanceBasedAlgorithm.java**文件中找到如下代码并进行修改

```java
public static final int MAX_DISTANCE_AT_ZOOM = 300; // essentially 100 dp.
```

修改 MAX_DISTANCE_AT_ZOOM 的值 可以控制 Marker 距离来进行聚合，如上面设置成 300 ，则当两个 Marker 距离为 300dp 时就开始进行聚合



## 3.4 设置点击聚合点展开

点击聚合点后将聚合点包含的各个 Marker 分散显示在屏幕中，在实现这个效果之前要确保已经让 mClusterManager 监听了地图的 Marker 点击事件，即设置了`mBaiDuMap.setOnMarkerClickListener(mClusterManager);`

效果实现代码如下：

```java
mClusterManager.setOnClusterClickListener(new ClusterManager
        .OnClusterClickListener<MyMarkerItem>() {

    @Override
    public boolean onClusterClick(Cluster<MyMarkerItem> cluster) {

        Toast.makeText(getApplicationContext(), "点击了聚合点", Toast.LENGTH_SHORT).show();
        // 设置点击聚合点展开各个 Marker 效果
        // 获得点聚合的 Marker 条目
        List<MyMarkerItem> itemList = (List<MyMarkerItem>) cluster.getItems();

        // 创建地理范围 LatLngBounds 创建器
        LatLngBounds.Builder builder = new LatLngBounds.Builder();

        // 将 itemList 中的 Marker 包含到 builder 中
        for (MyMarkerItem item : itemList) {
            builder.include(item.getPosition());
        }

        // 创建 LatLngBounds
        LatLngBounds bounds = builder.build();

        // 创建地图更新状态
        MapStatusUpdate statusUpdate = MapStatusUpdateFactory.newLatLngBounds(bounds);

        // 更新地图状态
        mBaiDuMap.setMapStatus(statusUpdate);

        return false;
    }
});
```

