<template>
  <div>
    <div style="overflow: hidden;">
      <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
          <el-form :inline="true">
            <el-form-item>
              <el-input v-model="inputVal" placeholder="城市名称" @keyup.native="inputChange"
                        style="width: 300px;"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="search">获取地理位置</el-button>
            </el-form-item>
            <el-form-item>
              <el-input v-model="latlngCurrent" placeholder="坐标值" style="width: 246px;"></el-input>
            </el-form-item>
          </el-form>
        </el-col>
      </section>
    </div>
    <div class="mapWrap">
      <div class="qqmap" id="qqmapCont">
      </div>
      <div class="lngTips"></div>
    </div>

  </div>

</template>

<script>
  export default {
    props: {
      mapcenter: {
        type: String,
        default: '34.759666,113.648071'
      },
      oldmarker: {
        type: String,
        default: '34.759666,113.648071'
      },
      inputDefault: {
        type: String,
        default: ''
      }
    },
    data: function () {
      return {
        qqmap: '',
        premarker: '',
        marker: '',
        inputVal: '',
        latlngCurrent: ''
      }
    },
    mounted() {
      let that = this;
      this.inputVal = this.inputDefault;
      if (this.oldmarker) {
        this.latlngCurrent = this.oldmarker || this.mapcenter;
      }
      if (typeof(qq) == 'object') {
        that.createMap();
      } else {
        this.loadQQmap();
      }
      window.onMapFileLoad = function () {
        console.log('qqmap加载完成')
        that.createMap();
      }
    },
    watch: {
      oldmarker(newVal, oldVal) {
        if (newVal) {
          console.log(typeof newVal)
          this.latlngCurrent = newVal;
          this.qqmap&&this.createMarker()
        }
      },
      inputDefault(newVal, oldVal) {
        console.log('默认地址变成：', newVal)
        this.inputVal = newVal;
      }
    },
    methods: {
      loadQQmap() {
        let script = document.createElement("script");
        //设置标签的type属性
        script.type = "text/javascript";
        //设置标签的链接地址
        script.src = "https://map.qq.com/api/js?v=2.exp&callback=onMapFileLoad";
        //添加标签到dom
        document.body.appendChild(script);
      },
      search() {
        let that = this;
        //调用地址解析类
        let geocoder = new qq.maps.Geocoder({
          complete: function (result) {
            that.qqmap.setCenter(result.detail.location);
            that.qqmap.setZoom(12)
          }
        });
        let address = this.inputVal;
        //通过getLocation();方法获取位置信息值
        console.log('搜索地址：', address)
        geocoder.getLocation(address);
      },
      inputChange() {
        this.$emit('addr', this.inputVal); // 地图点击坐标 传递给父组件
      },
      createMap() {
        let that = this;
        this.qqmap = new qq.maps.Map(document.getElementById("qqmapCont"), {
          center: new qq.maps.LatLng(that.mapcenter.split(',')[0], that.mapcenter.split(',')[1]),      // 地图的中心地理坐标。
          zoom: 12,  // 地图的中心地理坐标。
        });
        setTimeout(() => {
          this.createMarker();
          this.bindMapEvent()
        }, 500)
      },
      createMarker() {
        let that = this;
        if (that.premarker) {
          that.premarker.setMap(null);
        }
        if (this.oldmarker) {
          console.log('编辑模式：', this.oldmarker);
          that.qqmap.setCenter(new qq.maps.LatLng(that.oldmarker.split(',')[0], that.oldmarker.split(',')[1]));
          that.premarker = new qq.maps.Marker({
            position: new qq.maps.LatLng(that.oldmarker.split(',')[0], that.oldmarker.split(',')[1]),
            map: that.qqmap
          });
        } else {
          //获取城市列表接口设置中心点
          let citylocation = new qq.maps.CityService({
            complete: function (result) {
              that.qqmap.setCenter(result.detail.latLng);
            }
          });
          //调用searchLocalCity();方法
          citylocation.searchLocalCity();
        }
      },
      bindMapEvent() {
        let that = this;
        // 地图点击事件
        qq.maps.event.addListener(that.qqmap, 'click', function (event) {
          that.marker && that.marker.setMap(null);
          that.premarker && that.premarker.setMap(null);
          that.$emit('mapclick', event.latLng); // 地图点击坐标 传递给父组件
          that.latlngCurrent = event.latLng.lat + ',' + event.latLng.lng;
          that.marker = new qq.maps.Marker({
            position: event.latLng,
            map: that.qqmap
          });
        });
        // 地图鼠标滑动事件
        let $lngTipsBox = document.querySelector('.lngTips');
        let mapBoxWidth = window.getComputedStyle(document.querySelector('.mapWrap')).width;
        qq.maps.event.addListener(that.qqmap, 'mousemove', function (event) {
          $lngTipsBox.style.display = 'block';
          $lngTipsBox.style.top = (event.pixel.y + 10) + 'px';
          $lngTipsBox.style.left = (event.pixel.x + 15) + 'px';
          $lngTipsBox.innerText = '点击选择此坐标：' + event.latLng.lat + ',' + event.latLng.lng;
        });
        // 鼠标离开
        qq.maps.event.addListener(that.qqmap, 'mouseout', function (event) {
          $lngTipsBox.style.display = 'none';
        });
      },
    },
  }
</script>

<style>
  .qqmap { width:100%; height:600px; }
  .mapWrap { position:relative; width:100%; height:600px; overflow:hidden; margin-top:15px; }
  .lngTips { display:none; width:255px; height:40px; padding:5px 7px; overflow:hidden; position:absolute; left:0; top:0; z-index:123456; background:#ffffff; border-radius:5px;
    line-height:20px; box-shadow:#eeeeee 1px 1px 3px; border:#eeeeee 1px solid; font-size:12px; }
</style>
