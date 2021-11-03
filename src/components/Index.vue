<template>
  <div>
    <el-dialog
      :visible="resultShow && searchResult.data"
      width="50%"
      :before-close="handleClose"
    >
      <el-descriptions :column="2" border>
        <el-descriptions-item label="垃圾名称">{{
          searchResult.name
        }}</el-descriptions-item>
        <el-descriptions-item label="所属分类">{{
          searchResult.type
        }}</el-descriptions-item>
        <el-descriptions-item label="基本概念">{{
          searchResult.description.Concept
        }}</el-descriptions-item>
        <el-descriptions-item label="基本包含">
          <el-tag
            size="small"
            v-for="(item, i) of searchResult.description.Including.split('、')"
            :key="i"
            >{{ item }}</el-tag
          >
        </el-descriptions-item>
        <el-descriptions-item label="投放要求">{{
          searchResult.description.Release_requirement
        }}</el-descriptions-item>
      </el-descriptions>
    </el-dialog>
    <el-dialog
      :visible="resultShow && !searchResult.data"
      width="50%"
      :before-close="handleClose"
    >
      <el-empty description="未查询到结果"></el-empty>
    </el-dialog>
    <div>
      <el-tag type="success" class="waste-count">
        当前所拥有的垃圾桶数量:
        {{ selfWasteList ? selfWasteList.length : 0 }}
      </el-tag>
      <el-divider></el-divider>
      <div style="margin-top: 15px" align="center">
        <el-autocomplete
          class="inline-input"
          v-model="searchValue"
          :fetch-suggestions="querySearch"
          placeholder="请输入内容 回车搜素"
          @select="handleSelect"
          @change="handleSelect"
        ></el-autocomplete>
      </div>
    </div>
    <div id="allWaste">
      <div class="main">
        <div v-for="(item, i) in selfWasteList" :key="i">
          <Waste
            :ref="item.name"
            @create="add"
            @remove="remove"
            :name="item.name"
            :nameColor="item.nameColor"
            :bucketColor="item.bucketColor"
            :coverColor="item.coverColor"
            :circleColor="item.circleColor"
          />
        </div>
        <div>
          <Waste empty @create="add" />
        </div>
      </div>
    </div>

    <div
      align="center"
      v-show="
        Array.isArray(!this.selfWasteList) || this.selfWasteList.length === 0
      "
    >
      <el-row>
        <el-button @click="comeDefault">来些默认的</el-button>
      </el-row>
    </div>
  </div>
</template>

<script>
import Waste from "./Waste.vue";

export default {
  name: "index",
  data() {
    return {
      resultShow: false,
      searchResult: {
        data: false,
        description: {
          Including: "",
        },
      },
      documentObj: null,
      restaurants: [],
      searchValue: "",
      selfWasteList: [],
      defaultWasteList: [],
    };
  },
  methods: {
    handleClose() {
      this.resultShow = false;
      this.searchResult = {
        data: false,

        description: {
          Including: "",
        },
      };
    },
    handleSelect(e) {
      try {
        this.$refs[e.name][0].active();
      } catch {
        if (this.searchValue.trim() === "") {
          return;
        }
        //网络请求接口
        const loading = this.$loading({
          lock: true,
          text: "请稍等...",
          spinner: "el-icon-loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
        this.axios
          .get("https://api.muxiaoguo.cn/api/lajifl?m=" + this.searchValue)
          .then((res) => {
            this.resultShow = true;
            if (res.status === 200 && res.data.code === "200") {
              let results = res.data.data;
              results.data = true;
              this.searchResult = results;
            }
          });
        loading.close();
      }
    },
    querySearch(queryString, cb) {
      var restaurants = this.restaurants;
      var results = queryString
        ? restaurants.filter((i) => i.value.indexOf(queryString) >= 0)
        : restaurants;
      // 调用 callback 返回建议列表的数据
      cb(results);
    },
    createFilter(queryString) {
      return (restaurant) => {
        return (
          restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) ===
          0
        );
      };
    },
    init() {
      this.defaultWasteList = [
        {
          name: "可回收垃圾",
          bucketColor: "#0000cc",
          coverColor: "#0000aa",
          contains: ["废报纸", "金属制品", "塑料"],
        },
        {
          name: "厨余垃圾",
          bucketColor: "#006600",
          coverColor: "#005500",
          contains: ["香蕉皮", "瓜果壳", "餐后垃圾"],
        },
        {
          name: "有害垃圾",
          bucketColor: "#cc0000",
          coverColor: "#aa0000",
          contains: ["医疗垃圾", "电池", "酸类物质"],
        },
        {
          name: "其他垃圾",
          bucketColor: "#ff9900",
          coverColor: "#ff8800",
          contains: ["建筑垃圾", "砖块", "其他"],
        },
      ];
      this.selfWasteList = JSON.parse(localStorage.getItem("selfWasteList"));
    },
    comeDefault() {
      localStorage.removeItem("selfWasteList");
      location.reload();
    },
    remove(name) {
      this.$confirm("要删我吗?", "确认一下", {
        confirmButtonText: "是",
        cancelButtonText: "否",
        type: "warning",
        // center: true,
      })
        .then(() => {
          for (let i = 0; i < this.selfWasteList.length; i++) {
            console.log(i, this.selfWasteList[i]);
            if (this.selfWasteList[i].name === name) {
              this.selfWasteList.splice(i, 1);
            }
          }
          localStorage.setItem(
            "selfWasteList",
            JSON.stringify(this.selfWasteList)
          );
          this.$message({
            type: "success",
            message: "删除完成!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    add() {
      this.$router.push({ path: "/add" });
    },
    setScroolFun() {
      this.documentObj = document.getElementById("allWaste"); // 获取DOM元素节点
      // 添加监听事件（不同浏览器，事件方法不一样，所以可以作判断，也可以如下偷懒）
      if (this.documentObj) {
        this.documentObj.addEventListener(
          "DOMMouseScroll",
          this.handlerMouserScroll,
          false
        );
        this.documentObj.addEventListener(
          "mousewheel",
          this.handlerMouserScroll,
          false
        );
      }
    },
    handlerMouserScroll(event) {
      let detail = event.wheelDelta || event.detail;
      let moveForwardStep = -1;
      let moveBackStep = 1;
      let step = 0;
      step = detail > 0 ? moveForwardStep * 35 : moveBackStep * 35;
      this.documentObj.scrollLeft = this.documentObj.scrollLeft + step;
    },
  },
  components: {
    Waste,
  },
  beforeDestroy() {
    if (!this.documentObj) return;
    this.documentObj.removeEventListener(
      "DOMMouseScroll",
      this.handlerMouserScroll
    );
    this.documentObj.removeEventListener(
      "mousewheel",
      this.handlerMouserScroll
    );
  },
  beforeMount() {
    document.title = "你是什么垃圾";
    this.init();
    if (!Array.isArray(this.selfWasteList)) {
      this.selfWasteList = this.defaultWasteList.slice();
      localStorage.setItem("selfWasteList", JSON.stringify(this.selfWasteList));
    }
    if (this.$route.params.needLoadLS) {
      //获取localStorage
      let selfWasteList = JSON.parse(localStorage.getItem("selfWasteList"));
      this.selfWasteList = selfWasteList;
    }
  },
  mounted() {
    this.setScroolFun();
    let arrays = this.selfWasteList.map((item) => {
      let tobj = item.contains.map((it) => {
        return {
          value: it,
          name: item.name,
        };
      });
      return tobj;
    });
    let restaurants = [];
    arrays.forEach((item) => (restaurants = restaurants.concat(item)));
    this.restaurants = restaurants;
  },
};
</script>

<style scoped>
.result {
  background-color: white;
  z-index: 1000;
  position: relative;
  top: 50%;
  left: 50%;
  border: 1px solid red;
  width: 500px;
  height: 500px;
}
::-webkit-scrollbar {
  height: 5px;
  background-color: rgba(0, 0, 0, 0);
}
::-webkit-scrollbar-thumb {
  border-radius: 5px;
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
}

#allWaste {
  display: flex;
  flex-direction: row;
  justify-items: center;
  white-space: nowrap;
  overflow-x: hidden;
}

#allWaste:hover {
  overflow-x: auto;
}
.main {
  width: 100%;
  display: flex;
  flex-direction: row;
  justify-content: first baseline;
}
.waste-count {
  font-size: 24px;
}
.el-tag {
  margin: 5px;
}
.el-select .el-input {
  width: 100px;
}
.el-input-group__append .el-button {
  width: 75px;
}
.input-with-select .el-input-group__prepend {
  background-color: #fff;
}
.el-input {
  position: relative;
  font-size: 14px;
  display: inline-block;
  width: 500px;
}
</style>
