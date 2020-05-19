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
          <el-form-item label="员工编号">
            <el-input v-model="form.eid" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="工号" prop="username">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="教师姓名" prop="name">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="培养进修、交流类型">
            <el-input v-model="form.type" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="培养进修、交流地点">
            <el-input v-model="form.place" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="接收培训进修、交流单位">
            <el-input v-model="form.unit" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="培训内容">
            <el-input v-model="form.trainContent" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="开始时间">
            <el-date-picker v-model="form.startTime" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="结束时间">
            <el-date-picker v-model="form.finishTime" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="备注">
            <el-input v-model="form.remark" :rows="3" type="textarea" style="width: 370px;" />
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
        <el-table-column v-if="columns.visible('eid')" prop="eid" label="员工编号" />
        <el-table-column v-if="columns.visible('username')" prop="username" label="工号" />
        <el-table-column v-if="columns.visible('name')" prop="name" label="教师姓名" />
        <el-table-column v-if="columns.visible('type')" prop="type" label="培养进修、交流类型">
          <template slot-scope="scope">
            {{ dict.label.train_status[scope.row.type] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('place')" prop="place" label="培养进修、交流地点" />
        <el-table-column v-if="columns.visible('unit')" prop="unit" label="接收培训进修、交流单位" />
        <el-table-column v-if="columns.visible('trainContent')" prop="trainContent" label="培训内容" />
        <el-table-column v-if="columns.visible('startTime')" prop="startTime" label="开始时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.startTime, '{y}-{m}-{d}') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('finishTime')" prop="finishTime" label="结束时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.finishTime, '{y}-{m}-{d}') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('remark')" prop="remark" label="备注" />
        <el-table-column v-permission="['admin','employeetrain:edit','employeetrain:del']" label="操作" width="150px" align="center">
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
import crudEmployeetrain from '@/api/employeetrain'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

// crud交由presenter持有
const defaultCrud = CRUD({ title: 'employeetrain', url: 'api/employeetrain', sort: 'id,desc', crudMethod: { ...crudEmployeetrain }})
const defaultForm = { id: null, eid: null, remark: null, trainContent: null, username: null, name: null, place: null, unit: null, startTime: null, finishTime: null, type: null }
export default {
  name: 'Employeetrain',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
  dicts: ['train_status'],
  data() {
    return {
      permission: {
        add: ['admin', 'employeetrain:add'],
        edit: ['admin', 'employeetrain:edit'],
        del: ['admin', 'employeetrain:del']
      },
      rules: {
        username: [
          { required: true, message: '工号不能为空', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '教师姓名不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'eid', display_name: '员工编号' },
        { key: 'username', display_name: '工号' },
        { key: 'name', display_name: '教师姓名' }
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
