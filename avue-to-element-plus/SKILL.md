---
name: avue-to-element-plus
description: |
  Avue 框架迁移到 Element Plus 组件库。确保业务逻辑完全不变，仅替换视图层组件。
  使用场景：
  - 用户提到"avue迁移"、"avue重构"、"替换avue"
  - 用户提到"avue-crud"、"avue-form"、"avue-detail" 需要重构成 Element Plus
  - 用户提到"移除avue"、"不用avue了"
  - 用户提到 Avue 组件的性能问题或二次开发困难
  核心能力：将 Avue 的 option 配置自动转换为 Element Plus 的等效配置，保持 API 调用、数据处理、权限控制逻辑不变。
triggers:
  - avue迁移
  - avue重构
  - avue-crud
  - avue-form
  - avue-detail
  - avue组件替换
  - 移除avue
  - Element Plus重构
---

# Avue → Element Plus 迁移技能

## 核心原则

1. **逻辑不变**：保持 API 调用、数据处理、业务逻辑完全不变
2. **UI 重构**：只替换视图层组件，保持用户体验一致
3. **配置转换**：将 Avue option 配置转换为 Element Plus 等效配置

---

## Avue Option → Element Plus 配置转换

### 1. avue-crud 配置转换

#### Avue crud option 示例
```javascript
{
  menuWidth: 280,
  addBtn: false,
  editBtn: true,
  delBtn: true,
  viewBtn: true,
  column: [
    { label: "名称", prop: "name", search: true },
    { label: "状态", prop: "status", type: "select", dicData: statusList },
    { label: "创建时间", prop: "createTime", width: 180 }
  ]
}
```

#### 转换后的 Element Plus 配置
```vue
<template>
  <!-- 搜索表单 - 使用 SearchForm 组件 src\components\SearchForm\index.vue -->
  <SearchForm
    v-model="query"
    :fields="searchFields"
    @search="searchChange"
    @reset="resetChange"
    class="search-area"
  />

  <!-- 工具栏 - 新增按钮放在搜索区域下方，所有按钮使用图标+文字 -->
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

  <!-- 表格区域 - table-wrapper 需要设置 height: 100% -->
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
      <el-table-column v-for="col in tableColumns" :key="col.prop" :label="col.label" :prop="col.prop" :width="col.width" :min-width="col.minWidth">
        <template #default="{ row }">
          <!-- 使用 v-permission 保持权限控制 -->
          <span v-if="col.prop === 'status'">
            <el-tag :type="getStatusType(row.status)">{{ row.statusName }}</el-tag>
          </span>
          <slot v-else :name="col.prop" :row="row">{{ row[col.prop] }}</slot>
        </template>
      </el-table-column>
      <!-- 操作列 - 使用图标+文字 -->
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

    <!-- 分页器 - 写在表格区域内 -->
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
</template>
```

### 2. avue-form 配置转换

#### Avue form option 示例（分组）
```javascript
{
  span: 6,
  labelPosition: 'top',
  group: [
    {
      label: '基本信息',
      prop: 'info',
      column: [
        { label: '名称', prop: 'name', rules: [{ required: true, message: '请输入' }] },
        { label: '类型', prop: 'type', type: 'select', dicData: typeList }
      ]
    }
  ]
}
```

#### 转换后的 Element Plus
```vue
<template>
  <el-form ref="formRef" :model="form" :label-position="labelPosition" :label-width="labelWidth + 'px'">
    <!-- 分组 -->
    <div v-for="group in groups" :key="group.prop" class="form-group">
      <h3 v-if="group.label">{{ group.label }}</h3>
      <el-row :gutter="20">
        <el-col v-for="col in group.column" :key="col.prop" :span="col.span || 6">
          <el-form-item :label="col.label" :prop="col.prop" :rules="convertRules(col.rules)">
            <!-- 输入框 -->
            <el-input v-if="!col.type || col.type === 'input'" v-model="form[col.prop]" />
            <!-- 下拉选择 -->
            <el-select v-else-if="col.type === 'select'" v-model="form[col.prop]">
              <el-option v-for="opt in col.dicData" :key="opt.value" :label="opt.label" :value="opt.value" />
            </el-select>
            <!-- 日期选择 -->
            <el-date-picker v-else-if="col.type === 'date'" v-model="form[col.prop]" type="date" />
            <!-- 日期时间 -->
            <el-date-picker v-else-if="col.type === 'datetime'" v-model="form[col.prop]" type="datetime" />
            <!-- 文本域 -->
            <el-input v-else-if="col.type === 'textarea'" v-model="form[col.prop]" type="textarea" :rows="col.minRows || 3" />
            <!-- 默认输入 -->
            <el-input v-else v-model="form[col.prop]" />
          </el-form-item>
        </el-col>
      </el-row>
    </div>
  </el-form>
</template>
```

### 3. 关键配置转换对照表

| Avue 属性 | Element Plus 等效 | 说明 |
|-----------|------------------|------|
| `search: true` | el-form-item + query 对象 | 搜索字段 |
| `type: "select"` | el-select | 下拉选择 |
| `type: "date"` | el-date-picker type="date" | 日期 |
| `type: "datetime"` | el-date-picker type="datetime" | 日期时间 |
| `type: "textarea"` | el-input type="textarea" | 文本域 |
| `dicData` | el-option 列表 | 静态字典 |
| `dicUrl` | 远程数据 + watch | 远程字典 |
| `cascader: ["field1"]` | el-cascader 或联动 watch | 级联 |
| `cell: true` | 弹窗编辑或直接编辑 | 行内编辑 |
| `props: { label, value }` | :label, :value | 字段映射 |
| `span: 6` | :span="6" | 栅格布局 |

### 4. 事件转换对照表

| Avue 事件 | Element Plus 等效 | 说明 |
|-----------|------------------|------|
| `@search-change` | `@search` 或 `@query-change` | 搜索触发 |
| `@search-reset` | `@reset` | 重置触发 |
| `@refresh-change` | `@reload` | 刷新 |
| `@on-load` | `onMounted` 或 watch page | 初始化/分页 |
| `@selection-change` | `@selection-change` | 选择变化 |
| `@row-save` | `@add` (弹窗确认) | 行新增 |
| `@row-update` | `@edit` (弹窗确认) | 行更新 |
| `@row-del` | `@delete` | 行删除 |

### 5. 权限控制保持

Avue 的 `permission` 对象直接迁移，保持按钮级别权限控制：

```javascript
// 迁移前 (Avue)
// <avue-crud :permission="permissionList">

// 迁移后 (Element Plus) - 使用 v-if 保持权限
const permissionList = computed(() => ({
  addBtn: !!permission.value["hse:xxx:add"],
  editBtn: !!permission.value["hse:xxx:update"],
  delBtn: !!permission.value["hse:xxx:del"],
  viewBtn: !!permission.value["hse:xxx:view"]
}))

// 模板中使用 v-if
<el-button v-if="permissionList.addBtn" type="primary">新增</el-button>
```

---

## 迁移步骤

### 步骤 1：分析原 Avue 组件

读取原组件，完成以下分析：

1. **识别组件类型**：
   - `avue-crud` → 需要表格 + 分页 + 搜索
   - `avue-form` → 需要表单（可能有分组）
   - `avue-detail` → 需要详情展示

2. **提取配置**：
   - `option.column` → 列定义
   - `option.group` → 分组信息
   - `option.menu*` → 菜单配置

3. **提取数据处理逻辑**：
   - API 调用（getList, add, update, del）
   - 搜索参数处理
   - 分页逻辑
   - 表单提交处理

4. **提取权限控制**：
   - `permissionList` computed
   - 各按钮的 `v-if="permissionList.xxxBtn"`

### 步骤 2：创建 Element Plus 组件结构

```vue
<script setup name="迁移后的组件名">
// 1. 导入保持不变
import { getList, add, update, del } from "@/api/xxx"
import { ref, reactive, computed, onMounted } from 'vue'
import { View, Edit, Delete, Plus } from '@element-plus/icons-vue'
import SearchForm from '@/components/SearchForm/index.vue'

// 2. 数据定义 - 保持原有数据结构
const loading = ref(false)
const tableData = ref([])
const page = reactive({ currentPage: 1, pageSize: 10, total: 0 })
const query = reactive({})
const form = ref({})

// 3. 权限控制 - 保持原有逻辑
const permissionList = computed(() => ({
  addBtn: !!permission.value["xxx:add"],
  editBtn: !!permission.value["xxx:update"],
  delBtn: !!permission.value["xxx:del"]
}))

// 4. API 调用逻辑 - 保持完全不变
const onLoad = () => { /* 原有逻辑 */ }
const searchChange = (params) => { /* 原有逻辑 */ }

// 5. 表单配置 - 将 Avue option.column 转换为 Element Plus 配置
const tableColumns = computed(() => convertColumns(avueOption.column))
// SearchForm 字段配置
const searchFields = computed(() => convertSearchFields(avueOption.column || []))
</script>
```

### 步骤 3：转换 Avue Option 配置

#### 配置转换函数

```javascript
/**
 * 转换 Avue column 配置为 Element Plus 表格列定义
 */
function convertColumns(columns) {
  return columns.map(col => ({
    label: col.label,
    prop: col.prop,
    width: col.width,
    search: col.search,
    type: col.type,
    dicData: col.dicData,
    // ... 其他必要字段
  }))
}

/**
 * 转换 Avue column 为 SearchForm 字段配置
 * Avue search:true 的列 → SearchForm fields
 */
function convertSearchFields(columns) {
  return columns
    .filter(col => col.search)
    .map(col => ({
      prop: col.prop,
      label: col.label,
      type: col.type === 'select' ? 'select' : (col.type === 'date' ? 'date' : 'input'),
      placeholder: col.placeholder,
      options: col.dicData ? col.dicData.map(d => ({
        label: d.label || d.name,
        value: d.value || d.code
      })) : []
    }))
}

/**
 * 转换 Avue rules 为 Element Plus rules
 */
function convertRules(avueRules) {
  if (!avueRules) return []
  return avueRules.map(rule => ({
    required: rule.required,
    message: rule.message,
    trigger: rule.trigger
  }))
}
```

### 步骤 4：处理特殊场景

#### 4.1 级联选择 (cascader)

Avue 的 `cascader: ["area"]` 表示联动，转换为 watch：

```javascript
// Avue
{
  prop: "orgId",
  cascader: ["area"]  // orgId 变化时联动 area
}

// Element Plus
watch(() => form.orgId, async (newVal) => {
  if (newVal) {
    const res = await getChildDepts(newVal)
    form.area = '' // 重置联动字段
    areaOptions.value = res.data
  }
})
```

#### 4.2 行内编辑 (cell: true)

Avue 的行内编辑转换为弹窗编辑：

```vue
<!-- Avue -->
<!-- <avue-crud :option="option" :data="tableData" @row-save="handleSave"> -->

<!-- Element Plus - 使用弹窗代替行内编辑 -->
<el-table :data="tableData">
  <!-- 列定义 -->
  <el-table-column label="名称" prop="name">
    <template #default="{ row }">
      <el-input v-if="row.$editing" v-model="row.name" />
      <span v-else>{{ row.name }}</span>
    </template>
  </el-table-column>
</el-table>

<!-- 或使用专门的编辑弹窗 -->
<el-dialog v-model="editDialogVisible">
  <el-form :model="editForm">
    <el-form-item label="名称">
      <el-input v-model="editForm.name" />
    </el-form-item>
  </el-form>
</el-dialog>
```

#### 4.3 动态字典 (dicUrl)

```javascript
// Avue
{
  prop: "status",
  dicUrl: "/api/dict/status",
  props: { label: "name", value: "code" }
}

// Element Plus - 使用 watch + 远程加载
const statusOptions = ref([])
watch(() => form.orgId, async () => {
  if (form.orgId) {
    const res = await request({ url: '/api/dict/status' })
    statusOptions.value = res.data
  }
}, { immediate: true })
```

---

## 页面布局规范

### 布局顺序（从上到下）

1. **搜索区域** - `SearchForm` 组件，左内边距 10px
2. **工具栏** - 包含新增按钮等操作，左内边距 10px
3. **表格区域** - `el-table` 为主体，左内边距 10px
4. **分页器** - 固定在底部

### 整体容器内边距

搜索区域、工具栏、表格区域都需要设置 `padding-left: 10px`，确保内容不贴左边缘：

```vue
<template>
  <div class="page-container">
    <!-- 搜索区域 -->
    <SearchForm class="search-area" />

    <!-- 工具栏 -->
    <div class="toolbar">
      <el-button type="primary">
        <el-icon><Plus /></el-icon>新增
      </el-button>
      <el-button>
        <el-icon><Download /></el-icon>导出
      </el-button>
    </div>

    <!-- 表格区域 -->
    <el-table class="table-area" />
  </div>
</template>

<style lang="scss" scoped>
.page-container {
  height: 100%;
  box-sizing: border-box;

  .search-area,
  .toolbar,
  .table-area {
    padding-left: 10px;
  }

  .toolbar .el-button .el-icon {
    margin-right: 4px;
  }
}
</style>
```

### 表格高度固定

所有业务页面的 `el-table` 必须设置 `height="calc(100% - 240px)"`，使表格始终填满可用空间：

```vue
<!-- 错误：表格不设置固定高度 -->
<el-table :data="tableData">...</el-table>

<!-- 正确：表格设置固定高度，240px = 搜索区(约120px) + 工具栏(约50px) + 分页器(约70px) -->
<el-table :data="tableData" height="calc(100% - 240px)" border>...</el-table>
```

### 表格列 overflow tooltip

文本类列建议添加 `show-overflow-tooltip` 属性，当内容超出列宽时显示 tooltip 而不撑开表格行：

```vue
<!-- 正确：为文本列添加 show-overflow-tooltip -->
<el-table-column prop="name" label="名称" min-width="120" show-overflow-tooltip />
<el-table-column prop="code" label="编码" width="120" show-overflow-tooltip />

<!-- 数字/日期等固定宽度列不需要 tooltip -->
<el-table-column prop="amount" label="金额" width="120" />
```

### 表格样式规范

表格表头字体颜色设置为黑色，操作列表头居中：

```scss
.table-wrapper {
  height: 100%;
  box-sizing: border-box;

  // 表头字体颜色设置为黑色
  :deep(.el-table__header th) {
    color: #000;
  }

  :deep(.el-table__header .cell) {
    color: #000;
  }

  // 操作列表头居中
  :deep(.el-table__header .el-table__cell) {
    text-align: center;
  }

  // 操作列按钮图标与文字间距
  :deep(.el-button--small) {
    gap: 4px;
  }
}
```

### 工具栏样式

```vue
<div class="toolbar">
  <el-button type="primary" @click="handleAdd" v-if="permissionList.addBtn">
    <el-icon><Plus /></el-icon>新增
  </el-button>
  <el-button @click="handleExport" v-if="permissionList.exportBtn">
    <el-icon><Download /></el-icon>导出
  </el-button>
  <el-button @click="handleImport" v-if="permissionList.importBtn">
    <el-icon><Upload /></el-icon>导入
  </el-button>
</div>

<style lang="scss" scoped>
.zlzn-custom {
  display: flex;
  flex-direction: column;
  height: 100%;
  box-sizing: border-box;
  padding-left: 10px;

  .toolbar {
    margin-bottom: 10px;

    .el-button {
      .el-icon {
        margin-right: 4px;
      }
    }
  }

  .el-pagination {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    margin-top: 16px;
    padding: 0;
  }
}
</style>
```

### 分页器 layout 顺序固定

```
layout="total, sizes, prev, pager, next"
```

- **total**: 总条数
- **sizes**: 每页条数选择器
- **prev**: 上一页
- **pager**: 页码
- **next**: 下一页
- **jumper**: 页码跳转

---

## 操作列规范

### 操作列必须写在表格内

Avue 的 `#menu` 插槽中的操作按钮，必须转换为 Element Plus 表格的 `el-table-column` 操作列：


### 操作列使用图标+文字样式

使用 `link` 类型的按钮配合图标，简洁美观：

```vue
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
```

- `fixed="right"`：固定在右侧
- `align="center"`：表头文字居中
- `type="primary/warning/danger"`：文字颜色（查看=primary，编辑=warning，删除=danger）
- `link`：文字按钮样式，简洁不占空间
- `size="small"`：按钮尺寸为小型
- `<el-icon>`：图标组件，图标与文字之间自动有间距

---

## ⚠️ 重要：表单输入规范

### 查看模式表单字段规范

**查看模式下，所有可编辑字段必须使用 `:disabled="dialogType === 'view'"` 设置禁用状态。**

```vue
<!-- 正确：使用条件判断，查看时禁用，新增/编辑时可编辑 -->
<el-input v-model="form.name" placeholder="请输入名称" :disabled="dialogType === 'view'" />
<el-select v-model="form.type" placeholder="请选择类型" :disabled="dialogType === 'view'">...</el-select>
<el-date-picker v-model="form.date" placeholder="请选择日期" :disabled="dialogType === 'view'" />

<!-- 自动填充字段（联动回填字段）：始终禁用 -->
<el-input v-model="form.countryMaterialName" disabled />
```

**联动回填字段的处理：**
- 通过选择联动回填的字段（如国家危险废物名录名称、危险废物类别等），始终设置为 `disabled`，不参与编辑
- 其他业务字段使用 `:disabled="dialogType === 'view'"` 条件判断



### Input Placeholder 规范（必须遵守）

所有 `el-input` 输入框必须添加 `placeholder`，格式为 **"请输入" + label**：

```vue
<!-- 正确 -->
<el-input v-model="form.name" placeholder="请输入名称" />
<el-input v-model="form.code" placeholder="请输入编码" disabled />

<!-- 错误：缺少 placeholder -->
<el-input v-model="form.name" />
```

### 下拉选择框 Placeholder 规范

`el-select` 下拉选择框的 placeholder 格式为 **"请选择" + label** 或 **"请输入或选择" + label**：

```vue
<!-- 纯选择（不可输入） -->
<el-select v-model="form.type" placeholder="请选择类型">
  <el-option label="选项1" value="1" />
</el-select>

<!-- 可输入可选择 -->
<el-select
  v-model="form.code"
  filterable
  allow-create
  default-first-option
  placeholder="请输入或选择编码"
>
  <el-option label="选项1" value="1" />
</el-select>
```

### 日期选择框 Placeholder 规范

`el-date-picker` 日期选择框的 placeholder 格式为 **"请选择" + label**：

```vue
<!-- 日期选择 -->
<el-date-picker
  v-model="form.date"
  type="date"
  placeholder="请选择日期"
/>

<!-- 日期时间选择 -->
<el-date-picker
  v-model="form.datetime"
  type="datetime"
  value-format="YYYY-MM-DD HH:mm:ss"
  format="YYYY-MM-DD HH:mm:ss"
  placeholder="请选择时间"
/>
```

---

## 迁移检查清单

完成迁移后，确认以下内容保持不变：

- [ ] API 调用接口（getList, add, update, del）
- [ ] 分页逻辑（page 对象结构）
- [ ] 搜索参数处理（query 对象，SearchForm fields 配置）
- [ ] 权限控制（permissionList computed）
- [ ] 表单验证规则（rules）
- [ ] 数据提交逻辑（handleSubmit）
- [ ] 业务逻辑（canEdit, isUserOperate 等）
- [ ] ⚠️ 所有表单输入框 placeholder 格式为 "请输入" + label
- [ ] SearchForm 组件正确导入和使用
- [ ] ⚠️ 文本类表格列添加 `show-overflow-tooltip` 属性

---

## 常见问题处理

### 1. dicData 字典数据

Avue 的 `dicData` 可以是静态数组，Element Plus 需要 `el-option` 列表：

```javascript
// Avue
{ label: "状态", prop: "status", type: "select", dicData: [{ label: "启用", value: 1 }] }

// Element Plus
<el-select v-model="form.status">
  <el-option v-for="item in statusList" :key="item.value" :label="item.label" :value="item.value" />
</el-select>
```

### 2. 搜索字段过滤

```javascript
// Avue - search: true 自动生成搜索表单项
// Element Plus - 手动筛选 search: true 的列
const searchColumns = columns.filter(col => col.search)
```

### 3. 空值处理

```javascript
// Avue 自动处理空值
// Element Plus - 需要手动处理
const handleDelete = async (row) => {
  await ElMessageBox.confirm('确定删除吗?')
  await del(row.id)
  ElMessage.success('删除成功')
  onLoad()
}
```

---

## 工具脚本

### 批量转换脚本（可选）

对于大量相似页面，可以编写转换脚本：

```javascript
// convert-avue-option.js
/**
 * Avue Option 配置转换工具
 * 将 Avue 的 option 对象转换为 Element Plus 配置
 */

export function convertAvueOption(avueOption) {
  return {
    // 表格配置
    tableColumns: convertColumns(avueOption.column || []),
    searchFields: convertSearchFields(avueOption.column || []),
    menuWidth: avueOption.menuWidth || 220,
    hasSelection: avueOption.selection !== false,
    hasIndex: avueOption.index !== false,

    // 表单配置
    labelPosition: avueOption.labelPosition || 'right',
    labelWidth: avueOption.labelWidth || 100,
    span: avueOption.span || 6,
    groups: convertGroups(avueOption.group || [])
  }
}

/**
 * 转换 Avue column 为 SearchForm 字段配置
 */
function convertSearchFields(columns) {
  return columns
    .filter(col => col.search)
    .map(col => ({
      prop: col.prop,
      label: col.label,
      type: col.type === 'select' ? 'select' : (col.type === 'date' ? 'date' : 'input'),
      placeholder: col.placeholder,
      options: col.dicData ? col.dicData.map(d => ({
        label: d.label || d.name,
        value: d.value || d.code
      })) : []
    }))
}
```
