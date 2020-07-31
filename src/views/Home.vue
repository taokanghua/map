<template>
  <div class="home">
    <div id="container" ref="map"></div>
    <div class="btn" v-show="!autoZoom" @click="autoNav">继续导航</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      map: {},
      isTraffic: true,
      getlocation: null, //上3个都是地图实例
      position: [0, 0], //当前点
      radius: 1000, //圆半径
      points: [
        //标点
        {
          local: [113.710154, 23.038352],
          name: "东莞市万江中学",
        },
        {
          local: [113.716965, 23.039868],
          name: "东莞市黄冈理想学校",
        },
        {
          local: [113.709119, 23.034849],
          name: "琼林小学",
        },
        {
          local: [113.716266, 23.035002],
          name: "家汇生活广场",
        },
        {
          local:[113.71498, 23.03877],
          name:'泓福大厦'
        }
      ],
      updateTime: 2000, //获取位置更新间隔时间
      autoZoom:false,
      flag: true, //circul只执行一次
      autoTimer:null
    };
  },
  methods: {
    getMap() {
      let that = this;
      this.map = new AMap.Map(this.$refs.map, {
        zoom: 11, //级别
        //center: [], //中心点坐标
        viewMode: "3D",
      });
      AMap.plugin(
        [
          "AMap.ToolBar",
          "AMap.Scale",
          "AMap.OverView",
          "AMap.MapType",
          "AMap.Geolocation",
        ],
        function() {
          that.map.addControl(new AMap.ToolBar());
          that.map.addControl(new AMap.Scale());
          that.map.addControl(new AMap.OverView({ isOpen: false }));
          that.map.addControl(new AMap.MapType());
          that.getlocation = new AMap.Geolocation({
            enableHighAccuracy: true, //是否使用高精度定位，默认:true
            timeout: 10000, //超过10秒后停止定位，默认：无穷大
            maximumAge: 0, //定位结果缓存0毫秒，默认：0
            convert: true, //自动偏移坐标，偏移后的坐标为高德坐标，默认：true
            showButton: true, //显示定位按钮，默认：true
            buttonPosition: "LB", //定位按钮停靠位置，默认：'LB'，左下角
            buttonOffset: new AMap.Pixel(10, 20), //定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
            showMarker: true, //定位成功后在定位到的位置显示点标记，默认：true
            showCircle: true, //定位成功后用圆圈表示定位精度范围，默认：true
            panToLocation: that.autoZoom, //定位成功后将定位到的位置作为地图中心点，默认：true
            zoomToAccuracy: that.autoZoom, //定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
          });
          // that.map.addControl(that.getlocation);
        }
      );
      // console.log(this.getlocation)
      // this.getlocation.getCurrentPosition();
      // AMap.event.addListener(this.getlocation, "complete", function(e) { //获取位置 并渲染
      //   that.position[0] = e.position.lng;
      //   that.position[1] = e.position.lat;
      //   let circle = new AMap.Circle({
      //     center: new AMap.LngLat(that.position[0], that.position[1]),
      //     radius: 1000, // ⚪半径
      //     fillColor: "#A0CFF6", //填充颜色
      //     fillOpacity: 0.2, //填充透明度
      //     strokeColor: "#fff", // 描边颜色
      //     strokeWeight: 2, //描边宽度
      //   });
      //   that.map.add(circle);
      // });
      this.updateLocalhost();

      this.map.on("click", function(e) {
        //点击地图某点 获取定位
        console.log(e);
      });
    },
    updateLocalhost() {
      let that = this;
      this.map.plugin("AMp.Geolocation", function() {
        that.getlocation = new AMap.Geolocation({
          enableHighAccuracy: true, //是否使用高精度定位，默认:true
          timeout: 10000, //超过10秒后停止定位，默认：无穷大
          maximumAge: 0, //定位结果缓存0毫秒，默认：0
          convert: true, //自动偏移坐标，偏移后的坐标为高德坐标，默认：true
          showButton: true, //显示定位按钮，默认：true
          buttonPosition: "LB", //定位按钮停靠位置，默认：'LB'，左下角
          buttonOffset: new AMap.Pixel(10, 20), //定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
          showMarker: true, //定位成功后在定位到的位置显示点标记，默认：true
          showCircle: true, //定位成功后用圆圈表示定位精度范围，默认：true
          panToLocation: that.autoZoom, //定位成功后将定位到的位置作为地图中心点，默认：true
          zoomToAccuracy: that.autoZoom, //定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
        });
        that.map.addControl(that.getlocation);
        setInterval(()=>{that.getlocation.getCurrentPosition()}, that.updateTime)
        // that.getlocation.getCurrentPosition()
        let promise = new Promise((resolve, reject)=>{
          AMap.event.addListener(that.getlocation, "complete", function(e) {
          //获取位置 并渲染
          that.position[0] = e.position.lng;
          that.position[1] = e.position.lat;
          console.log('我的位置:'+ that.position)
          if(that.position[0] != 0){
            resolve('success')
          }
        });
        })
        promise.then(res=>{
          // console.log(that.position)
          if(that.flag){
            that.circular()
            that.flag = false
          }
          //执行算距离方法
          that.runtime();
        })
          
      });
    },
    markPoint() {
      //标点
      let that = this;
      this.points.forEach((item) => {
        AMapUI.loadUI(["overlay/SimpleMarker"], function(SimpleMarker) {
          let iconTheme = "default";
          let iconStyles = SimpleMarker.getBuiltInIconStyles(iconTheme);
          new SimpleMarker({
            iconTheme,
            iconStyle: iconStyles[1],
            iconLabel: {
              innerHTML: "*",
              style: {
                color: "aqua",
              },
            },
            map: that.map,
            position: new AMap.LngLat(...item.local),
            label: {
              content: '<div class="point">' + item.name + "</div>",
            },
          });
        });
      });

      // //线
      // let object3Dlayer = new AMap.Object3DLayer({zIndex:10})
      // this.map.add(object3Dlayer)
      // let path = [
      //   new AMap.LngLat(113.71505,23.02911),
      //   new AMap.LngLat(113.710154, 23.038352)
      //   ]
      // let thinline = new AMap.Object3D.ThinLine({
      //   path,
      //   color:'red'
      //   //color:'#0055ff',
      // })
      // object3Dlayer.add(thinline)
    },
    runtime() { //持续计算点的位置
      let p2 = this.position;
      this.points.forEach((item) => {
        let p1 = item.local;
        let distance = AMap.GeometryUtil.distance(p1, p2);
        if (distance <= this.radius) {
          //线
          let object3Dlayer = new AMap.Object3DLayer({ zIndex: 10 });
          this.map.add(object3Dlayer);
          let path = [
            new AMap.LngLat(...p1),
            new AMap.LngLat(...p2),
          ];
          let thinline = new AMap.Object3D.ThinLine({
            path,
            color: "red",
            //color:'#0055ff',
          });
          object3Dlayer.add(thinline);
        }
      });
    },
    circular(){
        let circle = new AMap.Circle({
            center: new AMap.LngLat(...this.position),
            radius: this.radius, // ⚪半径
            fillColor: "#A0CFF6", //填充颜色
            fillOpacity: 0.1, //填充透明度
            strokeColor: "#fff", // 描边颜色
            strokeWeight: 2, //描边宽度
          });
        this.map.add(circle); //添加圆
    },
    touchHandle(){
      // 防抖
      function debounce(fn, delay){
        let pre = 0
        return function(){
          let cur = Date.now()
          if(cur - pre >= delay){
            fn()
            pre = Date.now()
            // console.log(cur,pre)
          }
        }
      }
      let that = this
      document.querySelector('#container').addEventListener('touchmove', debounce(function(){
       that.autoZoom = false
       console.log(that.autoZoom)
      }, 2000))
    },
    autoNav(){
      this.autoZoom = true
    }
  },
  watch:{
    position(n){
      this.circular()
    },
    autoZoom(n){
      if(n == false){
        console.log('clear')
        clearInterval(this.autoTimer)
        // this.autoTimer = setInterval(this.updateLocalhost, this.updateTime)
      }else{
        // clearInterval(this.autoTimer)
        // this.autoTimer = setInterval(this.updateLocalhost, this.updateTime)
      }
    },
    autoTimer(n){
      console.log(n)
    }
  },
  mounted() {
    this.getMap();

    this.updateLocalhost();
    this.markPoint();
    this.touchHandle()

    // this.autoTimer = setInterval(this.updateLocalhost, this.updateTime)

    this.map.on("complete", function(e) {
      //地图加载完成事件
      console.log(e);
    });
  },
};
</script>

<style lang="less" scoped>
.home {
  width: 100vw;
  position: relative;
  #container {
    width: 100%;
    height: 100vh;
  }
}

/deep/.amap-geolocation-con .amap-geo {
  margin-bottom: 30px !important;
}
/deep/.amap-marker-label {
  border: none;
  // background-color: rgba(230, 230, 230, 0.5);
  transform: translate(-25%, -50%);
  background-color: #303030;
}
/deep/.point {
  color: #fff;
}
.btn{
  width: 40vw;
  height: 30px;
  background-color: black;
  position: absolute;
  left: 50%;
  bottom: 3%;
  z-index: 55;
  transform: translateX(-50%);
  line-height: 30px;
  color: #fff;
  font-weight: bold;
  border-radius: 5px;
}
</style>
