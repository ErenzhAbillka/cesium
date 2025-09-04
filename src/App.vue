<template>
  <div id="cesiumContainer"></div>
</template>

<script setup lang="ts">
import * as Cesium from "cesium";
import { onMounted } from "vue";
import axios from "axios";

// 辅助函数：获取本地时间字符串，格式 YYYY-MM-DD HH:MM:SS
function getLocalTimeString(date: Date = new Date()): string {
  const yyyy = date.getFullYear();
  const MM = String(date.getMonth() + 1).padStart(2, "0");
  const dd = String(date.getDate()).padStart(2, "0");
  const hh = String(date.getHours()).padStart(2, "0");
  const mm = String(date.getMinutes()).padStart(2, "0");
  const ss = String(date.getSeconds()).padStart(2, "0");
  return `${yyyy}-${MM}-${dd} ${hh}:${mm}:${ss}`;
}

onMounted(async () => {
  // 【离线模式】不要设置 Ion token、不要用在线地形/OSM Buildings/默认底图
  const viewer = new Cesium.Viewer("cesiumContainer", {
    animation: false,
    timeline: false,
    geocoder: false,
    homeButton: false,
    sceneModePicker: false,
    navigationHelpButton: false,
    fullscreenButton: false,
    baseLayerPicker: false,
    imageryProvider: undefined, // 无底图；如有本地瓦片再换 UrlTemplateImageryProvider
    terrainProvider: new Cesium.EllipsoidTerrainProvider(), // 椭球地形，完全离线
  });

  // 可选：隐藏右下角版权栏（演示更清爽）
  (viewer.cesiumWidget.creditContainer as HTMLElement).style.display = "none";

  // 先给一个大致视角（稍后会 zoomTo tileset）
  viewer.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(118.9361152, 32.1109622, 1500),
    orientation: {
      heading: Cesium.Math.toRadians(0),
      pitch: Cesium.Math.toRadians(-40),
      roll: 0,
    },
  });

  // 关键：加载【本地】3D Tiles（路径以 public 为根，用绝对路径）
  // 确保文件实际存在：/public/models/YANGSAHN/Data/tileset.json
  let tileset: Cesium.Cesium3DTileset | undefined;
  try {
    tileset = await Cesium.Cesium3DTileset.fromUrl("/models/YANGSAHN/Data/tileset.json");
    viewer.scene.primitives.add(tileset);
    await viewer.zoomTo(tileset);
  } catch (e) {
    console.error("加载本地 3D Tiles 失败，请检查路径与资源完整性:", e);
  }

  // 每秒请求本地后端并渲染闪烁点（与你原逻辑一致）
  setInterval(async () => {
    try {
      const now = new Date();
      const thisSecond = getLocalTimeString(now);
      const prevSecond = getLocalTimeString(new Date(now.getTime() - 1000));

      const response = await axios.get(
        `http://localhost:8080/vibration/by-second-range?times=${encodeURIComponent(thisSecond)},${encodeURIComponent(prevSecond)}`
      );
      const events = response.data;

      if (Array.isArray(events)) {
        events.forEach((event: any) => {
          const entity = viewer.entities.add({
            position: Cesium.Cartesian3.fromDegrees(event.longitude, event.latitude, 0),
            point: { pixelSize: 10, color: Cesium.Color.RED.withAlpha(1) },
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

          // 闪烁 3 秒后移除
          let visible = true;
          const blink = setInterval(() => {
            visible = !visible;
            entity.point!.color = visible
              ? Cesium.Color.RED.withAlpha(1)
              : Cesium.Color.RED.withAlpha(0);
          }, 500);

          setTimeout(() => {
            clearInterval(blink);
            viewer.entities.remove(entity);
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
#cesiumContainer { width: 100%; height: 100vh; }
</style>