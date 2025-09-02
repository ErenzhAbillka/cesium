<template>
  <div id="cesiumContainer"></div>
</template>

<script setup lang="ts">
import * as Cesium from 'cesium'
import { onMounted } from 'vue'

onMounted(async () => {
  // 离线：不用 Ion/默认底图
  // Cesium.Ion.defaultAccessToken = '...'

  const viewer = new Cesium.Viewer('cesiumContainer', {
    animation: false,
    timeline: false,
    geocoder: false,
    homeButton: false,
    sceneModePicker: false,
    navigationHelpButton: false,
    fullscreenButton: false,
    baseLayerPicker: false,
    imageryProvider: undefined, // 暂时不要底图；有本地瓦片再换 UrlTemplateImageryProvider
    terrainProvider: new Cesium.EllipsoidTerrainProvider(),
  })

  // 可选：隐藏右下角版权栏（演示时常用）
  ;(viewer.cesiumWidget.creditContainer as HTMLElement).style.display = 'none'

  // 相机视角
  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(118.9361152, 32.1109622, 800),
    duration: 2.5,
    orientation: {
      heading: Cesium.Math.toRadians(0),
      pitch: Cesium.Math.toRadians(-40),
      roll: 0,
    },
  })

  // 事件：修正拼写
  const handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas)
  handler.setInputAction((movement) => {
    console.log('屏幕坐标', movement.position)
  }, Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK)

  // 示例标注/线（可离线）
  const label = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(118.9391152, 32.12014, 100),
    label: { text: '0米无振动,振动等级：0', font: '60px sans-serif', fillColor: Cesium.Color.WHITE },
  })

  const label2 = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(118.9360152, 32.12014, 100),
    label: { text: '78光振动,振动等级：二级', font: '40px sans-serif', fillColor: Cesium.Color.YELLOW },
  })

  const polyline = viewer.entities.add({
    polyline: {
      positions: Cesium.Cartesian3.fromDegreesArray([
        118.9391152, 32.12014,
        118.9360152, 32.12014,
        118.934503, 32.1205,
      ]),
      width: 6,
      material: Cesium.Color.YELLOW,
    },
  })

  // 关键：加载本地 3D Tiles（public 下的绝对路径）
  const tileset = await Cesium.Cesium3DTileset.fromUrl('/models/YANGSAHN/Data/tileset.json')
  viewer.scene.primitives.add(tileset)
  await viewer.zoomTo(tileset)

  // Wall 示例：修正 geometryInstances 传参
  const wall = new Cesium.GeometryInstance({
    geometry: new Cesium.WallGeometry({
      positions: Cesium.Cartesian3.fromDegreesArrayHeights([
        118.9360152, 32.12014, 100,
        118.93601,   32.12014, 100,
        118.9360152, 32.12014, 100,
      ]),
    }),
  })
  const wallAppearance = new Cesium.MaterialAppearance({
    material: Cesium.Material.fromType('Color'),
  })
  const addWallGeometry = new Cesium.Primitive({
    geometryInstances: wall, // ← 这里要传几何实例，不是外观
    appearance: wallAppearance,
  })
  viewer.scene.primitives.add(addWallGeometry)

  // 鼠标移动获取经纬度（修正拼写）
  const ellipsoid = viewer.scene.globe.ellipsoid
  handler.setInputAction((movement) => {
    const cartesian = viewer.camera.pickEllipsoid(movement.endPosition, ellipsoid)
    if (!cartesian) return
    const cartographic = Cesium.Cartographic.fromCartesian(cartesian)
    const lat = Cesium.Math.toDegrees(cartographic.latitude).toFixed(6)
    const lon = Cesium.Math.toDegrees(cartographic.longitude).toFixed(6)
    const alt = (viewer.camera.positionCartographic.height / 1000).toFixed(2)
    console.log(`经度:${lon} 纬度:${lat} 高度:${alt}km`)
  }, Cesium.ScreenSpaceEventType.MOUSE_MOVE)

  // ❌ 离线请删除这些联网的代码：
  // var osmBuildingsTileset = Cesium.createOsmBuildingsAsync()
  // viewer.scene.primitives.add(osmBuildingsTileset)
  // const osmBuildings = viewer.scene.primitives.add(new Cesium.createOsmBuildings())
})
</script>

<style>
#cesiumContainer { width: 100%; height: 100vh; }
html, body, #app { margin: 0; height: 100%; }
</style>


<!--
<template>
  <div id="cesiumContainer"></div>
</template>

<script setup lang="ts">
import * as Cesium from "cesium";
import { onMounted } from "vue";
import axios from "axios";

// 辅助函数：获取本地时间字符串，格式 YYYY-MM-DD HH:MM:SS
function getLocalTimeString(date: Date = new Date()): string {
  const now = date;
  const yyyy = now.getFullYear();
  const MM = String(now.getMonth() + 1).padStart(2, '0');
  const dd = String(now.getDate()).padStart(2, '0');
  const hh = String(now.getHours()).padStart(2, '0');
  const mm = String(now.getMinutes()).padStart(2, '0');
  const ss = String(now.getSeconds()).padStart(2, '0');
  return `${yyyy}-${MM}-${dd} ${hh}:${mm}:${ss}`;
}

let lastTimestamp = "";

onMounted(async () => {
  Cesium.Ion.defaultAccessToken =
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJlMDZjNzk2YS05ZDI5LTQxNDQtOWQyZi1lZTExYzk5OTIxM2UiLCJpZCI6MjM5ODg0LCJpYXQiOjE3MjU3Nzc1MTB9.16MVVV-m6ndRsZ1k6Vli0b4Xv2mNFKcYO04qDorcuqk";

  const viewer = new Cesium.Viewer("cesiumContainer", {
    terrainProvider: await Cesium.CesiumTerrainProvider.fromIonAssetId(1),
  });

  viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(118.9361152, 32.1109622, 800),
    orientation: {
      heading: Cesium.Math.toRadians(0),
      pitch: Cesium.Math.toRadians(-40),
      roll: 0,
    },
  });

  setInterval(async () => {
    try {
      const now = new Date();
      // 使用本地时间格式（东八区等与你电脑一致）
      const thisSecond = getLocalTimeString(now);
      const prevSecond = getLocalTimeString(new Date(now.getTime() - 1000));

      console.log("查询时间：当前秒 =", thisSecond, "前一秒 =", prevSecond);

      const response = await axios.get(
        `http://localhost:8080/vibration/by-second-range?times=${encodeURIComponent(thisSecond)},${encodeURIComponent(prevSecond)}`
      );
      const events = response.data;
      console.log("收到事件数据:", events);

      if (Array.isArray(events)) {
        events.forEach((event: any) => {
          // 这里不作去重判断，直接绘制每条数据的点
          console.log("准备绘制的事件:", event);

          const entity = viewer.entities.add({
            position: Cesium.Cartesian3.fromDegrees(event.longitude, event.latitude, 0),
            point: {
              pixelSize: 10,
              color: Cesium.Color.RED.withAlpha(1),
            },
            label: {
              text: `振动: ${event.distance}米`,
              font: "14px sans-serif",
              fillColor: Cesium.Color.YELLOW,
              style: Cesium.LabelStyle.FILL_AND_OUTLINE,
              outlineWidth: 2,
              verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
              pixelOffset: new Cesium.Cartesian2(0, -15),
            },
          });
          console.log("绘制点成功:", entity);

          let visible = true;
          const blinkInterval = setInterval(() => {
            visible = !visible;
            entity.point!.color = visible ? Cesium.Color.RED.withAlpha(1) : Cesium.Color.RED.withAlpha(0);
          }, 500);

          setTimeout(() => {
            clearInterval(blinkInterval);
            viewer.entities.remove(entity);
            console.log("移除点:", entity);
          }, 3000);
        });
      }
    } catch (error) {
      console.error("读取实时数据失败:", error);
    }
  }, 1000);
});
</script>

<style scoped>
#cesiumContainer {
  width: 100%;
  height: 100vh;
}
</style>
-->