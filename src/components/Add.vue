<template>
  <div id="addBox">
    <div class="preview">
      <Waste
        disableContextmenu
        :name="waste.name"
        :nameColor="waste.nameColor"
        :bucketColor="waste.bucketColor"
        :coverColor="waste.coverColor"
        :circleColor="waste.circleColor"
      />
    </div>
    <div class="line"></div>
    <div id="param">
      <div id="from">
        <el-form label-width="80px">
          <el-form-item label="垃圾桶名称" :label-width="labelWidth">
            <el-input
              :width="inputWidth"
              v-model="waste.name"
              placeholder="垃圾桶名称"
            ></el-input>
          </el-form-item>
          <el-form-item label="手把颜色" :label-width="labelWidth">
            <el-color-picker
              @change="colorChange($event, 'circleColor')"
              @active-change="colorActiveChange($event, 'circleColor')"
              v-model="waste.circleColor"
              show-alpha
              :predefine="waste.predefine.circleColor"
            ></el-color-picker>
          </el-form-item>
          <el-form-item label="盖子颜色" :label-width="labelWidth">
            <el-color-picker
              @change="colorChange($event, 'coverColor')"
              @active-change="colorActiveChange($event, 'coverColor')"
              v-model="waste.coverColor"
              show-alpha
              :predefine="waste.predefine.coverColor"
            ></el-color-picker>
          </el-form-item>
          <el-form-item label="名称颜色" :label-width="labelWidth">
            <el-color-picker
              @change="colorChange($event, 'nameColor')"
              @active-change="colorActiveChange($event, 'nameColor')"
              v-model="waste.nameColor"
              show-alpha
              :predefine="waste.predefine.nameColor"
            ></el-color-picker>
          </el-form-item>
          <el-form-item label="桶颜色" :label-width="labelWidth">
            <el-color-picker
              @change="colorChange($event, 'bucketColor')"
              @active-change="colorActiveChange($event, 'bucketColor')"
              v-model="waste.bucketColor"
              show-alpha
              :predefine="waste.predefine.bucketColor"
            ></el-color-picker>
          </el-form-item>

          <el-form-item label="可投放垃圾" :label-width="labelWidth">
            <el-select
              v-model="waste.contains"
              :width="inputWidth"
              size="large"
              clearable
              multiple
              filterable
              allow-create
              default-first-option
              :remote-method="remoteSearchMethod"
              @change="selectLibChange"
              placeholder="输入物品名称，回车添加"
            >
              <el-option
                v-for="(item, i) in lib"
                :key="i"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
      <el-button-group>
        <el-button type="success" @click="add">添加</el-button>
        <el-button @click="back">返回</el-button>
      </el-button-group>
    </div>
  </div>
</template>

<script>
import Waste from "./Waste.vue";
export default {
  name: "add",
  data() {
    return {
      counter: {
        name: {
          num: 0,
          toast() {
            this.num++;
            if (this.num < 10) {
              return "给垃圾桶命个名吧!";
            } else if (this.num < 20) {
              return "垃圾桶不能没有名字";
            } else if (this.num < 30) {
              return "快要被你玩坏了!!!";
            } else if (this.num < 70) {
              setTimeout(() => {
                document.write("\
                <h1 align='center'>玩坏了，刷新一下吧</h1>");
                setTimeout(() => {
                  document.close();
                }, 1000);
              }, 1000);
              return "被你玩坏了";
            }
          },
        },
        contains: {
          num: 0,
          toast() {
            this.num++;
            if (this.num < 10) {
              return "垃圾桶不能放点东西吗?";
            } else if (this.num < 20) {
              return "需要预设点垃圾";
            } else if (this.num < 30) {
              return "快要被你玩坏了!!!";
            } else if (this.num < 70) {
              setTimeout(() => {
                document.write("\
                <h1 align='center'>玩坏了，刷新一下吧</h1>");
                setTimeout(() => {
                  window.top.close();
                }, 1000);
              }, 1000);
              return "被你玩坏了";
            }
          },
        },
      },
      inputWidth: "300px",
      labelWidth: "100px",
      lib: [],
      waste: {
        contains: [],
        name: "",
        predefine: {
          nameColor: ["#ffffff"],
          bucketColor: ["#00aa00"],
          coverColor: ["#329649"],
          circleColor: ["#555555"],
        },
        nameColor: "#ffffff",
        bucketColor: "#329649",
        coverColor: "#00aa00",
        circleColor: "#555555",
      },
    };
  },
  methods: {
    back() {
      window.history.back();
    },
    add() {
      if (this.waste.name === null || this.waste.name === "") {
        this.$message({
          message: this.counter.name.toast(),
          type: "warning",
        });
        return;
      }
      if (this.waste.contains === null || this.waste.contains.length === 0) {
        this.$message({
          message: this.counter.contains.toast(),
          type: "warning",
        });
        return;
      }
      let selfWasteList = JSON.parse(localStorage.getItem("selfWasteList"));
      if (selfWasteList.some((i) => i.name === this.waste.name)) {
        this.$message({
          message: "已经有这个了,换个名字吧!",
          type: "warning",
        });
        return;
      }
      const loading = this.$loading({
        lock: true,
        text: "请稍等...",
        spinner: "el-icon-loading",
        background: "rgba(0, 0, 0, 0.7)",
      });
      selfWasteList.push(this.waste);
      localStorage.setItem("selfWasteList", JSON.stringify(selfWasteList));
      loading.close();
      this.$router.push({
        path: "/",
        name: "index",
        params: { needLoadLS: true },
      });
    },
    remoteSearchMethod(e) {
      console.log("远程 " + e);
    },
    selectLibChange(e) {
      let item = e[e.length - 1];
      if (item.trim() === "") {
        this.waste.contains.remove(item);
      } else {
        if (!this.lib.some((i) => i.label === item)) {
          this.lib.push({ label: item, value: item });
        }
      }
    },
    print() {},
    colorChange(e, obj) {
      this.waste[obj] = e;
      if (this.waste.predefine[obj].indexOf(e) < 0) {
        this.waste.predefine[obj].unshift(e);
      }
    },
    colorActiveChange(e, obj) {
      this.waste[obj] = e;
    },
  },
  components: {
    Waste,
  },
  beforeDestroy() {},
  mounted() {
    Array.prototype.remove = function (val) {
      var index = this.indexOf(val);
      if (index > -1) {
        this.splice(index, 1);
      }
    };
  },
};
</script>

<style scoped>
#addBox {
  display: flex;
  flex-direction: row;
  justify-content: center;
}
#param {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
#from {
  width: 400px;
}
.line {
  border: 1px solid rgba(0, 0, 0, 0.1);
  margin: 0px 40px 0px 40px;
}
.el-select .el-input__inner {
  width: 300px;
}
.el-button {
  width: 200px;
}
</style>

