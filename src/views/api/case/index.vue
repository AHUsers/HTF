<template>
  <div>
    <el-card shadow="hover">
      <div class="mb8">
        <div>
          <el-input v-model="listQuery.name" placeholder="请输入用例名称" style="max-width: 180px"></el-input>
          <el-button type="primary" class="ml10" @click="getList">
            <el-icon>
              <ele-Search/>
            </el-icon>
            查询
          </el-button>
          <el-button type="success" class="ml10" @click="onOpenSaveOrUpdate('save', null)">
            <el-icon>
              <ele-FolderAdd/>
            </el-icon>
            新增
          </el-button>
        </div>
      </div>
      <div class="mb8">
        <el-button type="text" class="" @click="openImportPage">
          <el-icon>
            <ele-FolderAdd/>
          </el-icon>
          导入
        </el-button>
      </div>
      <el-table
          border
          v-loading="tableLoading"
          :data="listData"
          style="width: 100%">
        <el-table-column type="selection" width="55" align="center"></el-table-column>

        <el-table-column
            v-for="field in fieldData"
            :key="field.fieldName"
            :label="field.label"
            :align="field.align"
            :width="field.width"
            :show-overflow-tooltip="field.show"
            :prop="field.fieldName"
        >
          <template #default="{row}">
            <template v-if="field.fieldName === 'name'">
              <el-button size="small"
                         type="text"
                         @click="onOpenSaveOrUpdate('update', row)">
                {{ row[field.fieldName] }}
              </el-button>
            </template>

            <template v-else>
              {{ row[field.fieldName] }}
            </template>
          </template>
        </el-table-column>

        <el-table-column label="操作" width="150" align="center">
          <template #default="scope">
            <el-button type="text"
                       @click="onOpenRunPage(scope.row)">运行
            </el-button>

            <el-button type="text"
                       @click="onOpenSaveOrUpdate('update', scope.row)">修改
            </el-button>
            <el-button type="text" @click="deleted(scope.row)">
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <pagination
          v-show="total>0"
          :total="total"
          :page="listQuery.page"
          :limit="listQuery.pageSize"
          @pagination="getList"/>
    </el-card>
    <save-or-update ref="saveOrUpdateRef" @getList="getList"/>
    <!--    运行用例-->
    <el-dialog
        draggable
        v-model="showRunPage"
        width="600px"
        top="8vh"
        title="运行用例"
        :close-on-click-modal="false">
      <el-form
          :model="runForm"
          label-width="70px"

      >
        <el-form-item label="运行模式" prop="run_mode">
          <el-select v-model="runForm.run_mode" placeholder="选择运行模式" filterable style="width:100%">
            <el-option :value="1" label="同步运行(同步执行,等待执行结果)"></el-option>
            <el-option :value="2" label="异步运行(异步执行用例,后台运行,执行结束后报告列表查看)"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="运行环境" prop="base_url">
          <el-select v-model="runForm.base_url" placeholder="选择环境" filterable style="width:100%">
            <el-option :value="''" label="自带环境">自带环境</el-option>
            <el-option
                v-for="item in envList"
                :key="item.id"
                :label="item.name"
                :value="item.url">
              <span style="float: left">{{ item.name }}</span>
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
            <span class="dialog-footer">
              <el-button @click="showRunPage = !showRunPage">取消</el-button>
              <el-button type="primary" :loading="runCaseLoading" @click="runTestCase">运行</el-button>
            </span>
      </template>
    </el-dialog>

    <el-dialog
        draggable
        v-model="showTestReportDialog"
        width="80%"
        top="8vh"
        destroy-on-close
        :close-on-click-modal="false">
      <test-report :reportBody="reportBody"/>
    </el-dialog>

    <!--    postman 导入 import-->
    <el-dialog
        draggable
        v-model="showImportPage"
        width="600px"
        top="8vh"
        title="导入"
        :close-on-click-modal="false">


      <el-form
          :rules="importRules"
          ref="importFormRef"
          :model="importForm"
          label-width="80px"
      >

        <el-form-item label="所属项目" prop="project_id">
          <el-select
              v-model="importForm.project_id" placeholder="选择归属项目"
              clearable
              filterable
              style="width: 100%;" @change="selectProject">
            <el-option
                v-for="project in projectList"
                :key="project.id"
                :label="project.name"
                :value="project.id">
              {{ project.name }}
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="所属模块" prop="module_id">
          <el-select
              v-model="importForm.module_id"
              placeholder="选择模块"
              filterable style="width: 100%;"
          >
            <el-option
                v-for="modules in moduleList"
                :key="modules.id"
                :label="modules.name"
                :value="modules.id">
              {{ modules.name }}
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="上传文件">
          <div class="file-input-container">
            <div class="file-input">
              <input size="small" type="file" id="selectFile" @change="fileChange($event)"
                     class="file-input__native">

              <el-button v-if="!importForm.file_info.raw" type="info" size="small" @click="selectFile()">选择文件
              </el-button>
              <div v-else class="file-input__name">
                <div class="file-input__name__title" :title="importForm.file_info.name">{{ importForm.file_info.name }}
                </div>
                <el-button class="file-input__name__delete-icon" size="small" type="text"
                           @click="deletedFile">
                  <el-icon>
                    <ele-Close/>
                  </el-icon>
                </el-button>
              </div>
            </div>
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="showImportPage = false">取消</el-button>
          <el-button type="primary" @click="submitUpload" :loading="importButtonStart">导入</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts">
import {defineComponent, onMounted, reactive, ref, toRefs} from 'vue';
import {ElMessage, ElMessageBox} from 'element-plus';
import saveOrUpdate from '/@/views/api/module/components/saveOrUpdate.vue';
import Pagination from '/@/components/Pagination/index.vue';
import {useTestCaseApi} from "/@/api/useAutoApi/testcase";
import {useRouter} from "vue-router";
import {useEnvApi} from "/@/api/useAutoApi/env";
import TestReport from '/@/views/api/Report/components/report.vue';
import {useModuleApi} from "/@/api/useAutoApi/module";
import {useProjectApi} from "/@/api/useAutoApi/project";


export default defineComponent({
  name: 'apiTestCase',
  components: {saveOrUpdate, Pagination, TestReport},
  setup() {
    const saveOrUpdateRef = ref();
    const importFormRef = ref();
    const uploadRef = ref();
    const router = useRouter();
    const state = reactive({
      fieldData: [
        {fieldName: 'id', label: 'ID', width: '80', align: 'center', show: true},
        {fieldName: 'name', label: '用例名', width: '', align: 'center', show: true},
        {fieldName: 'project_name', label: '所属项目', width: '', align: 'center', show: true},
        {fieldName: 'module_name', label: '所属模块', width: '', align: 'center', show: true},
        {fieldName: 'config_id', label: '关联配置', width: '', align: 'center', show: true},
        {fieldName: 'updation_date', label: '更新时间', width: '150', align: 'center', show: true},
        {fieldName: 'updated_by_name', label: '更新人', width: '', align: 'center', show: true},
        {fieldName: 'creation_date', label: '创建时间', width: '150', align: 'center', show: true},
        {fieldName: 'created_by_name', label: '创建人', width: '', align: 'center', show: true},
      ],
      listData: [],
      tableLoading: false,
      total: 0,
      listQuery: {
        page: 1,
        pageSize: 20,
        name: '',
      },
      // run test case
      showRunPage: false,
      runCaseLoading: false,
      envList: [],
      runForm: {
        id: null,
        base_url: '',
        run_type: 'case',
        run_mode: 1,
      },
      // report
      reportBody: {},
      showTestReportDialog: false,

      //project
      projectList: [],
      projectQuery: {
        page: 1,
        pageSize: 1000,
      },
      //module
      moduleList: [],
      moduleQuery: {
        page: 1,
        pageSize: 1000,
        project_id: null
      },

      // import
      showImportPage: false,
      importButtonStart: false,
      importForm: {
        file_info: {
          raw: '',
          name: '',
        },
        project_id: '',
        module_id: '',
      },
      importRules: {
        project_id: [{required: true, message: '请选择项目', trigger: 'blur'}],
        module_id: [{required: true, message: '请选择模块', trigger: 'blur'}],
      },
    });
    // 初始化表格数据
    const getList = () => {
      state.tableLoading = true
      useTestCaseApi().getList(state.listQuery)
          .then(res => {
            state.listData = res.data.rows
            state.total = res.data.rowTotal
            state.tableLoading = false
          })
    };

    // 新增或修改
    const onOpenSaveOrUpdate = (editType: string, row: any | null) => {
      let query: any = {}
      query.editType = editType
      if (row) query.id = row.id
      router.push({name: 'saveOrUpdateTestCase', query: query})
    };

    // 删除
    const deleted = (row: any) => {
      ElMessageBox.confirm('是否删除该条数据, 是否继续?', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning',
      })
          .then(() => {
            useTestCaseApi().deleted({id: row.id})
                .then(() => {
                  ElMessage.success('删除成功');
                  getList()
                })
          })
          .catch(() => {
          });
    };

    // 打开运行页面
    const onOpenRunPage = (row: any) => {
      state.showRunPage = true;
      state.runForm.id = row.id;
      getEnvList();
    };
    // 获取环境信息
    const getEnvList = () => {
      useEnvApi().getList({page: 1, pageSize: 1000})  // 请求数据写死，后面优化
          .then(res => {
            state.envList = res.data.rows
          })
    }
    // 运行测试用例
    const runTestCase = () => {
      state.runCaseLoading = !state.runCaseLoading;
      useTestCaseApi().runTestCaseNew(state.runForm)
          .then((res: any) => {
            if (state.runForm.run_mode === 1) {
              ElMessage.success('运行成功');
              state.showTestReportDialog = !state.showTestReportDialog;
              state.reportBody = res.data
              state.runCaseLoading = !state.runCaseLoading;
            } else {
              ElMessage.success(res.msg);
              state.runCaseLoading = !state.runCaseLoading;
            }

          })
          .catch((err: any) => {
            ElMessage.error(err.message);
            state.runCaseLoading = !state.runCaseLoading;
          })
    }

    // import
    //  ----------------project start-------------------------------------

    // show import page
    const openImportPage = () => {
      state.showImportPage = !state.showImportPage
      state.importForm.project_id = ''
      state.importForm.module_id = ''
      state.importForm.file_info.raw = ''
      state.importForm.file_info.name = ''
      getProjectList()
    }
    // 获取项目列表
    const getProjectList = () => {
      useProjectApi().getList(state.projectQuery) // 请求数据写死，后面优化
          .then(res => {
            state.projectList = res.data.rows
          })
    }
    // 选择项目
    const selectProject = (project_id: any) => {
      state.moduleQuery.project_id = project_id
      state.moduleList = []
      state.importForm.module_id = ''
      getModuleList()
    }

    //  ----------------module start-------------------------------------
    // 获取模块列表
    const getModuleList = () => {
      useModuleApi().getList(state.moduleQuery) // 请求数据写死，后面优化
          .then(res => {
            state.moduleList = res.data.rows
          })
    }

    // 文件上传
    const submitUpload = () => {
      state.importButtonStart = true
      importFormRef.value.validate(async (vai: any) => {
        if (vai) {
          if (!state.importForm.file_info.raw) {
            ElMessage.info('请选择上传文件！')
            state.importButtonStart = false
            return
          }
          try {
            let formData = new FormData()
            formData.append('file', state.importForm.file_info.raw)
            formData.append('project_id', state.importForm.project_id)
            formData.append('module_id', state.importForm.module_id)
            let res: any = await useTestCaseApi().postman2case(formData)
            ElMessage.success(`成功导入${res.data}条用例！`)
            state.importButtonStart = false
            state.showImportPage = false
            getList()

            // useTestCaseApi().postman2case(formData)
            //     .then((res: any) => {
            //       ElMessage.success(`成功导入${res.data}条用例！`)
            //       state.importButtonStart = false
            //       state.showImportPage = false
            //       getList()
            //     })
          } catch (e) {
            state.importButtonStart = false
          }
        } else {
          state.importButtonStart = false
        }
      })
    }

    // 选择文件时触发，上传文件，回写地址
    const fileChange = (e: any) => {
      let file: any = e.target.files[0]
      state.importForm.file_info.raw = file
      state.importForm.file_info.name = file.name
    }
    // 删除文件处理
    const deletedFile = () => {
      let fileRef: any = document.getElementById('selectFile')
      if (fileRef) fileRef.value = ''
      state.importForm.file_info.raw = ''
      state.importForm.file_info.name = ''
    }

    // 选择文件
    const selectFile = () => {
      let fileRef = document.getElementById('selectFile')
      if (fileRef) fileRef.click()
    }

    // 页面加载时
    onMounted(() => {
      getList();
    });
    return {
      getList,
      saveOrUpdateRef,
      importFormRef,
      uploadRef,
      getEnvList,
      onOpenRunPage,
      runTestCase,
      onOpenSaveOrUpdate,
      deleted,
      router,
      fileChange,
      deletedFile,
      selectFile,
      openImportPage,
      getProjectList,
      selectProject,
      getModuleList,
      submitUpload,
      ...toRefs(state),
    };
  },
});
</script>

<style lang="scss" scoped>
.file-input-container {
  display: inline-block;
  max-width: 100%;

  .file-input {
    display: flex;
    align-items: center;
    padding: var(--spacing-xs);

    .file-input__native {
      opacity: 0;
      position: absolute;
      width: 0;
      height: 0;
      pointer-events: none;
    }

    .file-input__fake {
      position: relative;
      height: 20px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      cursor: pointer;
      background-color: #F2F2F2;
      min-width: 100px;
      color: #6B6B6B;
      font-weight: 600;

      &:hover {
        color: #212121;
        background-color: #E6E6E;
      }
    }

    .btn {
      box-sizing: border-box;
      border-radius: 4px;
    }

    .file-input__name {
      box-sizing: border-box;
      display: flex;
      min-width: 0;
      height: 24px;
      align-items: center;
      border-radius: 4px;
      border: 1px solid #E6E6E6;
      font-size: 12px;
      font-family: "Inter", system-ui, -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica, Arial, sans-serif;
      color: #212121;
      background-color: transparent;
      padding: 4px 2px;

      .file-input__name__title {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      .file-input__name__delete-icon {
        display: flex;
        align-items: center;
        margin-left: 8px;
        padding-right: 2px;
        cursor: pointer;
        color: #212121;
      }
    }
  }
}
</style>