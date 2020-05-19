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
          <el-form-item label="基本工资">
            <el-input v-model="form.basic" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="奖金">
            <el-input v-model="form.bonus" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="午餐补助">
            <el-input v-model="form.lunch" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交通补助">
            <el-input v-model="form.traffic" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="养老金">
            <el-input v-model="form.pension" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="启用时间">
            <el-date-picker v-model="form.createDate" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="医疗保险">
            <el-input v-model="form.medical" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="公积金">
            <el-input v-model="form.accumulation" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="应发工资">
            <el-input v-model="form.all" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="姓名">
            <el-input v-model="form.name" style="width: 370px;" />
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
        <el-table-column v-if="columns.visible('basic')" prop="basic" label="基本工资" />
        <el-table-column v-if="columns.visible('bonus')" prop="bonus" label="奖金" />
        <el-table-column v-if="columns.visible('lunch')" prop="lunch" label="午餐补助" />
        <el-table-column v-if="columns.visible('traffic')" prop="traffic" label="交通补助" />
        <el-table-column v-if="columns.visible('pension')" prop="pension" label="养老金" />
        <el-table-column v-if="columns.visible('createDate')" prop="createDate" label="启用时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createDate) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('medical')" prop="medical" label="医疗保险" />
        <el-table-column v-if="columns.visible('accumulation')" prop="accumulation" label="公积金" />
        <el-table-column v-if="columns.visible('all')" prop="all" label="应发工资" />
        <el-table-column v-if="columns.visible('name')" prop="name" label="姓名" />
        <el-table-column v-permission="['admin','salary:edit','salary:del']" label="操作" width="150px" align="center">
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
import crudSalary from '@/api/salary'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

// crud交由presenter持有
const defaultCrud = CRUD({ title: 'salary', url: 'api/salary', sort: 'id,desc', crudMethod: { ...crudSalary }})
const defaultForm = { id: null, username: null, basic: null, bonus: null, lunch: null, traffic: null, pension: null, createDate: null, medical: null, accumulation: null, all: null, name: null }
export default {
  name: 'Salary',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
  data() {
    return {
      permission: {
        add: ['admin', 'salary:add'],
        edit: ['admin', 'salary:edit'],
        del: ['admin', 'salary:del']
      },
      rules: {
        username: [
          { required: true, message: '工号不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'username', display_name: '工号' },
        { key: 'basic', display_name: '基本工资' },
        { key: 'bonus', display_name: '奖金' },
        { key: 'lunch', display_name: '午餐补助' },
        { key: 'traffic', display_name: '交通补助' },
        { key: 'pension', display_name: '养老金' },
        { key: 'medical', display_name: '医疗保险' },
        { key: 'accumulation', display_name: '公积金' },
        { key: 'all', display_name: '应发工资' },
        { key: 'name', display_name: '姓名' }
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
