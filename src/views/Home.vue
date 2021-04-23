<template>
  <div class="home">
    <div>
      <h1>小图预览</h1>
      <image-self
        sourse="https://www.assassina.cn/images/index/shutterstock_1.png"
        class="luminous"
      >
        <div>不垂直居中预览</div>
      </image-self>
      <image-self
        sourse="https://www.assassina.cn/images/index/shutterstock_1.png"
        class="luminous"
        :verticalCenter="true"
      >
        <div>垂直居中预览</div>
      </image-self>
    </div>
    <div>
      <h1>大图预览</h1>
      <image-self
        sourse="http://localhost:9009/static/img/home_pic.eb816cc9.png"
        class="luminous"
      >
        <div>不垂直居中预览</div>
      </image-self>
      <image-self
        sourse="http://localhost:9009/static/img/home_pic.eb816cc9.png"
        class="luminous"
        :verticalCenter="true"
      >
        <div>垂直居中预览</div></image-self
      >
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import imageSelf from "@/components/PreviewImage.vue";

@Component({
  components: {
    imageSelf,
  },
})
export default class Home extends Vue {}
</script>
<style scoped>
.luminous {
  width: 360px;
  height: 240px;
  display: inline-block;
  position: relative;

}
.luminous+.luminous{
  margin-left: 100px;
}

</style>
