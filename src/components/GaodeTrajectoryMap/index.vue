
<template>
  <div class="gaodeTrajectoryMap-content-div" id="container"/>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader';
export default {
  name: 'GaodeTrajectoryMap',
  components: {},
  props: {
    data: {
      type: Array,
      default: () => []
    },
  },
  data() {
    return {
      mapLoading:false,
      mapDataItems:[],
    }
  },
  computed: {},
  watch: {
    data:{
      handler(items){
        let that = this
        that.mapDataItems = items
        if(that.mapDataItems.length > 0){
          //启动页面
          that.initAMap();
        }
      },
      immediate:true, //立即执行
    }
  },
  created() {
  },
  mounted() {
    this.initAMap();
  },
  beforeDestroy() {
    this.clearMap()
  },
  unmounted() {
    this.clearMap()
  },
  methods: {
    clearMap(){
      document.querySelector(`canvas.amap-layer`)?.getContext('webgl')?.getExtension('WEBGL_lose_context')?.loseContext()
    },
    initAMap() {
      const that = this
      that.mapLoading = true
      that.clearMap()
      that.$nextTick(function(){
        try {
          window._AMapSecurityConfig = {
            securityJsCode: "「」",
          };
          AMapLoader.load({
            key: "", // 申请好的Web端开发者Key，首次调用 load 时必填
            version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
            plugins: [
              'AMap.Autocomplete',
              'AMap.PlaceSearch',
              'AMap.Scale',
              'AMap.OverView',
              'AMap.ToolBar',
              'AMap.MapType',
              'AMap.PolyEditor',
              'AMap.CircleEditor',
              'AMap.MoveAnimation'], //需要使用的的插件列表，如比例尺'AMap.Scale'，支持添加多个如：['...','...']
            AMapUI: {
              version: '1.1',
              plugins:['overlay/SimpleMarker']
            }
          })
            .then((AMap) => {
              //加载地图
              let map = new AMap.Map("container", {
                // 设置地图容器id
                viewMode: "2D", // 是否为3D地图模式
                zoom: 17, // 初始化地图级别
                center: [116.397428, 39.90923], // 初始化地图中心点位置
              });

              map.on('complete', function() {
                // alert('地图加载完成')
                that.mapLoading = false
                that.initPage(map)
              });
            })
            .catch((e) => {
              console.log(e);
            });
        }catch (error) {
          console.error('地图加载失败:', error)
        }
      })
    },
    initPage(map){
      const that = this
      that.mapLoading = true
      let defaultLatAndLonData = []
      that.mapDataItems.forEach(function(currentValue){
        defaultLatAndLonData.push([currentValue.lon,currentValue.lat])

      })

      let latAndLonData = []
      that.convertFrom(defaultLatAndLonData,function(data) {
        let lnglats = data
        lnglats.forEach(function(currentValue){
          let llData = []
          llData.push(currentValue.lng)
          llData.push(currentValue.lat)
          latAndLonData.push(llData)
        })
          /*判断只有再有经纬度坐标数据的情况下，才会创建组件实例*/
        console.log('latAndLonData>>>',JSON.stringify(latAndLonData))
          if(latAndLonData.length > 0){
            // 绘制轨迹
            let polyline = new AMap.Polyline({
              path: latAndLonData,
              showDir:true,
              strokeColor: "#A3AEAC",  //线颜色
              // strokeOpacity: 1,     //线透明度
              strokeWeight: 8,      //线宽
              lineWidth: 8,
              dirArrowStyle: true
              // strokeStyle: "solid"  //线样式
            });
            map.add(polyline);
            let passedPolyline = new AMap.Polyline({
              strokeColor: "#008360",  //线颜色
              strokeWeight: 8,      //线宽
            });
            map.add(passedPolyline);

            let carIcon = new AMap.Icon({
              // 图标尺寸
              size: new AMap.Size(25, 34),
              // 图标的取图地址
              image: 'car.png',
              // 图标所用图片大小
              imageSize: new AMap.Size(30, 35),
              // 图标取图偏移量
              imageOffset: new AMap.Pixel(0, 0)
            });

            console.log('latAndLonData[0]>>>',latAndLonData[0])
            let marker = new AMap.Marker({
              position: latAndLonData[0],
              icon: carIcon,
              offset: new AMap.Pixel(-13, -26),
            });
            map.add(marker);

            //移动轨迹
            marker.on('moving', function (e) {
              passedPolyline.setPath(e.passedPath);
            });
            map.setFitView(); // 根据覆盖物自适应展示地图
            marker.moveAlong(latAndLonData, {
              // 每一段的时长
              duration: 500,//可根据实际采集时间间隔设置
              circlable:true,//开启动画是否循环报错：
              // JSAPI2.0 是否延道路自动设置角度在 moveAlong 里设置
              autoRotation: true,
            });

          }

      })
    },
    convertFrom(LngLatArray, success) {
      const that = this
      var LngLatArray2 = [];
      // https://lbs.amap.com/api/javascript-api/reference/lnglat-to-address/#m_AMap.convertFrom
      // 将其他地图服务商的坐标批量转换成高德地图经纬度坐标。最多支持40对坐标。
      var size = 40;
      var pageNum = parseInt((LngLatArray.length + size - 1) / size);
      var convertNum = 0;
      for (var i = 0; i < pageNum; i++) {
        var LngLatArraySlice = LngLatArray.slice(i * size, (i + 1) * size);
        convert(LngLatArraySlice, i);
      }
      function convert(LngLatArray, n) {
        AMap.convertFrom(LngLatArray, 'gps', function (status, result) {
          if (status == 'complete') {
            LngLatArray2[n] = (result.locations);
            convertNum++;
            if (convertNum >= pageNum) {
              typeof success == "function" ? success([].concat.apply([], LngLatArray2)) : null;
            }
          }
        });
      }
    },
  }
}
</script>

<style scoped>
.gaodeTrajectoryMap-content-div {
    padding:0px;
    margin: 0px;
    width: 100%;
  height: calc(100vh - 160px);
  border: 1px solid red;
}
</style>
