<template>
  <div id="cesiumContainer"></div>
  
  
  
  
</template>

 
<script setup lang='ts'>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
import {
  Cesium3DTile,
  Cartesian3,
  Matrix4,
  Transforms,
  Math as CesiumMath 
} from 'cesium'


onMounted(async () =>{
  Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJlMDZjNzk2YS05ZDI5LTQxNDQtOWQyZi1lZTExYzk5OTIxM2UiLCJpZCI6MjM5ODg0LCJpYXQiOjE3MjU3Nzc1MTB9.16MVVV-m6ndRsZ1k6Vli0b4Xv2mNFKcYO04qDorcuqk'
  //viewer是
  


  const viewer = new Cesium.Viewer('cesiumContainer',{
    //timeline: false,//时间轴
    //animation: false,//动画
    //geocoder: false,//搜索
    //homeButton: false,//主页
    //sceneModePicker: false,//投影方式
    //baseLayer: false,//图层选择
    //navigationHelpButton: false,//帮助
    //fullscreenButton: false,//全屏
   

  });
 
 
 

  viewer.camera.flyTo({
    destination:Cesium.Cartesian3.fromDegrees(118.93611520,32.1109622,800),
    //方向，俯仰角度
    duration: 4,
    orientation:{
      heading:Cesium.Math.toRadians(0),
      pitch:Cesium.Math.toRadians(-40),
      roll: (0),
    }
  })

  
  
  var handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
  handler.setInputAction(function (movement) {
    consolo.log('屏幕坐标',movement.position );
  },Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK);

  






  const label = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(118.93911520,32.120140,100),
    label: {
      text:"0米无振动,振动等级：0",
      font:'60px',
      fillColor:Cesium.Color.WHITE,
      //showBackground:true,
    }
  })
  viewer.zoomTo(label)


  const label2 = viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(118.93601520,32.120140,100),
    label: {
      text:"78光振动,振动等级：二级",
      font:'100px',
      fillColor:Cesium.Color.YELLOW,
      //showBackground:true,
    }
  })
  viewer.zoomTo(label2)

  

  const polyline = viewer.entities.add({
    polyline: {
      positions:Cesium.Cartesian3.fromDegreesArray([118.93911520,32.120140,118.93601520,32.120140,118.93450300,32.120500]),
      width: 6,
      material: Cesium.Color.YELLOW
    
    }
  })
  viewer.zoomTo(polyline)
  
  const line = viewer.entities.add({
    polyline: {
      positions:Cesium.Cartesian3.fromDegreesArray([118.93911520,32.120140,118.93601520,32.120140]),
      
      material: Cesium.Color.YELLOW
    
    }
  })
  viewer.zoomTo(line)

  const line1 = viewer.entities.add({
    polyline: {
      positions:Cesium.Cartesian3.fromDegreesArrayHeights([118.93911520,32.120140,0,118.93911520,32.120140,100]),
      
      material: Cesium.Color.WHITE
    
    }
  })
  viewer.zoomTo(line1)

  const line2 = viewer.entities.add({
    polyline: {
      positions:Cesium.Cartesian3.fromDegreesArrayHeights([118.93450300,32.120500,0,118.93450300,32.120500,100]),
      
      material: Cesium.Color.WHITE
    
    }
  })
  viewer.zoomTo(line2)

  const line3 = viewer.entities.add({
    polyline: {
      positions:Cesium.Cartesian3.fromDegreesArrayHeights([118.93601520,32.120140,0,118.93601520,32.120140,100]),
      
      material: Cesium.Color.WHITE
    
    }
  })
  viewer.zoomTo(line3)


  const tileset = await Cesium.Cesium3DTileset.fromUrl('../node_modules/YANGSAHN/Data/tileset.json')
  viewer.scene.primitives.add(tileset)
  viewer.zoomTo(tileset)  
  viewer.entities.add(polyline)

  


 

  var wall = new Cesium.GeometryInstance({
    geometry: new Cesium.WallGeometry({
      positions: Cesium.Cartesian3.fromDegreesArrayHeights([
      118.93601520,32.120140,100,
      118.93601,32.120140,100,
      118.93601,32.120140,100,
      118.93601520,32.120140,100,
      118.93601520,32.120140,100,
      ])
    })
  })
  var wallAppearance = new Cesium.MaterialAppearance({
    material: Cesium.Material.fromType('Color'),
  })
  var addWallGeometry = new Cesium.Primitive({

    geometryInstances: wallAppearance,
    appearance: wallAppearance
  })
  
  viewer.scene.primitives.add(addWallGeometry);
  
  
  
  var pArray = [118.93601520,32.120140,101,
                118.00000000,32.000000,100];

  var instance = new Cesium.GeometryInstancec({
    geometry: new Cesium.WallGeometry({
      position:Cesium.Cartesian3.fromDegreesArrayHeights(pArray),
      miniumHeights: [99.88888,99.99999]
    })
  });

  var material = Cesium.Material.fromType('Image');
    material.uniforms.image = 'photo3.png';
    viewer.scene.primitives.add(new Cesium.Primitive({
          geometryInstances: instance,
          appearance: new Cesium.MaterialAppearance({
                closed: false,
                material: material
          })
    }))

  
  var longitude_show = document.getElementById('longitude_show');
  var latitude_show = document.getElementById('latitude_show');
  var altitude_show = document.getElementById('altitude_show');
  var canvas = viewer.scene.canvas;
  handler.setInputAction(function(movement){
    var cartesian = 
    viewer.camera.pickEllipsoid(movement.endPosition,ellipsoid);
    if(cartesian){
      var cartographic = 
      viewer.scene.global.ellispsoid.cartesianToCartographic(cartesian);
      var lat_String = 
      Cesium.Math.toDegrees(cartographic.latitude).toFixed(6);
      var log_String = 
      Cesium.Math.toDegrees(cartographic.longtitude).toFixed(6);
      var alti_String = 
      (viewer.camera.positionCartographic.height/1000).toFixed(2);
      
      consolo("经度:"+Cesium.Math.toDegrees(cartographic.longtitude)+"纬度:"+Cesium.Math.toDegrees(cartographic.latitude)+"高度："+alti_String);
      longitude_show.innerHTML = log_String;
      latitude_show.innerHTML = lat_String;
      altitude_show.innerHTML = alti_String;
    }
  })
  


 
  
 
 // var czml = [
  //  {
  //    id: "document",
  //    name: "CZML Model",
  //    version: "1.0",
  //  },
  //  {
  //    id: "aircraft model",
   //  name: "Cesium Air",
  //    position: {
   //     cartographicDegress:[118.93911520,32.120140,100],
   //   },
   //   model: {
   //     gltf: "    "//模型路径URL
   //     scale: 2.0,
   //   },
   // },
 // ]

 var osmBuildingsTileset = Cesium.createOsmBuildingsAsync();
 viewer.scene.primitives.add(osmBuildingsTileset);

 const osmBuildings = viewer.scene.primitives.add(
    new Cesium.createOsmBuildings()
  );

  const czml = [
  {
    id: "document",
    name: "CZML Wall",
    version: "1.0",
  },
  {
    id: "wall",
    wall: {
      positions: {
        cartographicDegrees: [
        118.93601520,32.120140,100,
        118.93601520,30.120140,100,
        118.93601520,20.120140,100,
        118.93601520,10.120140,100,
        ],
      },
      material: {
        solidColor: {
          color: {
            rgba: [255, 0, 0, 150],
          },
        },
      },
    },
  },
];
  
 
})
</script>



    
    

