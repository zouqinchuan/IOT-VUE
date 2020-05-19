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
          <el-form-item label="时间">
            <el-date-picker v-model="form.time" type="date" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="工号" prop="username">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="教师姓名" prop="name">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="作者类型">
            <el-input v-model="form.type" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="论文名称">
            <el-input v-model="form.paper" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="发表期刊">
            <el-input v-model="form.periodical" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="收录期刊">
            <el-input v-model="form.situation" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="发表时间">
            <el-date-picker v-model="form.postedTime" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="他引次数">
            <el-input v-model="form.leadTimes" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="是否与行业联合发表">
            <el-radio v-for="item in dict.industry_status" :key="item.id" v-model="form.industry" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="是否与地方联合发表">
            <el-radio v-for="item in dict.place_status" :key="item.id" v-model="form.place" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="是否与国际联合发表">
            <el-radio v-for="item in dict.international_status" :key="item.id" v-model="form.international" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="是否是跨学科论文">
            <el-radio v-for="item in dict.interdisciplinary_status" :key="item.id" v-model="form.interdisciplinary" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" border :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column v-if="columns.visible('time')" prop="time" label="时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.time, '{y}年') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('username')" prop="username" label="工号" />
        <el-table-column v-if="columns.visible('name')" prop="name" label="教师姓名" />
        <el-table-column v-if="columns.visible('type')" prop="type" label="作者类型" />
        <el-table-column v-if="columns.visible('paper')" prop="paper" label="论文名称" />
        <el-table-column v-if="columns.visible('periodical')" prop="periodical" label="发表期刊" />
        <el-table-column v-if="columns.visible('situation')" prop="situation" label="收录期刊" />
        <el-table-column v-if="columns.visible('postedTime')" prop="postedTime" label="发表时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.postedTime, '{y}-{m}') }}</span>
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('leadTimes')" prop="leadTimes" label="他引次数" />
        <el-table-column v-if="columns.visible('industry')" prop="industry" label="是否与行业联合发表">
          <template slot-scope="scope">
            {{ dict.label.industry_status[scope.row.industry] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('place')" prop="place" label="是否与地方联合发表">
          <template slot-scope="scope">
            {{ dict.label.place_status[scope.row.place] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('international')" prop="international" label="是否与国际联合发表">
          <template slot-scope="scope">
            {{ dict.label.international_status[scope.row.international] }}
          </template>
        </el-table-column>
        <el-table-column v-if="columns.visible('interdisciplinary')" prop="interdisciplinary" label="是否是跨学科论文">
          <template slot-scope="scope">
            {{ dict.label.interdisciplinary_status[scope.row.interdisciplinary] }}
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','paper:edit','paper:del']" label="操作" width="150px" align="center">
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
import crudPaper from '@/api/paper'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

// crud交由presenter持有
const defaultCrud = CRUD({ title: 'paper', url: 'api/paper', sort: 'id,desc', crudMethod: { ...crudPaper }})
const defaultForm = { id: null, time: null, username: null, name: null, type: null, paper: null, periodical: null, situation: null, postedTime: null, leadTimes: null, industry: null, place: null, international: null, interdisciplinary: null }
export default {
  name: 'Paper',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
  dicts: ['industry_status', 'place_status', 'international_status', 'interdisciplinary_status'],
  data() {
    return {
      permission: {
        add: ['admin', 'paper:add'],
        edit: ['admin', 'paper:edit'],
        del: ['admin', 'paper:del']
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
        { key: 'username', display_name: '工号' },
        { key: 'name', display_name: '教师姓名' },
        { key: 'paper', display_name: '论文名称' }
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
