<template>
  <div class="preview" v-show="sourse">
    <!-- 缩略图 -->
    <div
      :style="
        'background: url(' + sourse + ') no-repeat;background-size: 100% 100%;'
      "
      class="height-100"
    >
      <!-- 鼠标移入时添加cover -->
      <div class="cover" @click="showHide(true)">
        <slot></slot>
      </div>
    </div>
    <!-- 缩略图 结束-->

    <div
      :class="['bigCover', verticalCenter ? 'pureCenter' : 'topCenter']"
      v-show="show"
      :style="'user-select:none;'"
    >
      <!-- 预览图 菜单栏 -->
      <header>
        <ul>
          <li>
            <img
              src="../assets/缩小 (1).svg"
              :class="['icon']"
              alt=""
              @click="changeSize(false)"
            />
          </li>
          <li>
            <img
              src="../assets/放大 (1).svg"
              :class="['icon']"
              alt=""
              @click="changeSize(true)"
            />
          </li>
          <li>
            <img src="../assets/关  闭 (2).svg" alt="" class="icon" @click="showHide(false)">
          </li>
        </ul>
      </header>
      <!-- 预览图 菜单栏  结束-->

      <!-- 预览图大图 img -->

      <div :class="['bigImg']">
        <div
          @mousedown="mouseDown"
          :id="img"
          :style="`transform: scale(${size});`"
        >
          <img :src="sourse" alt="" :style="'pointer-events: none;'" />
        </div>
      </div>
      <!-- 预览图大图 img  结束-->
    </div>
  </div>
</template>

<script>
import uuid from "uuid";
const headerHeight = 64;
export default {
  props: {
    verticalCenter: {
      type: Boolean,
      default: false,
    },
    sourse: {
      //图片路径
      type: String,
    },
  },
  data() {
    return {
      status: false,
      size: 1, //预览图默认尺寸
      show: false, //是否打开预览图模式
      variation: 0.25, //预览图放大缩小变化量
      minSize: 1, //预览图可缩小最大百分比
      maxSize: 1 + 0.25 * 3, //预览图可放大最大百分比
      img: uuid.v4(),
      disX: "", // 鼠标距离图片left的距离
      disY: "", // 鼠标距离图片top的距离
      imgDom: null,
    };
  },
  mounted() {
    this.imgDom = document.getElementById(this.img);
  },
  watch: {
    show() {
      //监听预览图模式下禁止页面滚动
      this.size = this.minSize;
      this.imgDom.style.top = "";
      this.imgDom.style.left = "";
      if (this.show) {
        document.documentElement.style.overflow = "hidden";
        addEventListener("keyup", this.escAction);
        addEventListener("mousemove", this.mouseMove);
        addEventListener("mouseup", this.mouseUp);
      }
      //退出预览模式则恢复滚动条
      else {
        removeEventListener("keyup", this.escAction);
        removeEventListener("mousemove", this.mouseMove);
        removeEventListener("mouseup", this.mouseUp);
        document.documentElement.style.overflow = "";
      }
    },
  },
  methods: {
    mouseDown(e) {
      document.onmouseup = null;
      this.disX = e.clientX - this.imgDom.offsetLeft;
      this.disY = e.clientY - this.imgDom.offsetTop;
      window.aaa = this.imgDom;
      this.imgDom.classList.remove("trans");
      this.status = true;
    },
    mouseMove(e) {
      //1.用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
      //2.移动当前元素
      if (this.status && this.imgDom) {
        this.imgDom.style.top = e.clientY - this.disY + "px";
        this.imgDom.style.left = e.clientX - this.disX + "px";
      }
    },
    mouseUp() {
      this.imgDom.classList.add("trans");
      this.adjustImgPosition();
      this.status = false;
    },
    adjustImgPosition() {
      //鼠标抬起修改状态
      const elem = this.imgDom.getBoundingClientRect();
      const imgOriginWidth = elem.width / this.size;
      const imgOriginHeight = elem.height / this.size;
      const offsetWidth = (imgOriginWidth * (this.size - 1)) / 2;
      let offsetHeight = (imgOriginHeight * (this.size - 1)) / 2;
      // 图片宽度超过window
      if (elem.width > window.innerWidth) {
        // 确定当前状态没有left或者right没有贴边
        let spaceX = 0;
        if (window.innerWidth > imgOriginWidth) {
          spaceX = (window.innerWidth - imgOriginWidth) / 2;
        }
        if (elem.left > 0) {
          // 左边离边，贴左边
          this.imgDom.style.left = `${offsetWidth - spaceX}px`;
        } else if (elem.right < window.innerWidth) {
          // 右边离边，贴右边
          this.imgDom.style.left = `${
            (elem.width - window.innerWidth - offsetWidth + spaceX) * -1
          }px`;
        }
      } else {
        // 还原
        this.imgDom.style.left = `0px`;
      }
      // 图片高度超过window
      if (elem.height > window.innerHeight - headerHeight * 2) {
        let spaceY = 0;
        if (this.verticalCenter) {
          //如果可视区域高度小于元素高度
          if (window.innerHeight < imgOriginHeight) {
            spaceY = -headerHeight;
          } else {
            spaceY = (window.innerHeight - imgOriginHeight) / 2 - headerHeight;
          }
        }
        // 确定当前状态没有top或者bottom没有贴边
        if (elem.top > headerHeight) {
          // 上边离边，贴上边
          this.imgDom.style.top = `${offsetHeight - spaceY}px`;
        } else if (elem.bottom < window.innerHeight) {
          // 下边离边，贴下边
          this.imgDom.style.top = `${
            (elem.height -
              window.innerHeight -
              offsetHeight +
              headerHeight +
              spaceY) *
            -1
          }px`;
        }
      } else {
        // 还原
        if (this.verticalCenter) {
          //如果选择垂直居中,还原到顶部只需要将偏移量清零
          offsetHeight = 0;
        }
        this.imgDom.style.top = `${offsetHeight}px`;
      }
    },
    /**
     * @description:
     * 打开/关闭预览大图模式
     * @param {params:Boolean}
     * @return {}
     */
    showHide(params) {
      this.show = params;
    },
    changeSize(params) {
      if (params) {
        if (this.size < this.maxSize) this.size += this.variation;
      } else {
        if (this.size > this.minSize) this.size -= this.variation;
      }
      setTimeout(() => {
        this.adjustImgPosition();
      }, 300);
    },
    escAction(e) {
      //此处填写你的业务逻辑即可
      if (e.keyCode == 27) {
        this.showHide(false);
      }
    },
  },
};
</script>

<style lang="scss">
/*


控件优化:


  1.可以将icon替换为字体图标或者用多图条件语句切换当不可放大缩小时的显示样式

  2.可自主添加旋转功能

  3.verticalCenter bool类型可以控制该预览图是否垂直居中
*/
@import "./index.scss";
</style>
