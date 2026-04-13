---
name: element-admin-dev
description: |
  从零开发 Element Plus 管理页面。根据业务需求创建完整的增删改查页面，包括列表页、表单弹窗、详情查看。
  使用场景：
  - 用户提到"开发新页面"、"创建页面"、"新增页面"
  - 用户提到"做个管理页面"、"写个增删改查"
  - 用户提到"Element Plus 页面"、"Element 页面开发"
  - 用户提到"表单页面"、"列表页面"
  - 用户提到需要一个完整的业务功能页面
  核心能力：基于业务需求和 API 接口，创建符合项目规范的 Element Plus 管理页面，包含搜索、表格、分页、新增、编辑、删除、查看功能。
triggers:
  - 开发新页面
  - 创建页面
  - 新增页面
  - 管理页面
  - 增删改查
  - Element Plus 页面
  - Element 页面
  - 表单页面
  - 列表页面
  - 业务页面
  - 功能页面
---

# Element Plus 管理页面开发技能

## 技能概述

本技能用于从零开发 Element Plus 管理页面，遵循项目既有的编码规范和 UI 规范，确保页面与项目风格一致。

---

## 页面类型

管理页面通常包含以下类型，根据业务需求选择：

1. **列表页** - 表格展示 + 搜索 + 分页 + 操作按钮
2. **表单弹窗** - 新增/编辑/查看的表单弹窗
3. **详情页** - 纯展示的详情查看页面（通常与表单弹窗复用）

---

## 页面结构模板

### 标准列表页结构

```vue
<template>
  <div class="page-container">
    <!-- 搜索区域 -->
    <SearchForm
      v-model="query"
      :fields="searchFields"
      @search="searchChange"
      @reset="resetChange"
      class="search-area"
    />

    <!-- 工具栏 -->
    <div class="toolbar">
      <el-button type="primary" @click="handleAdd" v-if="permissionList.addBtn">
        <el-icon><Plus /></el-icon>新增
      </el-button>
      <el-button type="danger" @click="batchDel" v-if="permissionList.delBtn" :disabled="!selectData.length">
        <el-icon><Delete /></el-icon>批量删除
      </el-button>
      <el-button @click="handleExport" v-if="permissionList.exportBtn">
        <el-icon><Download /></el-icon>导出
      </el-button>
    </div>

    <!-- 表格区域 -->
    <div class="table-wrapper">
      <el-table
        ref="tableRef"
        v-loading="loading"
        :data="tableData"
        @selection-change="handleSelectionChange"
        height="calc(100% - 240px)"
        border
        class="table-area"
      >
        <el-table-column v-if="hasSelection" type="selection" width="55" fixed="left" />
        <el-table-column v-if="hasIndex" type="index" label="序号" width="80" fixed="left" />
        <el-table-column
          v-for="col in tableColumns"
          :key="col.prop"
          :label="col.label"
          :prop="col.prop"
          :width="col.width"
          :min-width="col.minWidth"
          :align="col.align || 'center'"
        >
          <template #default="{ row }">
            <!-- 状态显示 -->
            <el-tag v-if="col.prop === 'status'" :type="getStatusType(row.status)">
              {{ row.statusName }}
            </el-tag>
            <!-- 自定义插槽 -->
            <slot v-else :name="col.prop" :row="row">{{ row[col.prop] }}</slot>
          </template>
        </el-table-column>
        <!-- 操作列 -->
        <el-table-column label="操作" :width="menuWidth" fixed="right" align="center">
          <template #default="{ row }">
            <el-button type="primary" link size="small" @click="handleView(row)">
              <el-icon><View /></el-icon>查看
            </el-button>
            <el-button type="warning" link size="small" @click="handleEdit(row)" v-if="permissionList.editBtn">
              <el-icon><Edit /></el-icon>编辑
            </el-button>
            <el-button type="danger" link size="small" @click="handleDelete(row)" v-if="permissionList.delBtn">
              <el-icon><Delete /></el-icon>删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页器 -->
      <el-pagination
        v-model:current-page="page.currentPage"
        v-model:page-size="page.pageSize"
        :total="page.total"
        :page-sizes="[10, 20, 50, 100]"
        layout="total, sizes, prev, pager, next"
        @size-change="onLoad"
        @current-change="onLoad"
      />
    </div>

    <!-- 表单弹窗 -->
    <FormDialog
      v-model="dialogVisible"
      :dialog-type="dialogType"
      :form-data="formData"
      @confirm="handleConfirm"
      @cancel="dialogVisible = false"
    />
  </div>
</template>
```

### 标准表单弹窗结构

```vue
<template>
  <el-dialog
    v-model="dialogVisible"
    :title="dialogTitle"
    width="600px"
    @close="handleClose"
  >
    <el-form ref="formRef" :model="form" :label-position="labelPosition" :label-width="labelWidth + 'px'">
      <el-row :gutter="20">
        <el-col :span="span">
          <el-form-item :label="label" :prop="prop" :rules="rules">
            <!-- 输入框 -->
            <el-input
              v-if="!type || type === 'input'"
              v-model="form[prop]"
              :placeholder="`请输入${label}`"
              :disabled="isView"
            />
            <!-- 下拉选择 -->
            <el-select
              v-else-if="type === 'select'"
              v-model="form[prop]"
              :placeholder="`请选择${label}`"
              :disabled="isView"
            >
              <el-option
                v-for="opt in options"
                :key="opt.value"
                :label="opt.label"
                :value="opt.value"
              />
            </el-select>
            <!-- 日期选择 -->
            <el-date-picker
              v-else-if="type === 'date'"
              v-model="form[prop]"
              type="datetime"
              :placeholder="`请选择${label}`"
              value-format="YYYY-MM-DD HH:mm:ss"
              format="YYYY-MM-DD HH:mm:ss"
              :disabled="isView"
            />
            <!-- 文本域 -->
            <el-input
              v-else-if="type === 'textarea'"
              v-model="form[prop]"
              type="textarea"
              :rows="3"
              :placeholder="`请输入${label}`"
              :disabled="isView"
            />
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <template #footer>
      <el-button @click="dialogVisible = false">取消</el-button>
      <el-button type="primary" @click="handleConfirm" v-if="!isView">确定</el-button>
    </template>
  </el-dialog>
</template>
```

---

## 开发步骤

### 步骤 1：分析需求

开发页面之前，必须清楚以下信息：

1. **业务场景** - 这个页面用于什么业务？
2. **API 接口** - 后端提供了哪些接口？（参考既有 API 模块）
   - `getList` - 获取列表
   - `add` - 新增
   - `update` - 编辑
   - `del` - 删除
   - `view` - 详情
3. **字段定义** - 有哪些表单项？类型、验证规则、字典数据？
4. **权限标识** - 按钮权限的 key 是什么？

### 步骤 2：确定文件结构

```
src/views/hse/views/{业务模块}/
├── {业务名称}/
│   ├── index.vue              # 列表页入口（可选）
│   ├── {业务名称}.vue          # 列表页组件
│   └── components/
│       └── FormDialog.vue     # 表单弹窗组件（可选内嵌）
```

### 步骤 3：编写列表页

#### 3.1 导入依赖

```javascript
import { ref, reactive, computed, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { View, Edit, Delete, Plus, Download, Upload } from '@element-plus/icons-vue'
import SearchForm from '@/components/SearchForm/index.vue'
import FormDialog from './components/FormDialog.vue'
import { getList, add, update, del } from "@/api/xxx"
import { useStore } from 'vuex'
import dayjs from 'dayjs'
```

#### 3.2 数据定义

```javascript
// 列表数据
const loading = ref(false)
const tableData = ref([])
const tableRef = ref(null)

// 分页
const page = reactive({
  currentPage: 1,
  pageSize: 10,
  total: 0
})

// 搜索参数
const query = reactive({})

// 表格列配置
const tableColumns = [
  { label: '名称', prop: 'name', width: 150 },
  { label: '状态', prop: 'status', width: 100 },
  { label: '创建时间', prop: 'createTime', width: 180 }
]

// 搜索字段配置
const searchFields = [
  { prop: 'name', label: '名称', type: 'input', placeholder: '请输入名称' },
  { prop: 'status', label: '状态', type: 'select', options: [] }
]

// 选择数据
const selectData = ref([])
const hasSelection = computed(() => true)
const hasIndex = computed(() => true)
const menuWidth = 220
```

#### 3.3 权限控制

```javascript
const store = useStore()
const permission = computed(() => store.getters['user/permission'])

const permissionList = computed(() => ({
  addBtn: !!permission.value["xxx:add"],
  editBtn: !!permission.value["xxx:update"],
  delBtn: !!permission.value["xxx:del"],
  exportBtn: !!permission.value["xxx:export"],
  importBtn: !!permission.value["xxx:import"]
}))
```

#### 3.4 列表加载

```javascript
const onLoad = async () => {
  loading.value = true
  try {
    const params = {
      currentPage: page.currentPage,
      pageSize: page.pageSize,
      ...query
    }
    const res = await getList(params)
    tableData.value = res.data.records || res.data.list || []
    page.total = res.data.total || 0
  } finally {
    loading.value = false
  }
}

// 搜索
const searchChange = (params) => {
  Object.assign(query, params)
  page.currentPage = 1
  onLoad()
}

// 重置
const resetChange = () => {
  page.currentPage = 1
  onLoad()
}
```

#### 3.5 操作处理

```javascript
// 新增
const handleAdd = () => {
  dialogType.value = 'add'
  dialogVisible.value = true
}

// 编辑
const handleEdit = (row) => {
  dialogType.value = 'edit'
  formData.value = { ...row }
  dialogVisible.value = true
}

// 查看
const handleView = (row) => {
  dialogType.value = 'view'
  formData.value = { ...row }
  dialogVisible.value = true
}

// 删除
const handleDelete = async (row) => {
  await ElMessageBox.confirm('确定删除该数据吗？', '提示', { type: 'warning' })
  await del(row.id)
  ElMessage.success('删除成功')
  onLoad()
}

// 批量删除
const batchDel = async () => {
  const ids = selectData.value.map(item => item.id)
  await ElMessageBox.confirm(`确定删除选中的 ${ids.length} 条数据吗？`, '提示', { type: 'warning' })
  await del(ids.join(','))
  ElMessage.success('批量删除成功')
  onLoad()
}

// 导出
const handleExport = async () => {
  const params = { ...query }
  const res = await request({
    url: '/api/xxx/export',
    method: 'post',
    data: params,
    responseType: 'blob'
  })
  downloadStream(res.data, '导出数据.xlsx')
}

// 选择变化
const handleSelectionChange = (selection) => {
  selectData.value = selection
}
```

### 步骤 4：编写表单弹窗

#### 4.1 Props 定义

```javascript
const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false
  },
  dialogType: {
    type: String,
    default: 'add' // 'add' | 'edit' | 'view'
  },
  formData: {
    type: Object,
    default: () => ({})
  }
})

const emit = defineEmits(['update:modelValue', 'confirm', 'cancel'])
```

#### 4.2 表单配置

```javascript
const formRef = ref(null)
const labelPosition = ref('right')
const labelWidth = 120
const span = 12

// 表单数据
const form = reactive({})

// 监听 formData 变化
watch(() => props.formData, (val) => {
  Object.assign(form, val)
}, { immediate: true, deep: true })

// 弹窗标题
const dialogTitle = computed(() => ({
  add: '新增',
  edit: '编辑',
  view: '查看'
}[props.dialogType]))

// 是否查看模式
const isView = computed(() => props.dialogType === 'view')
```

#### 4.3 确认提交

```javascript
const handleConfirm = async () => {
  try {
    await formRef.value.validate()
    if (props.dialogType === 'add') {
      await add(form)
      ElMessage.success('新增成功')
    } else {
      await update(form)
      ElMessage.success('编辑成功')
    }
    emit('update:modelValue', false)
    emit('confirm')
  } catch (error) {
    console.error('表单验证失败', error)
  }
}

const handleClose = () => {
  formRef.value?.resetFields()
  emit('cancel')
}
```

---

## 组件使用规范

### 搜索表单（SearchForm）

搜索字段类型：`input` | `select` | `date` | `dateRange`

```javascript
const searchFields = [
  { prop: 'name', label: '名称', type: 'input' },
  { prop: 'status', label: '状态', type: 'select', options: [
    { label: '启用', value: 1 },
    { label: '禁用', value: 0 }
  ]},
  { prop: 'dateRange', label: '时间', type: 'dateRange' }
]
```

### 表格列配置

```javascript
const tableColumns = [
  { label: '序号', prop: 'index', type: 'index', width: 80 },
  { label: '名称', prop: 'name', width: 150, minWidth: 100 },
  { label: '状态', prop: 'status', width: 100, align: 'center' },
  { label: '时间', prop: 'createTime', width: 180 }
]
```

### 操作列

```vue
<el-table-column label="操作" :width="220" fixed="right" align="center">
  <template #default="{ row }">
    <el-button type="primary" link size="small" @click="handleView(row)">
      <el-icon><View /></el-icon>查看
    </el-button>
    <el-button type="warning" link size="small" @click="handleEdit(row)">
      <el-icon><Edit /></el-icon>编辑
    </el-button>
    <el-button type="danger" link size="small" @click="handleDelete(row)">
      <el-icon><Delete /></el-icon>删除
    </el-button>
  </template>
</el-table-column>
```

### 分页器

```vue
<el-pagination
  v-model:current-page="page.currentPage"
  v-model:page-size="page.pageSize"
  :total="page.total"
  :page-sizes="[10, 20, 50, 100]"
  layout="total, sizes, prev, pager, next"
  @size-change="onLoad"
  @current-change="onLoad"
/>
```

---

## 样式规范

### 布局顺序

从上到下依次是：搜索区域 → 工具栏 → 表格区域 → 分页器

### 内边距

搜索区域、工具栏、表格区域都需要设置 `padding-left: 10px`：

```scss
.page-container {
  height: 100%;
  box-sizing: border-box;
  padding-left: 10px;

  .search-area,
  .toolbar,
  .table-area {
    padding-left: 0;
  }

  .toolbar {
    margin-bottom: 10px;

    .el-button .el-icon {
      margin-right: 4px;
    }
  }
}
```

### 表格高度

必须设置固定高度：
- `height="calc(100% - 200px)"` - 无搜索区
- `height="calc(100% - 240px)"` - 有搜索区

```vue
<el-table height="calc(100% - 240px)" border>...</el-table>
```

### 表格样式

```scss
.table-wrapper {
  height: 100%;
  box-sizing: border-box;

  :deep(.el-table__header th) {
    color: #000;
  }

  :deep(.el-table__header .cell) {
    color: #000;
  }

  :deep(.el-table__cell) {
    text-align: center;
  }
}
```

---

## 表单输入规范

### Input Placeholder

所有输入框必须添加 placeholder，格式为 **"请输入" + label**：

```vue
<el-input v-model="form.name" placeholder="请输入名称" />
```

### Select Placeholder

格式为 **"请选择" + label** 或 **"请输入或选择" + label**：

```vue
<!-- 纯选择 -->
<el-select v-model="form.status" placeholder="请选择状态">
  <el-option label="启用" value="1" />
</el-select>

<!-- 可输入可选择 -->
<el-select v-model="form.code" filterable allow-create default-first-option placeholder="请输入或选择编码">
  <el-option label="选项1" value="1" />
</el-select>
```

### Date Picker Placeholder

格式为 **"请选择" + label**：

```vue
<el-date-picker
  v-model="form.date"
  type="datetime"
  placeholder="请选择时间"
  value-format="YYYY-MM-DD HH:mm:ss"
  format="YYYY-MM-DD HH:mm:ss"
/>
```

### 查看模式表单字段

**查看模式下，所有可编辑字段必须使用 `:disabled="isView"` 设置禁用状态。**

```vue
<el-input v-model="form.name" placeholder="请输入名称" :disabled="isView" />
<el-select v-model="form.type" placeholder="请选择类型" :disabled="isView">...</el-select>

<!-- 联动回填字段：始终禁用 -->
<el-input v-model="form.countryMaterialName" disabled />
```

---

## API 调用规范

### 标准 CRUD 接口

```javascript
import { getList, add, update, del, view } from "@/api/xxx"

// 列表
const res = await getList({ currentPage, pageSize, ...query })
tableData.value = res.data.records
page.total = res.data.total

// 新增
await add(form)
ElMessage.success('新增成功')

// 编辑
await update(form)
ElMessage.success('编辑成功')

// 删除
await del(row.id)
ElMessage.success('删除成功')

// 详情
const res = await view(row.id)
Object.assign(form, res.data)
```

### 文件导出

```javascript
import { downloadStream } from '@/utils/util'

const handleExport = async () => {
  const res = await request({
    url: '/api/xxx/export',
    method: 'post',
    data: query,
    responseType: 'blob'
  })
  downloadStream(res.data, '导出文件.xlsx')
}
```

---

## 字典数据处理

### 静态字典

```javascript
const statusOptions = [
  { label: '启用', value: 1 },
  { label: '禁用', value: 0 }
]
```

### 动态字典

```javascript
const typeOptions = ref([])

onMounted(async () => {
  const res = await getDictData('xxx_type')
  typeOptions.value = res.data
})
```

### 级联联动

```javascript
watch(() => form.orgId, async (newVal) => {
  if (newVal) {
    const res = await getChildDepts(newVal)
    form.areaId = '' // 重置联动字段
    areaOptions.value = res.data
  }
})
```

---

## 日期格式化规范

使用 dayjs 格式化日期：

```javascript
import dayjs from 'dayjs'

// 初始化时间
const defaultTime = dayjs().format('YYYY-MM-DD HH:mm:ss')

// 格式化
dayjs().format('YYYY-MM-DD')
dayjs().format('YYYY-MM-DD HH:mm:ss')

// 解析
dayjs('2024-01-01').format('YYYY-MM-DD')
```

---

## 常见场景处理

### 场景 1：仅新增和查看（无编辑）

```javascript
// 不显示编辑按钮
<el-button type="warning" link size="small" @click="handleEdit(row)" v-if="permissionList.editBtn && false">
  <el-icon><Edit /></el-icon>编辑
</el-button>

// 或移除 editBtn 权限
const permissionList = computed(() => ({
  addBtn: !!permission.value["xxx:add"],
  delBtn: !!permission.value["xxx:del"],
  viewBtn: !!permission.value["xxx:view"]
}))
```

### 场景 2：弹窗内表单分组

```vue
<div v-for="group in formGroups" :key="group.prop" class="form-group">
  <h3 v-if="group.label">{{ group.label }}</h3>
  <el-row :gutter="20">
    <el-col v-for="col in group.column" :key="col.prop" :span="col.span || 12">
      <el-form-item :label="col.label" :prop="col.prop">
        <!-- 表单项 -->
      </el-form-item>
    </el-col>
  </el-row>
</div>
```

### 场景 3：确认删除

```javascript
const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(
      `确定删除 "${row.name}" 吗？`,
      '删除确认',
      { type: 'warning' }
    )
    await del(row.id)
    ElMessage.success('删除成功')
    onLoad()
  } catch {
    // 用户取消
  }
}
```

---

## 开发检查清单

完成页面开发后，确认以下内容：

- [ ] API 接口调用正确（getList, add, update, del）
- [ ] 分页逻辑正确（page 对象结构）
- [ ] 搜索参数处理正确（query 对象，SearchForm fields）
- [ ] 权限控制正确（permissionList computed）
- [ ] 表单验证规则正确（rules）
- [ ] 提交逻辑正确（handleConfirm）
- [ ] 所有 placeholder 格式正确
- [ ] 表格高度设置正确
- [ ] 操作列样式正确（图标+文字）
- [ ] 日期使用 dayjs 格式化
