# openLayers简易学习手册

[toc]

## [官网地址](https://openlayers.org/)

## 安装依赖

```nodejs
yarn add ol
```

## 常用对象

### [Map](https://openlayers.org/en/latest/apidoc/module-ol_Map-Map.html)

OpenLayers的核心组件，在实际的dom元素中渲染生成。构造器里的所有属性都可以通过set方法去生成。例. setTarget().

```html
<div id="map" style="width: 100%, height: 400px"></div>
```

```typescript
import Map from 'ol/Map.js';
const map = new Map({target: 'map'});
# const map = new Map({});
# map.setTarget('map')
```

### [View](https://openlayers.org/en/latest/apidoc/module-ol_View-View.html)

map的视图对象。配置视图中心以及缩放层级等参数

```typescript
import View from 'ol/View.js';

map.setView(new View({
  center: [0, 0],
  zoom: 2,
}));

```

### [Layer](https://openlayers.org/en/latest/apidoc/module-ol_layer_Layer.html)

图层对象。有4中基本类型的图层：

1. [ol/layer/Tile](https://openlayers.org/en/latest/apidoc/module-ol_layer_Tile-TileLayer.html) 瓦片图层
2. [ol/layer/Image](https://openlayers.org/en/latest/apidoc/module-ol_layer_Image-ImageLayer.html) 图片图层
3. [ol/layer/Vector](https://openlayers.org/en/latest/apidoc/module-ol_layer_Vector-VectorLayer.html) 矢量数据图层
4. [ol/layer/VectorTile](https://openlayers.org/en/latest/apidoc/module-ol_layer_VectorTile-VectorTileLayer.html) 矢量瓦片图层

```typescript
import TileLayer from 'ol/layer/Tile.js';

// ...
const layer = new TileLayer({source: source});
map.addLayer(layer);

```

### [Source](https://openlayers.org/en/latest/apidoc/module-ol_source_Source-Source.html)

图层的数据对象。可以加载免费的在线底图，也可以加载通过接口返回的矢量数据。

```typescript
import OSM from 'ol/source/OSM.js';

const source = OSM();

```

### 简单例子

```typescript
<template>
  <div id="map" style="width: 100%; height: 100%"></div>
</template>

<script setup lang="ts">
import { onMounted } from "vue";
import Map from "ol/Map.js";
import View from "ol/View.js";
import TileLayer from "ol/layer/Tile.js";
import XYZ from "ol/source/XYZ";

const initMap = () => {
  const key = "5d2682a1f7e86ddb85851bfaae6b5ea1";
  const map = new Map({
    layers: [
      new TileLayer({
        source: new XYZ({
          url: `http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=${key}`,
        }),
        zIndex: 0,
        visible: true,
      }), // 在线source生成的图层
    ],
    view: new View({
      projection: "EPSG:4326", // 默认3857 是投影坐标系，4326对应WGS84坐标系
      center: [104.06, 30.67],
      zoom: 16,
      maxZoom: 20,
      minZoom: 2,
    }),
    target: "map",
  });
};

onMounted(() => {
  initMap();
});
</script>

```

### [Feature](https://openlayers.org/en/latest/apidoc/module-ol_Feature-Feature.html)

矢量对象。包含geometry（[点](https://openlayers.org/en/latest/apidoc/module-ol_geom_Point-Point.html)，[线](https://openlayers.org/en/latest/apidoc/module-ol_geom_LineString-LineString.html)，[面](https://openlayers.org/en/latest/apidoc/module-ol_geom_MultiPolygon-MultiPolygon.html)等几何形状）对象，也可以携带属性信息。均可以自定义设置样式

```typescript
<template>
  <div id="map" style="width: 100%; height: 100%"></div>
</template>

<script setup lang="ts">
import { onMounted } from "vue";
import GeoJSON from "ol/format/GeoJSON";
import { Feature } from "ol";
import { Stroke, Fill, Style, Icon } from "ol/style";
import { Point, LineString, Circle } from "ol/geom";
import Map from "ol/Map.js";
import View from "ol/View.js";
import VectorLayer from "ol/layer/Vector";
import VectorSource from "ol/source/Vector";
import TileLayer from "ol/layer/Tile.js";
import XYZ from "ol/source/XYZ";

// 初始化几何图层的矢量数据对象
const geometrySource = new VectorSource({
  format: new GeoJSON(),
});
// 展示添加点线面的图层
const geometryLayer = new VectorLayer({
  source: geometrySource,
  zIndex: 9,
  visible: true,
  style: new Style({
    image: new Icon({
      anchor: [0.5, 1],
      size: [26, 34],
      src: new URL("@/assets/images/icons/check.png", import.meta.url).href,
    }),
    stroke: new Stroke({
      color: "#ffffff",
      width: 2,
      lineDash: [5, 5, 5, 5],
    }),
    fill: new Fill({
      color: "rgba(9, 135, 252, 0.3)",
    }),
  }),
});

const map = new Map({
  layers: [geometryLayer],
  view: new View({
    projection: "EPSG:4326", // 默认3857 是投影坐标系，4326对应WGS84坐标系
    center: [104.06, 30.67],
    zoom: 16,
    maxZoom: 20,
    minZoom: 2,
  }),
});

/**
 * 初始化底图图层以及将map对象绑定到真实dom上
 */
const initMap = () => {
  const key = "5d2682a1f7e86ddb85851bfaae6b5ea1";
  const baseLayer = new TileLayer({
    source: new XYZ({
      url: `http://t3.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}&tk=${key}`,
    }),
    zIndex: 0,
    visible: true,
  }); // 在线source生成的图层
  map.addLayer(baseLayer);
  map.setTarget("map");
};

/**
 * 绘制几何图层
 */
const drawGeometryLayer = () => {
  // 可以异步获取数据
  geometrySource.addFeature(
    new Feature({
      geometry: new Point([104.06, 30.67]),
      name:'test',
      status:'1', //可以设置自定义属性
    })
  ); // 点
  geometrySource.addFeature(
    new Feature({
      geometry: new Point([104.061, 30.671]),
    })
  ); // 点
  geometrySource.addFeature(
    new Feature({
      geometry: new LineString([
        [104.06, 30.67],
        [104.061, 30.671],
      ]),
    })
  ); // 线
  geometrySource.addFeature(
    new Feature({
      geometry: new Circle([104.06, 30.67], 0.005),
    })
  ); // 圆
};

onMounted(() => {
  initMap();
  drawGeometryLayer();
});
</script>

```

![图片](/images/feature.jpg)

### [Draw](https://openlayers.org/en/latest/apidoc/module-ol_interaction_Draw-Draw.html)

绘制矢量图形的事件。在地图上绘制点线面的时候会用到。

```typescript
<template>
  <div class="top-right-box">
    <Select placeholder="请选择" v-model:value="drawType">
      <Select.Option value="Point">点</Select.Option>
      <Select.Option value="LineString">线</Select.Option>
      <Select.Option value="Polygon">面</Select.Option>
      <Select.Option value="Circle">圆</Select.Option>
    </Select>
    <div class="tool-box" @click="addFeature">
      <img alt="" src="@/assets/images/icons/add.png" />
    </div>
    <div class="tool-box" @click="undo">
      <img alt="" src="@/assets/images/icons/jianqie.png" />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { inject, ref } from "vue";
import { Draw } from "ol/interaction";
import GeoJSON from "ol/format/GeoJSON";
import { Feature, Map } from "ol";
import VectorLayer from "ol/layer/Vector";
import { Select } from "ant-design-vue";
import { Options } from "ol/interaction/Draw";
import VectorSource from "ol/source/Vector";
import Geometry from "ol/geom/Geometry";

const map: Map = inject("map") as Map;
const layer: VectorLayer<VectorSource<Geometry>> = inject(
  "layer"
) as VectorLayer<VectorSource<Geometry>>;

const drawType = ref<string>("Point");
const drawFeatures = ref<Feature<Geometry>[]>([]);

/**
 * 在地图上绘制矢量图形
 */
const addFeature = () => {
  const source = layer.getSource();
  const draw = new Draw({
    source: source,
    type: drawType.value,
  } as Options);
  draw.on("drawend", (evt) => {
    if (evt.feature != null) {
      // source.addFeature(evt.feature);
      drawFeatures.value.push(evt.feature);
      const featureGeoJson = JSON.parse(
        new GeoJSON().writeFeature(evt.feature)
      );
      console.info(featureGeoJson); // json形式的矢量数据，可以传给后台保存
      map.removeInteraction(draw);
    }
  });
  map.addInteraction(draw);
};

/**
 * 在地图上删除绘制的矢量图形
 */
const undo = () => {
  const source = layer.getSource();
  drawFeatures.value.forEach((item: any) => {
    source?.removeFeature(item);
  });
};
</script>

<style lang="less" scoped>
.top-right-box {
  position: absolute;
  top: 94px;
  right: 24px;
}
</style>

```

### [Select](https://openlayers.org/en/latest/apidoc/module-ol_interaction_Select-Select.html)

地图选择事件。一般在地图上选择矢量对象时进行编辑或者弹窗提示时会用到。

```typescript
/**
 * 初始化图层选择对象
 */
const layerSelect = new Select({
  layers: [layer], // 定义选择对象生效的图层
  condition: click, // 选择事件，单选，多选，按键，鼠标移入移出等事件均可以设置
  style: new Style({
    image: new Icon({
      anchor: [0.5, 1],
      size: [26, 34],
      src: new URL("@/assets/images/icons/operate.png", import.meta.url).href,
    }),
  }), //可以用来设置选中样式
});
/**
 * 定义选中事件
 */
layerSelect.on("select", (event) => {
  if (event.selected.length) {
    currentFeature.value = event.selected[0];
    console.info(currentFeature);
  } else {
    console.info("移除选择状态");
  }
});
// 将选择对象添加至地图
map.addInteraction(layerSelect);
```

### [Modify](https://openlayers.org/en/latest/apidoc/module-ol_interaction_Modify-Modify.html)

修改对象。一般在地图上编辑矢量数据的时候会用到。

```typescript
<template>
  <div class="top-right-box">
    <VueSelect placeholder="请选择" v-model:value="drawType">
      <VueSelect.Option value="Point">点</VueSelect.Option>
      <VueSelect.Option value="LineString">线</VueSelect.Option>
      <VueSelect.Option value="Polygon">面</VueSelect.Option>
      <VueSelect.Option value="Circle">圆</VueSelect.Option>
    </VueSelect>
    <div class="tool-box" @click="addFeature">
      <img alt="" src="@/assets/images/icons/add.png" />
    </div>
    <div class="tool-box" @click="startModify">
      <img alt="" src="@/assets/images/icons/bianji.png" />
    </div>
    <div class="tool-box" @click="undo">
      <img alt="" src="@/assets/images/icons/jianqie.png" />
    </div>
  </div>
</template>

/**
 * 在地图上修改选中对象
 */
const startModify = () => {
  if (currentFeature.value) {
    // 定义编辑对象
    const lineFeature = layer.getSource()?.getFeatureById("line");
    const modify = new Modify({
      features: new Collection([currentFeature.value, lineFeature as Feature]), // 修改的集合，可以将与点关联的线矢量对象传进去
    });
    modify.on("modifystart", (evt) => {
      // 移动前的矢量数据，可以存下来以供还原
      const modifyBefore = evt.features.getArray();
      console.info(modifyBefore);
    });
    modify.on("modifyend", (evt) => {
      const modifyAfter = evt.features.getArray();
      // 移动后的矢量数据
      console.info(modifyAfter);
      // 清除选中状态
      layerSelect.getFeatures().clear();
      currentFeature.value = undefined;
      // 移除修改事件
      map.removeInteraction(modify);
    });
    map.addInteraction(modify);
  }
};
```