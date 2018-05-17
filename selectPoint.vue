<template>
    <div>
       <div style="overflow: hidden;">
           <section>
               <!--工具条-->
               <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
                   <el-form :inline="true">
                       <el-form-item>
                           <el-input v-model="pointName" placeholder="城市名称" @keyup.enter.native="search"></el-input>
                       </el-form-item>
                       <el-form-item>
                           <el-button type="primary" @click="search">搜索</el-button>
                       </el-form-item>
                       <el-form-item>
                           <el-input v-model="latlngCurrent" placeholder="坐标值" style="width: 360px;"></el-input>
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
    export default{
        props: {
            mapcenter: {
                type: String,
                default: '34.759666,113.648071'
            },
            oldmarker: {
                type: String,
                default: ''
            }
        },
        data: function () {
            return {
                pointName: '邓州',
                qqmap: null,
                newMarker: null,
                latlngCurrent:''
            }
        },
        mounted(){
            let that=this;
            if(typeof(qq)=='object'){
                that.createMap();
            }else {
                this.loadQQmap();
            }
            window.onMapFileLoad=function () {
               console.log('qqmap加载完成')
                that.createMap();
            }
        },
        methods: {
            loadQQmap(){
                let script = document.createElement("script");
                //设置标签的type属性
                script.type = "text/javascript";
                //设置标签的链接地址
                script.src = "https://map.qq.com/api/js?v=2.exp&callback=onMapFileLoad";
                //添加标签到dom
                document.body.appendChild(script);
            },
            search(){
                let that = this;
                //调用地址解析类
                let geocoder = new qq.maps.Geocoder({
                    complete: function (result) {
                        that.qqmap.setCenter(result.detail.location);
                        that.qqmap.setZoom(13)
                    }
                });
                let address = this.pointName;
                //通过getLocation();方法获取位置信息值
                console.log('搜索地址：',address)
                geocoder.getLocation(address);
            },

            createMap(){
                let that = this;
                that.qqmap = new qq.maps.Map(document.getElementById("qqmapCont"), {
                    center: new qq.maps.LatLng(that.mapcenter.split(',')[0], that.mapcenter.split(',')[1]),      // 地图的中心地理坐标。
                    zoom: 13,  // 地图的中心地理坐标。
                });
                that.createMarker();
            },
            createMarker(){
                let that = this;
                let premarker = null;
                if (this.oldmarker) {
                    console.log('编辑模式：', this.oldmarker);
                    that.qqmap.panTo(new qq.maps.LatLng(that.mapcenter.split(',')[0], that.mapcenter.split(',')[1]));
                    premarker = new qq.maps.Marker({
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
                    //调用searchLocalCity();方法    根据用户IP查询城市信息。
                    citylocation.searchLocalCity();
                }
               // 地图点击事件
                qq.maps.event.addListener(that.qqmap, 'click', function (event) {
                    premarker.setMap(null);
                    that.$emit('mapclick', event.latLng); // 地图点击坐标 传递给父组件
                    that.latlngCurrent= event.latLng.lat+','+ event.latLng.lng;
                    that.newMarker = event.latLng;
                    let marker = new qq.maps.Marker({
                        position: event.latLng,
                        map: that.qqmap
                    });
                    qq.maps.event.addListener(that.qqmap, 'click', function (event) {
                        marker.setMap(null);
                    });
                });
                // 地图鼠标滑动事件
                let $lngTipsBox=document.querySelector('.lngTips');
                let mapBoxWidth=window.getComputedStyle(document.querySelector('.mapWrap')).width;
                qq.maps.event.addListener(that.qqmap, 'mousemove', function (event) {
                    $lngTipsBox.style.display='block';
                    $lngTipsBox.style.top=(event.pixel.y+10)+'px';
                    $lngTipsBox.style.left = (event.pixel.x + 15) + 'px';
                    $lngTipsBox.innerText= '点击选择此坐标：'+event.latLng.lat + ',' + event.latLng.lng;
                });
                // 鼠标离开
                qq.maps.event.addListener(that.qqmap, 'mouseout', function (event) {
                    $lngTipsBox.style.display = 'none';
                });
            }


        },
    }
</script>

<style>
    .qqmap { width: 900px; height: 600px; }
    .mapWrap{ position: relative; width: 700px; height: 600px; overflow: hidden; }
    .lngTips{ display: none; width: 255px; height: 40px; padding:5px 7px; overflow: hidden; position: absolute; left: 0; top: 0; z-index: 123456; background: #ffffff; border-radius: 5px;
        line-height: 20px; box-shadow: #eee 1px 1px 3px; border: #eee 1px solid; }
</style>
