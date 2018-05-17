# vue_qqmap

> 腾讯地图vue组件，实现异步加载腾讯地图，坐标选取功能
![此处输入图片的描述][1]

         <mapselect :mapcenter="centerLatLng" :oldmarker="oldMarker" @mapclick="pointChange"></mapselect>
         

 - mapcenter：地图初始化时的中心坐标
 - oldmarker：可选项，用于marker点位编辑场景，显示已有的marker坐标，点击其他地方后自动消失
 - @mapclick：点击地图后的回调，参数为地图的坐标值

  [1]: http://mat1.gtimg.com/henan/0219/vue_qqmapaaaaa.png