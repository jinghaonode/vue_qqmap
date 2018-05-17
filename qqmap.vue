<template>
    <div class="padd20">
        <mapselect :mapcenter="centerLatLng" :oldmarker="oldMarker" @mapclick="pointChange"></mapselect>
    </div>

</template>

<script type="text/ecmascript-6">
    let qqMapSelectPoint = require('./selectPoint.vue');
    export default{
        components: {
            mapselect: qqMapSelectPoint
        },
        data: function () {
            return {
                pointName: '郑州',
                qqmap: null,
                centerLatLng: '34.759666,113.648071',
                oldMarker: '34.759666,113.648071',
                newMarker: null
            }
        },
        mounted(){

        },
        methods: {
            pointChange(ev){
                console.log('捕获到点击坐标', ev)
            }
        }
    }
</script>

<style>
    .qqmap { width: 700px;
        height: 600px;
    }
</style>
