<template>
  <div class="javascriptEdit2" :style="getBackColor1">
    <div class="javascriptEdit0" :style="getBackColor2">
      <el-menu
          class="el-menu-vertical-demo"
          :collapse="true"
      >
        <el-menu-item @click="Back($event)">

          <el-tooltip class="item" effect="dark"
                      content="返回"
                      placement="top">
            <el-icon>
              <Back/>
            </el-icon>
          </el-tooltip>
        </el-menu-item>
        <div v-if="scriptLogView===false">
          <el-menu-item @click="saveCode($event)">
            <el-icon>
              <VideoPlay/>
            </el-icon>
            <template #title>测试、保存 脚本代码[CTRL+S]</template>
          </el-menu-item>
          <el-menu-item @click="defaultCode($event)">
            <el-icon>
              <Refresh/>
            </el-icon>
            <template #title>恢复到默认</template>
          </el-menu-item>
          <el-menu-item @click="formatCode($event)">
            <el-icon>
              <Connection/>
            </el-icon>
            <template #title><span>格式化脚本代码</span></template>
          </el-menu-item>
        </div>
        <div v-if="scriptLogView">
          <el-menu-item disabled>
            <el-icon>
              <VideoPlay/>
            </el-icon>
            <template #title>测试、保存 脚本代码[CTRL+S]</template>
          </el-menu-item>
          <el-menu-item disabled>
            <el-icon>
              <Refresh/>
            </el-icon>
            <template #title>恢复到默认</template>
          </el-menu-item>
          <el-menu-item disabled>
            <el-icon>
              <Connection/>
            </el-icon>
            <template #title><span>格式化脚本代码</span></template>
          </el-menu-item>
        </div>
        <el-menu-item @click.native="scriptLog($event)">
          <el-icon>
            <View v-if="scriptLogView===false"/>
            <Hide v-if="scriptLogView"/>
          </el-icon>
          <template #title><span v-if="scriptLogView===false">查看脚本日志</span><span
              v-if="scriptLogView">关闭脚本日志</span></template>
        </el-menu-item>
      </el-menu>
    </div>
    <div class="javascriptEdit">
      <JavaScriptEdit v-show="scriptLogView===false" ref="goCode" height="100%" :textType="'go'" :glyphMargin="false"
                      :readOnly="false" @keydown="handleKeyDown" offWordWrap="on"
                      Text="" Name="JavaScriptEdit"/>
      <div v-if="scriptLogView" style="height: 100%;width: 100%">
        <span> 仅显示最后1000条日志 </span>
        <Text ref="scriptLog" height="calc(100% - 20px)" :textType="'plaintext'" :glyphMargin="false"
              :readOnly="false" offWordWrap="on"
              Text="" Name="scriptLogView"/>
      </div>

    </div>
  </div>
</template>

<script>
import {defineComponent} from "vue";
import JavaScriptEdit from "./JavaScriptEdit.vue";
import {Base64DecodeStr, CallGoDo, StrBase64Encode} from "./CallbackEventsOn.js";
import {ElMessage} from "element-plus";
import Text from "./Request/Text.vue";

export default defineComponent({
  components: {Text, JavaScriptEdit},
  data() {
    return {
      scriptLogView: false,
      get theme() {
        return window.Theme.IsDark
      },
      set theme(newValue) {
        window.Theme.IsDark = newValue
      },
    }
  },
  mounted() {

  }, computed: {
    getBackColor1() {
      if (this.theme){
        return "background-color: rgb(45,52,54);z-index: 1000"
      }
      return "background-color:"+"#ffffff"+";z-index: 1000";
    },
    getBackColor2() {
      if (this.theme){
        return "background-color: rgb(45,52,54)"
      }
      return "background-color:"+"#ffffff"+";";
    }
  },
  methods: {
    SetCode(code) {
      this.$refs.goCode.SetCode(code)
      this.formatCode(null)
    },
    formatCode(event) {
      this.scriptLogView = false
      this.$nextTick(() => {
        CallGoDo("格式化Go脚本代码", {code: StrBase64Encode(this.$refs.goCode.GetCode())}).then(res => {
          this.$refs.goCode.SetCode(Base64DecodeStr(res))
        })
      })
    },
    Back() {
      window.vm.Settings.deselect()
    },
    saveCode(event) {
      this.scriptLogView = false
      this.$nextTick(() => {
        CallGoDo("格式化Go脚本代码", {code: StrBase64Encode(this.$refs.goCode.GetCode())}).then(res => {
          this.$refs.goCode.SetCode(Base64DecodeStr(res))
          this.$nextTick(() => {
            CallGoDo("保存Go脚本代码", {code: StrBase64Encode(this.$refs.goCode.GetCode())}).then(res => {
              if (res === "") {
                ElMessage({
                  message: '测试并保存脚本代码成功',
                  type: 'success',
                })
              } else {
                ElMessage({
                  message: res,
                  type: 'error',
                })
              }
              //this.$refs.goCode.SetCode(Base64DecodeStr(res))
            })
          })
        })
      })
    },
    defaultCode(event) {
      this.scriptLogView = false
      this.$nextTick(() => {
        CallGoDo("获取默认Go脚本代码", {code: StrBase64Encode(this.$refs.goCode.GetCode())}).then(res => {
          this.SetCode(Base64DecodeStr(res))
        })
      })
    },
    scriptLog(event) {
      this.scriptLogView = !this.scriptLogView
      if (!this.scriptLogView) {
        return
      }
      this.$nextTick(() => {
        CallGoDo("获取脚本日志", null).then(res => {
          this.$refs.scriptLog.SetCode(res)
        })
      })
    },
    handleKeyDown(event) {
      if (event.ctrlKey && (event.key === "S" || event.key === "s")) {
        this.saveCode(event)
      }
    },
  }
})
</script>
<style scoped>
.javascriptEdit {
  height: 100%;
  width: calc(100% - 60px);
  left: 60px;
  position: absolute;
}

.javascriptEdit0 {
  position: absolute;
  height: 100%;
  width: 80px;
  top: 0px;
  top: calc((100% - 180px) / 2)
}

.javascriptEdit2 {
  top: 0px;
  left: 0px;
  height:calc(100% - 0px);
  width: calc(100% - 0px);
  position: absolute;
}
</style>
