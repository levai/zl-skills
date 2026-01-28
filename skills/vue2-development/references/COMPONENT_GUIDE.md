# Vue 组件开发详细指南

本文档包含 Vue 组件开发的详细规范和示例。

## 组件结构

### 完整组件示例

```vue
<template>
  <div class="component-wrapper">
    <el-form :model="model" :rules="rules" ref="formRef">
      <el-form-item prop="name" label="名称">
        <el-input v-model.trim="model.name" />
      </el-form-item>
    </el-form>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Ref, Watch } from 'vue-property-decorator'
import { namespace } from 'vuex-class'
import { AbsWorkbenchView } from '@/components/AbsView'
import { getRule, genRequired } from '@/utils/validator'

const $store = namespace('module/store')

@Component({
  components: {
    // 组件注册
  },
  methods: {
    getRule,
    genRequired
  }
})
export default class ComponentName extends AbsWorkbenchView {
  @Prop({ default: {} })
  value!: Kv

  @Ref()
  formRef!: ElForm

  model: Kv = {}

  rules: Kv = {
    name: [genRequired('请输入名称')]
  }

  @Watch('value', { immediate: true })
  setModel (newVal: Kv) {
    this.model = newVal
  }

  validate () {
    return this.formRef.validate()
  }
}
</script>

<style lang="scss" scoped>
.component-wrapper {
  padding: 20px;
}
</style>
```

## 装饰器详解

### @Prop 装饰器

```typescript
// 基本用法
@Prop()
name!: string

// 带默认值
@Prop({ default: '' })
description!: string

// 带类型和默认值
@Prop({ type: String, default: '' })
code!: string

// 必填属性
@Prop({ required: true })
id!: number
```

### @Ref 装饰器

```typescript
// 表单引用
@Ref()
formRef!: ElForm

// 表格引用
@Ref()
tableRef!: ElTable

// 组件引用
@Ref()
childComponent!: ChildComponent
```

### @Watch 装饰器

```typescript
// 基本监听
@Watch('value')
onValueChange (newVal: any, oldVal: any) {
  // 处理变化
}

// 立即执行
@Watch('value', { immediate: true })
onValueChange (newVal: any) {
  // 立即执行
}

// 深度监听
@Watch('obj', { deep: true })
onObjChange (newVal: any) {
  // 深度监听对象变化
}
```

### Vuex 装饰器

```typescript
const $store = namespace('module/store')

// Action
@$store.Action('fetchData')
fetchData!: IGenericAction<IListParams, IListResult<any[]>>

// State
@$store.State('dataList')
dataList!: any[]

// Getter
@$store.Getter('filteredList')
filteredList!: any[]

// Mutation
@$store.Mutation('setData')
setData!: (data: any) => void
```

## 表单验证

### 验证规则类型

```typescript
// 必填验证
rules: Kv = {
  name: [genRequired('请输入名称')]
}

// 长度验证
rules: Kv = {
  name: [
    genRequired('请输入名称'),
    { max: 64, message: '名称不得超过64个字符', trigger: 'change' },
    { min: 2, message: '名称至少2个字符', trigger: 'change' }
  ]
}

// 正则验证
rules: Kv = {
  code: [
    genRequired('请输入编码'),
    {
      pattern: /^[a-z][a-z0-9_]*$/,
      message: '编码只支持小写字母、数字、下划线，且必须以字母开头'
    }
  ]
}

// 自定义验证
rules: Kv = {
  name: [
    {
      validator: this.validateName,
      trigger: 'blur'
    }
  ]
}

async validateName (rule: any, value: string, callback: Function) {
  if (!value) {
    callback()
    return
  }
  
  const exists = await this.checkNameExists(value)
  if (exists) {
    callback(new Error('名称已存在'))
  } else {
    callback()
  }
}
```

### 使用工具函数

```typescript
import { getRule, genRequired } from '@/utils/validator'

// 使用 getRule
rules: Kv = {
  name: [getRule('required', '请输入名称')]
}

// 使用 genRequired
rules: Kv = {
  name: [genRequired('请输入名称')]
}
```

## 样式规范

### Scoped 样式

```vue
<style lang="scss" scoped>
.component-wrapper {
  padding: 20px;
  
  .form-item {
    margin-bottom: 20px;
  }
  
  .button-group {
    display: flex;
    gap: 10px;
  }
}
</style>
```

### 深度选择器

```vue
<style lang="scss" scoped>
.component-wrapper {
  ::v-deep .el-form-item {
    margin-bottom: 20px;
  }
}
</style>
```

## 最佳实践

1. **组件继承**：始终继承对应的 AbsView 基类
2. **类型安全**：使用 TypeScript 类型注解
3. **错误处理**：异步操作包含错误处理
4. **代码复用**：提取公共逻辑到工具函数
5. **性能优化**：合理使用 computed 和 watch
