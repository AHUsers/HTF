<template>
  <div>
    <el-card shadow="hover">
      <div class="mb15">
        <el-input v-model="listQuery.name" placeholder="请输入套件名称" style="max-width: 180px"></el-input>
        <el-button type="primary" class="ml10" @click="search">
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
      <el-table
          border
          v-loading="tableLoading"
          :data="listData"
          stripe
          highlight-current-row
          style="width: 100%">

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
          <template #default="{row}">
            <el-button size="small" type="text" @click="runSuitePage(row)">
              运行
            </el-button>
            <el-button size="small"
                       type="text"
                       @click="onOpenSaveOrUpdate('update', row)">修改
            </el-button>
            <el-button size="small" type="text" @click="deleted(row)">
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

    <!--    运行   -->
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
        <el-form-item label="运行环境" prop="belong_project_id">
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
              <el-button type="primary" :loading="runCaseLoading" @click="runTestSuite">运行</el-button>
            </span>
      </template>
    </el-dialog>

  </div>
</template>

<script lang="ts">
import {defineComponent, onMounted, reactive, ref, toRefs} from 'vue';
import {ElMessage, ElMessageBox} from 'element-plus';
import Pagination from '/@/components/Pagination/index.vue';
import {useTestSuiteApi} from "/@/api/useAutoApi/suite";
import {useTestCaseApi} from "/@/api/useAutoApi/testcase";
import {useRouter} from 'vue-router'
import {useEnvApi} from "/@/api/useAutoApi/env";

// 定义接口来定义对象的类型
// interface TableData {
//   roleName: string;
//   roleSign: string;
//   describe: string;
//   sort: number;
//   status: boolean;
//   createTime: string;
// }


export default defineComponent({
  name: 'apiCaseSuite',
  components: {Pagination},
  setup() {
    const saveOrUpdateRef = ref();
    const router = useRouter();
    const state = reactive({
      fieldData: [
        {fieldName: 'id', label: 'ID', width: '55', align: 'center', show: true},
        {fieldName: 'name', label: '套件名称', width: '', align: 'center', show: true},
        {fieldName: 'project_name', label: '所属项目', width: '', align: 'center', show: true},
        {fieldName: 'updation_date', label: '更新时间', width: '150', align: 'center', show: true},
        {fieldName: 'updated_by_name', label: '更新人', width: '', align: 'center', show: true},
        {fieldName: 'creation_date', label: '创建时间', width: '150', align: 'center', show: true},
        {fieldName: 'created_by_name', label: '创建人', width: '', align: 'center', show: true},
      ],
      // list
      listData: [],
      tableLoading: false,
      total: 0,
      listQuery: {
        page: 1,
        pageSize: 20,
        name: '',
      },
      // run
      showRunPage: false,
      runCaseLoading: false,
      runForm: {
        id: null,
        base_url: '',
        run_type: 'suite',
      },
      // env
      envList: [],
    });
    // 初始化表格数据
    const getList = () => {
      state.tableLoading = true
      useTestSuiteApi().getList(state.listQuery)
          .then(res => {
            state.listData = res.data.rows
            state.total = res.data.rowTotal
            state.tableLoading = false
          })
    };

     // 查询
    const search = () => {
      state.listQuery.page = 1
      getList()
    }

    // 新增或修改角色
    const onOpenSaveOrUpdate = (editType: string, row: any) => {
      let query: any = {}
      query.editType = editType
      if (row) query.id = row.id
      router.push({name: 'saveOrUpdateSuite', query: query})
    };

    // 删除角色
    const deleted = (row: any) => {
      ElMessageBox.confirm('是否删除该条数据, 是否继续?', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning',
      })
          .then(() => {
            useTestSuiteApi().deleted({id: row.id})
                .then(() => {
                  ElMessage.success('删除成功');
                  getList()
                })
          })
          .catch(() => {
          });
    };

    // 获取环境信息
    const getEnvList = () => {
      useEnvApi().getList({page: 1, pageSize: 1000})  // 请求数据写死，后面优化
          .then(res => {
            state.envList = res.data.rows
          })
    }

    //runSuitePage
    const runSuitePage = (row) => {
      state.runForm.id = row.id
      state.showRunPage = !state.showRunPage
      getEnvList()
    }

    //runSuitePage
    const runTestSuite = () => {
      useTestCaseApi().runTestCaseNew(state.runForm).then(res => {
        console.log(res)
      })
    }

    // 页面加载时
    onMounted(() => {
      getList();
    });
    return {
      getList,
      router,
      saveOrUpdateRef,
      onOpenSaveOrUpdate,
      deleted,
      getEnvList,
      runSuitePage,
      runTestSuite,
      ...toRefs(state),
    };
  },
});
</script>
