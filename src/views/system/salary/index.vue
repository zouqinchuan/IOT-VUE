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
            <el-input v-model="form.jbbasic" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="奖金">
            <el-input v-model="form.jjbonus" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="午餐补助">
            <el-input v-model="form.bzlunch" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="交通补助">
            <el-input v-model="form.bztraffic" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="养老金">
            <el-input v-model="form.ylpension" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="启用时间">
            <el-date-picker v-model="form.qdcreateDate" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="医疗保险">
            <el-input v-model="form.ylmedical" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="公积金">
            <el-input v-model="form.gjjaccumulation" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="应发工资">
            <el-input v-model="form.yfall" style="width: 370px;" />
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
        <el-table-column v-if="columns.visible('jbbasic')" prop="jbbasic" label="基本工资" />
        <el-table-column v-if="columns.visible('jjbonus')" prop="jjbonus" label="奖金" />
        <el-table-column v-if="columns.visible('bzlunch')" prop="bzlunch" label="午餐补助" />
        <el-table-column v-if="columns.visible('bztraffic')" prop="bztraffic" label="交通补助" />
        <el-table-column v-if="columns.visible('ylpension')" prop="ylpension" label="养老金" />
        <el-table-column v-if="columns.visible('qdcreateDate')" prop="qdcreateDate" label="启用时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.qdcreateDate) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('ylmedical')" prop="ylmedical" label="医疗保险" />
        <el-table-column v-if="columns.visible('gjjaccumulation')" prop="gjjaccumulation" label="公积金" />
        <el-table-column v-if="columns.visible('yfall')" prop="yfall" label="应发工资" />
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
const defaultCrud = CRUD({ title: '薪酬', url: 'api/salary', sort: 'id,desc', crudMethod: { ...crudSalary }})
const defaultForm = { id: null, username: null, jbbasic: null, jjbonus: null, bzlunch: null, bztraffic: null, ylpension: null, qscreateDate: null, ylmedical: null, gjjaccumulation: null, yfall: null, name: null }
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
        { key: 'jbbasic', display_name: '基本工资' },
        { key: 'jjbonus', display_name: '奖金' },
        { key: 'bzlunch', display_name: '午餐补助' },
        { key: 'bztraffic', display_name: '交通补助' },
        { key: 'ylpension', display_name: '养老金' },
        { key: 'ylmedical', display_name: '医疗保险' },
        { key: 'gjjaccumulation', display_name: '公积金' },
        { key: 'yfall', display_name: '应发工资' },
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
