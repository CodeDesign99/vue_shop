<template>
  <div class="map">
    <el-input
      v-model="addressKeyword"
      placeholder="请输入地址来直接查找相关位置"
    ></el-input>
    <!-- 给地图加点击事件getLocationPoint，点击地图获取位置相关的信息，经纬度啥的 -->
    <!-- scroll-wheel-zoom：是否可以用鼠标滚轮控制地图缩放，zoom是视图比例 -->
    <baidu-map
      :scroll-wheel-zoom="true"
      :center="location"
      :zoom="zoom"
      @click="getLocationPoint"
      ak="YOUR_APP_KEY"
    >
      <bm-view class="bmView"></bm-view>
      <bm-local-search
        :keyword="addressKeyword"
        :auto-viewport="true"
        style="display: none"
      ></bm-local-search>
    </baidu-map>
  </div>
</template>

<script>
export default {
  data() {
    return {
      location: {
        lng: 116.404,
        lat: 39.915
      },
      zoom: 12.8,
      addressKeyword: ''
    }
  },
  methods: {
    getLocationPoint(e) {
      this.lng = e.point.lng
      this.lat = e.point.lat
      // 创建地址解析器的实例
      let geoCoder = new BMap.Geocoder()

      // 获取位置地址对应坐标
      geoCoder.getPoint(this.addressKeyword, point => {
        this.slelectsdLng = point.lng
        this.slelectsdLat = point.lat
      })
      //利用坐标获取地址的详细信息
      geoCoder.getLocation(e.point, res => {
        console.log(res)
      })
    }
  }
}
</script>

<style scoped>
.bmView {
  width: 100%;
  height: 807px;
  flex: 1;
}
</style>
