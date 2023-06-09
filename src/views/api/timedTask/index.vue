<template>
  <div>
    <el-card shadow="hover">
      <div class="mb15">
        <el-input v-model="listQuery.name" placeholder="请输入名称" style="max-width: 180px"></el-input>
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
      <el-table
          border
          v-loading="tableLoading"
          :data="listData"
          style="width: 100%">
        <el-table-column label="序号" width="50px" align="center">
          <template #default="scope">
            {{ scope.$index + (listQuery.page - 1) * listQuery.pageSize + 1 }}
          </template>
        </el-table-column>

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

            <template v-if="field.fieldName === 'enabled'">
              <div style="align-items: center; display: flex; justify-content: center">
                <div class="request-editor-tabs-badge"
                     :class="[row['enabled'] === 1? 'start': 'stop']">
                </div>
                <span>{{ formatLookup(field.lookupCode, row['enabled']) }}</span>
              </div>

            </template>

            <template v-else>
              <span>{{
                  field.lookupCode ? formatLookup(field.lookupCode, row[field.fieldName]) : row[field.fieldName]
                }}</span>
            </template>

          </template>
        </el-table-column>

        <el-table-column label="操作" width="150">
          <template #default="{row}">

            <el-button size="small"
                       type="text"
                       @click="taskSwitch(row)">
              {{ row.enabled ? '停止' : '启动' }}
            </el-button>
            <el-button size="small" type="text"
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
    <save-or-update ref="saveOrUpdateRef" @getList="getList"/>
  </div>
</template>

<script lang="ts">
import {defineComponent, onMounted, reactive, ref, toRefs} from 'vue';
import {ElMessage, ElMessageBox} from 'element-plus';
import saveOrUpdate from '/@/views/api/timedTask/components/saveOrUpdate.vue';
import Pagination from '/@/components/Pagination/index.vue';
import {useTimedTasksApi} from "/@/api/useAutoApi/timedTasks";
import {formatLookup} from "/@/utils/lookup";

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
  name: 'apiTimedTask',
  components: {saveOrUpdate, Pagination},
  setup() {
    const saveOrUpdateRef = ref();
    const state = reactive({
      fieldData: [
        {fieldName: 'name', label: '任务名称', width: '', align: 'center', show: true},
        {fieldName: 'project_name', label: '所属项目', width: '', align: 'center', show: true},
        {fieldName: 'crontab_str', label: '执行时间', width: '', align: 'center', show: true},
        {
          fieldName: 'run_type',
          label: '任务类型',
          width: '',
          align: 'center',
          show: true,
          lookupCode: 'api_report_run_type'
        },
        {
          fieldName: 'enabled',
          label: '任务状态',
          width: '',
          align: 'center',
          show: true,
          lookupCode: 'api_timed_task_status'
        },
        {fieldName: 'description', label: '备注', width: '', align: 'center', show: true},
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
    });
    // 初始化表格数据
    const getList = () => {
      state.tableLoading = true
      useTimedTasksApi().getList(state.listQuery)
          .then(res => {
            state.listData = res.data.rows
            state.total = res.data.rowTotal
            state.tableLoading = false
          })
    };

    // 新增或修改角色
    const onOpenSaveOrUpdate = (editType: string, row: any) => {
      saveOrUpdateRef.value.openDialog(editType, row);
    };

    // 新增或修改角色
    const taskSwitch = (row: any) => {
      ElMessageBox.confirm(`${row.enabled ? '停止' : '启动' }当前任务, 是否继续?`, '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        useTimedTasksApi().taskSwitch({id: row.id})
            .then(() => {
              ElMessage.success('操作成功！');
              getList()
            })
      })

    };

    // 删除角色
    const deleted = (row: any) => {
      ElMessageBox.confirm('是否删除该条数据, 是否继续?', '提示', {
        confirmButtonText: '确认',
        cancelButtonText: '取消',
        type: 'warning',
      })
          .then(() => {
            useTimedTasksApi().deleted({id: row.id})
                .then(() => {
                  ElMessage.success('删除成功');
                  getList()
                })
          })
          .catch(() => {
          });
    };
    // 页面加载时
    onMounted(() => {
      getList();
    });
    return {
      getList,
      saveOrUpdateRef,
      onOpenSaveOrUpdate,
      deleted,
      taskSwitch,
      formatLookup,
      ...toRefs(state),
    };
  },
});
</script>

<style>

.stop {
  background-color: #c1bfc7;
}

.start {
  background-color: #0cbb52;
}

.request-editor-tabs-badge {
  display: inline-flex;
  width: 8px;
  height: 8px;
  margin-right: 5px;
  border-radius: 8px;
}
</style>