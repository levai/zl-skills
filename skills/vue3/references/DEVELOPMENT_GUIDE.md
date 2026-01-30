# 业务开发详细指南

本文档包含业务开发的详细规范和最佳实践。

## 目录结构规范

### 组件目录结构

```
src/components/MyModule/
├── AbsView/
│   └── AbsMyModuleView.ts          # 基类视图
├── Selectors/
│   ├── AbsMyModuleSelect.tsx        # 选择器组件
│   └── index.ts                     # 导出
├── source/
│   ├── List.vue                     # 列表页面
│   ├── Form.vue                     # 表单页面
│   └── Detail.vue                   # 详情页面
├── store/
│   └── index.ts                     # Vuex store
├── types/
│   └── index.ts                     # 类型定义
└── index.ts                         # 模块导出
```

### 视图目录结构

```
src/views/myModule/
├── overview/
│   └── index.vue                    # 概览页面
├── list/
│   └── index.vue                    # 列表页面
└── detail/
    └── index.vue                     # 详情页面
```

## 路由配置详解

### 路由 Meta 配置

```typescript
meta: {
  name: 'My Module',              // 菜单显示名称
  icon: 'top-nav-icon',            // 图标名称
  globalKey: 'myModule',           // 全局键，用于 API
  globalModel: 'myModule',         // 全局模型
  pid: PidValue.MY_MODULE,         // 模块ID
  matrixSet: { projectId: true },  // 是否需要项目ID
  affix: true,                      // 是否固定在标签页
  hidden: false,                    // 是否隐藏
  noCache: false,                   // 是否不缓存
  breadcrumb: true                  // 是否显示面包屑
}
```

### 路由类型

```typescript
// 概览页面
{
  path: 'overview',
  component: () => import('@/views/myModule/overview/index.vue'),
  meta: { name: '概览', affix: true }
}

// 列表页面
{
  path: 'list',
  component: () => import('@/views/myModule/list/index.vue'),
  meta: { name: '列表管理' }
}

// 详情页面（动态路由）
{
  path: 'detail/:id',
  component: () => import('@/views/myModule/detail/index.vue'),
  meta: { name: '详情', hidden: true }
}
```

## Vuex Store 详解

### Store 结构

```typescript
import { VuexModule } from '@/store'

interface State {
  // 列表数据
  list: any[]
  total: number
  
  // 详情数据
  detail: any
  
  // 加载状态
  loading: boolean
  
  // 其他状态
  filters: any
}

const state: State = {
  list: [],
  total: 0,
  detail: {},
  loading: false,
  filters: {}
}

const actions = {
  // 获取列表
  async fetchList ({ commit, state }, params: IListParams) {
    commit('SET_LOADING', true)
    try {
      const res = await http.get('/api/my-module/list', { params })
      commit('SET_LIST', res.data.list)
      commit('SET_TOTAL', res.data.total)
      return res
    } catch (error) {
      console.error('获取列表失败:', error)
      throw error
    } finally {
      commit('SET_LOADING', false)
    }
  },
  
  // 获取详情
  async fetchDetail ({ commit }, id: number) {
    try {
      const res = await http.get(`/api/my-module/${id}`)
      commit('SET_DETAIL', res.data)
      return res
    } catch (error) {
      console.error('获取详情失败:', error)
      throw error
    }
  },
  
  // 新增
  async addItem ({ commit }, data: any) {
    try {
      const res = await http.post('/api/my-module', data)
      return res
    } catch (error) {
      console.error('新增失败:', error)
      throw error
    }
  },
  
  // 更新
  async updateItem ({ commit }, { id, ...data }: any) {
    try {
      const res = await http.put(`/api/my-module/${id}`, data)
      return res
    } catch (error) {
      console.error('更新失败:', error)
      throw error
    }
  },
  
  // 删除
  async deleteItem ({ commit }, id: number) {
    try {
      const res = await http.delete(`/api/my-module/${id}`)
      return res
    } catch (error) {
      console.error('删除失败:', error)
      throw error
    }
  }
}

const mutations = {
  SET_LIST (state: State, list: any[]) {
    state.list = list
  },
  SET_TOTAL (state: State, total: number) {
    state.total = total
  },
  SET_DETAIL (state: State, detail: any) {
    state.detail = detail
  },
  SET_LOADING (state: State, loading: boolean) {
    state.loading = loading
  },
  SET_FILTERS (state: State, filters: any) {
    state.filters = filters
  }
}

const getters = {
  filteredList (state: State) {
    // 计算属性
    return state.list.filter(item => {
      // 过滤逻辑
    })
  }
}

const store: VuexModule<State> = {
  name: 'myModule',
  namespaced: true,
  state,
  actions,
  mutations,
  getters
}

export default store
```

## AbsView 基类详解

### 标准基类结构

```typescript
import { Component, Vue } from 'vue-property-decorator'
import { namespace } from 'vuex-class'
import { StoreProvide } from '@/utils/decorators'
import store from '../store'

const $store = namespace('myModule')

@Component
@StoreProvide([store])
export default class AbsMyModuleView extends Vue {
  // Actions
  @$store.Action('fetchList')
  fetchList!: IGenericAction<IListParams, IListResult<any[]>>

  @$store.Action('fetchDetail')
  fetchDetail!: IGenericAction<{ id: number }, any>

  @$store.Action('addItem')
  addItem!: IGenericAction<any, boolean>

  @$store.Action('updateItem')
  updateItem!: IGenericAction<any, boolean>

  @$store.Action('deleteItem')
  deleteItem!: IGenericAction<{ id: number }, boolean>

  // States
  @$store.State('list')
  list!: any[]

  @$store.State('loading')
  loading!: boolean

  // Getters
  @$store.Getter('filteredList')
  filteredList!: any[]

  // 通用方法
  protected handleError (error: any) {
    console.error('操作失败:', error)
    this.$message.error(error.message || '操作失败')
  }
}
```

## 权限控制

### 权限标识

```typescript
// 在组件 data 中定义
data () {
  return {
    PERMISSION_FLAG
  }
}
```

### 使用权限控制

```vue
<!-- 按钮权限 -->
<el-button 
  v-acl="`/myModule|${PERMISSION_FLAG.ADD}`" 
  type="primary"
>
  新增
</el-button>

<!-- 菜单权限 -->
<!-- 路由配置中会自动处理 -->

<!-- 条件渲染 -->
<template v-if="$acl.check(`/myModule|${PERMISSION_FLAG.EDIT}`)">
  <el-button @click="handleEdit">编辑</el-button>
</template>
```

## 国际化

### 使用 $t() 函数

```typescript
// 在代码中使用
this.$message.success($t('Action Success'))
this.$message.error($t('Action Failed'))

// 在模板中使用
<el-button>{{ $t('Add') }}</el-button>
```

### 添加国际化文本

在 `src/lang/i18n/zh_CN/` 目录下添加对应的国际化文件。

## 表单验证

### 标准验证规则

```typescript
rules: Kv = {
  name: [
    genRequired('请输入名称'),
    { max: 64, message: '名称不得超过64个字符', trigger: 'change' },
    {
      pattern: /^[a-zA-Z0-9_\u4e00-\u9fa5]+$/g,
      message: '仅支持中文、字母、数字和下划线'
    }
  ],
  code: [
    genRequired('请输入编码'),
    {
      pattern: /^[a-z][a-z0-9_]*$/,
      message: '编码只支持小写字母、数字、下划线，且必须以字母开头'
    }
  ],
  email: [
    { type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur' }
  ]
}
```

### 自定义验证

```typescript
async validateName (rule: any, value: string, callback: Function) {
  if (!value) {
    callback()
    return
  }
  
  // 异步验证
  const exists = await this.checkNameExists(value)
  if (exists) {
    callback(new Error('名称已存在'))
  } else {
    callback()
  }
}
```

## 错误处理

### 标准错误处理模式

```typescript
try {
  await this.addItem(data)
  this.$message.success($t('Action Success'))
  this.loadData()
} catch (error) {
  console.error('操作失败:', error)
  this.$message.error(error.message || $t('Action Failed'))
}
```

### 统一错误处理

在 AbsView 基类中定义统一错误处理方法：

```typescript
protected handleError (error: any, defaultMessage: string = '操作失败') {
  console.error('操作失败:', error)
  const message = error?.response?.data?.message || error?.message || defaultMessage
  this.$message.error(message)
}
```

## 数据加载模式

### 列表加载

```typescript
loadData (params?: Partial<IListParams>) {
  const genericList = (this.$refs.list as any)
  return genericList.load(params)
}

// 刷新到第一页
loadData({ pager: { page: 1 } })

// 保持当前查询条件
loadData({ query: this.currentQuery })
```

### 详情加载

```typescript
async loadDetail () {
  const id = Number(this.$route.params.id)
  try {
    const data = await this.fetchDetail({ id })
    this.model = data
  } catch (error) {
    this.handleError(error)
  }
}
```

## 最佳实践

1. **模块化开发**：每个业务模块独立目录，包含完整的 store、组件、视图
2. **代码复用**：提取公共逻辑到基类或工具函数
3. **类型安全**：使用 TypeScript 类型定义，避免使用 any
4. **错误处理**：所有异步操作包含错误处理
5. **权限控制**：所有操作按钮添加权限控制
6. **国际化**：所有用户可见文本使用国际化
7. **代码规范**：遵循项目代码规范和 Git 提交规范
