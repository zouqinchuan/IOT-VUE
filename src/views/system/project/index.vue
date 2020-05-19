<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.value" clearable placeholder="输入搜索内容" style="width: 200px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <el-select v-model="query.type" clearable placeholder="类型" class="filter-item" style="width: 130px">
          <el-option v-for="item in queryTypeOptions" :key="item.key" :label="item.display_name" :value="item.key" />
        </el-select>
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="工号" prop="username">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="教师姓名">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="项目名称">
            <el-input v-model="form.project" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="项目性质">
            <el-input v-model="form.nature" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="单位">
            <el-input v-model="form.unit" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="项目经费">
            <el-input v-model="form.expenditure" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="立项时间">
            <el-date-picker v-model="form.projectTime" type="date" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="立项编号">
            <el-input v-model="form.projectId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="结题验收或鉴定时间">
            <el-date-picker v-model="form.finishTime" type="date" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column v-if="columns.visible('username')" prop="username" label="工号" />
        <el-table-column v-if="columns.visible('name')" prop="name" label="教师姓名" />
        <el-table-column v-if="columns.visible('project')" prop="project" label="项目名称" />
        <el-table-column v-if="columns.visible('nature')" prop="nature" label="项目性质">
          <template slot-scope="scope">
            {{ dict.label.project_status[scope.row.nature] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('unit')" prop="unit" label="单位" />
        <el-table-column v-if="columns.visible('expenditure')" prop="expenditure" label="项目经费" />
        <el-table-column v-if="columns.visible('projectTime')" prop="projectTime" label="立项时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.projectTime, '{y}-{m}-{d}') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('projectId')" prop="projectId" label="立项编号" />
        <el-table-column v-if="columns.visible('finishTime')" prop="finishTime" label="结题验收或鉴定时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.finishTime, '{y}-{m}-{d}') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','project:edit','project:del']" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudProject from '@/api/project'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

// crud交由presenter持有
const defaultCrud = CRUD({ title: 'project', url: 'api/project', sort: 'id,desc', crudMethod: { ...crudProject }})
const defaultForm = { id: null, username: null, name: null, project: null, nature: null, unit: null, expenditure: null, projectTime: null, projectId: null, finishTime: null }
export default {
  name: 'Project',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
  dicts: ['project_status'],
  data() {
    return {
      permission: {
        add: ['admin', 'project:add'],
        edit: ['admin', 'project:edit'],
        del: ['admin', 'project:del']
      },
      rules: {
        username: [
          { required: true, message: '工号不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'username', display_name: '工号' },
        { key: 'name', display_name: '教师姓名' },
        { key: 'project', display_name: '项目名称' },
        { key: 'unit', display_name: '单位' },
        { key: 'projectId', display_name: '立项编号' }
      ]
    }
  },
  methods: {
    // 获取数据前设置好接口地址
    [CRUD.HOOK.beforeRefresh]() {
      const query = this.query
      if (query.type && query.value) {
        this.crud.params[query.type] = query.value
      }
      return true
    }
  }
}
</script>

<style scoped>

</style>
