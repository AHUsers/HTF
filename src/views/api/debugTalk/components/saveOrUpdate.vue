<template>
  <div class="echart-pie-wrap">
    <div class="myEditorTop">[{{ isEdit ? '编辑' : '只读' }}] - [{{ debugTalkFrom.project_name }}]
      <el-button v-show="isEdit" type="success" @click="saveOrUpdate" style="margin-left: 10px;">保存</el-button>
    </div>
    <v-ace-editor
        ref="jsonEditorRef"
        v-model:value="debugTalkFrom.debug_talk"
        :options="options"
        class="debugTalk"
    ></v-ace-editor>
  </div>
</template>

<script lang="ts">
import {defineComponent, onMounted, reactive, ref, toRefs} from "vue";
import {useRoute} from 'vue-router'
import {useDebugTalkApi} from "/@/api/useAutoApi/debugTalk";
import {ElMessage} from "element-plus/es";
import {VAceEditor} from '/@/components/VaceEditor/index.js';


export default defineComponent({
  name: 'saveOrUpdateDebugTalk',
  components: {VAceEditor},
  setup() {
    const debugTalkRef = ref()
    const route = useRoute()
    const state = reactive({
      isEdit: true,
      editor: null,
      content: '',  // bulk内容
      debugTalkFrom: {
        id: null,
        debug_talk: '',
        project_name: ''
      },
      options: {
        fontSize: 14,
        mode: "ace/mode/python",
        theme: "ace/theme/monokai",
        enableAutoIndent: true, // 启用基本自动完成
        enableSnippets: true, // 启用代码段
        enableLiveAutocompletion: true, // 启用实时自动完成
        printMarginColumn: 30,
        displayIndentGuides: true, // 显示参考线
        // enableEmmet: true, // 启用Emmet
        tabSize: 4, // 标签大小
        useWorker: true, // 使用辅助对象
        showPrintMargin: false, //去除编辑器里的竖线
        // enableMultiselect: true, //     选中多处
        readOnly: false, // 是否只读
        showFoldWidgets: true, // 显示折叠部件
        fadeFoldWidgets: true, // 淡入折叠部件
        wrap: true, //换行
        animatedScroll: true,
        navigateWithinSoftTabs: true,
      },
      // headers

    });

    const initData = () => {
      if (route.query) {
        useDebugTalkApi().getDebugTalkInfo(route.query)
            .then(res => {
              state.debugTalkFrom.debug_talk = res.data.debug_talk
              state.debugTalkFrom.project_name = res.data.project_name
              state.debugTalkFrom.id = res.data.id
              state.isEdit = res.data.edit
              // initEditor(state.debugTalkFrom.debug_talk)
            })
      }
    }

    // 新增
    const saveOrUpdate = () => {
      useDebugTalkApi().saveOrUpdate(state.debugTalkFrom)
          .then(() => {
            ElMessage.success('操作成功');
          })
      // setBackEndControlRefreshRoutes() // 刷新菜单，未进行后端接口测试
    };

    onMounted(() => {
      initData()
    })

    return {
      initData,
      debugTalkRef,
      saveOrUpdate,
      ...toRefs(state),
    };
  },
})

</script>

<style lang="scss" scoped>
.echart-pie-wrap {
  //height: calc(100% - 45px);

  .myEditorTop {
    height: 45px;
    display: flex;
    align-items: center;
    background-color: #333;
    padding-left: 50px;
    color: rgb(241, 238, 8);
    font-weight: bold;
  }

}

.debugTalk {
  height: calc(100vh - 45px - 84px);
}
</style>
