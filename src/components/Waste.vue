<template>
  <div>
    <div v-if="empty" class="trush" @click="create">
      <div class="gaizi">
        <div
          class="circle"
          :style="{
            borderTop: '10px solid ' + colorEmpty,
            borderRight: '10px solid ' + colorEmpty,
            borderLeft: '10px solid ' + colorEmpty,
            borderBottom: circleColorNull,
          }"
        ></div>
        <div class="cover" :style="{ backgroundColor: colorEmpty }"></div>
      </div>
      <div
        class="trapezoid"
        :style="{ width: '175px', borderTop: '275px solid ' + colorEmpty }"
      ></div>
      <div class="name-div" @click="create">
        <!-- <img class="icon" src="../assets/icon_lj/shilaji.png" alt="" srcset="" /> -->
        <div class="name">
          <font size="6" color="white">点我创建一个</font>
        </div>
      </div>
    </div>
    <div
      v-else
      class="trush"
      @focus="contextmenuFocus"
      @contextmenu.prevent="contextmenuClick"
    >
      <div class="gaizi" :class="{ pick: pickbr }">
        <div
          class="circle"
          :style="{
            borderTop: '10px solid ' + circleColor,
            borderRight: '10px solid ' + circleColor,
            borderLeft: '10px solid ' + circleColor,
            borderBottom: circleColorNull,
          }"
        ></div>
        <div class="cover" :style="{ backgroundColor: coverColor }"></div>
      </div>
      <div
        class="trapezoid"
        :style="{ width: '175px', borderTop: '275px solid ' + bucketColor }"
      ></div>
      <div class="name-div" @click="active">
        <!-- <img class="icon" src="../assets/icon_lj/shilaji.png" alt="" srcset="" /> -->

        <div class="name">
          <font size="6" :color="nameColor">{{
            name.trim() === "" ? "XX垃圾桶" : name.trim()
          }}</font>
        </div>
      </div>
      <ul
        v-show="contextmenu.visible && !disableContextmenu"
        :style="{ left: contextmenu.left + 'px', top: contextmenu.top + 'px' }"
        class="contextmenu"
      >
        <li @click="create">再去创建一个</li>
        <li @click="active">盖子动一下</li>
        <li @click="active">哈哈哈</li>
        <li @click="remove">不要这个了</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "Waste",
  data() {
    return {
      contextmenu: {
        visible: false,
        top: 0,
        left: 0,
      },
      pickbr: false,
      circleColorNull: "none",
      colorEmpty: "#a1a3a6",
    };
  },
  props: {
    empty: {
      type: Boolean,
      default: false,
    },
    disableContextmenu: {
      type: Boolean,
      default: false,
    },
    //   定义垃圾桶的名称
    name: {
      type: String,
      default: "XX垃圾桶",
    },
    //   定义垃圾桶名称的字体颜色
    nameColor: {
      type: String,
      default: "#ffffff",
    },
    //   定义垃圾桶的颜色
    bucketColor: {
      type: String,
      default: "#329649",
    },
    //   定义垃圾桶盖子的颜色
    coverColor: {
      type: String,
      default: "#00aa00",
    },
    circleColor: {
      type: String,
      default: "#555555",
    },
  },
  methods: {
    showList() {
      alert("showme");
    },
    remove() {
      this.$emit("remove", this.name);
    },
    create() {
      this.$emit("create");
    },
    edit() {
      this.$emit("edit");
    },
    contextmenuFocus() {
      this.contextmenu.visible = false;
    },
    contextmenuClick(event) {
      this.contextmenu.visible = true;
      this.contextmenu.top = event.y;
      this.contextmenu.left = event.x;
    },
    active() {
      this.pickbr = true;
      setTimeout(() => {
        this.pickbr = false;
      }, 500);
    },
  },
  mounted() {
    document.addEventListener("click", (e) => {
      if (!this.$el.contains(e.target)) {
        this.contextmenu.visible = false; //点击其他区域关闭
      } else {
        this.contextmenu.visible = false;
      }
    });
    document.addEventListener("contextmenu", (e) => {
      if (!this.$el.contains(e.target)) {
        this.contextmenu.visible = false; //点击其他区域关闭
      } else {
        this.contextmenu.visible = true;
      }
    });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.contextmenu {
  margin: 0px;
  border-radius: 2px;
  z-index: 800;
  padding: 5px;
  /* padding-left: 2px; */
  width: 120px;
  /* height: 150px; */
  line-height: 35px;
  color: #000000;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  font-size: 18px;
  background-color: #f7f7f7;
  border: 1px solid #e7e7e7;
  list-style: none;
  position: absolute;
}
.contextmenu :hover {
  cursor: pointer;
  background-color: #d0d0d0;
}

.trush {
  margin: 50px;
  width: 275px;
  height: 400px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
}
.name-div {
  width: 275px;
  display: flex;
  flex-direction: row;
  justify-content: center;
  position: relative;
  top: -165px;
}

.icon {
  width: 275px;
  display: inline-block;
  text-align: center;
  position: relative;
  top: -165px;
}
.name {
  z-index: 700;

  width: 200px;
  display: inline-block;
  text-align: center;

  overflow: hidden;
  text-overflow: ellipsis;
  white-space: wrap;
}

.trapezoid {
  border-left: 40px solid transparent;
  border-right: 40px solid transparent;
  margin-left: 10px;
  position: relative;
  bottom: 0px;
}

.cover {
  height: 10px;
  width: 275px;
}
.circle {
  height: 20px;
  width: 75px;
  border-radius: 10px 10px 0 0;
  margin-left: 90px;
}
.pick {
  animation: pick 0.5s linear;
  animation-fill-mode: forwards;
}

@keyframes pick {
  0% {
    margin-bottom: 0px;
    transform: rotate(0deg);
  }
  25% {
    margin-bottom: 25px;
    transform: rotate(15deg);
  }
  50% {
    margin-bottom: 25px;
    transform: rotate(0deg);
  }
  75% {
    margin-bottom: 25px;
    transform: rotate(-5deg);
  }
  100% {
    margin-bottom: 0px;
    transform: rotate(0deg);
  }
}
</style>
