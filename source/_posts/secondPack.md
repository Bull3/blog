---
title: 基于Vue + Element的二次封装之旅
categories: 前端类
description: 描述
tags:
  - 前端
date: 2019-12-03 17:18:05
---

> **本章内容旨在对管理系统的核心组件进行封装, 通常是 表格组件 和 表单组件**
> 文章进度如下:
> 1. [x] 二次封装的核心思想
> 2. [x] 封装表格组件
> 3. [ ] 封装表单组件
> 4. [ ] 打包并发布开源的npm包

## 1. 二次封装的核心思想

> * 尽量少的改动原始组件的使用方法, 降低新组件的使用成本
> * 以数据配置为核心, 根据数据自动渲染模板, 而不是手动编写模板

## 2. 封装表格组件

### 2.1 期望封装结果

```html
<!-- 传入 columns 数组即可动态渲染表格, 其他属性及事件的用法均与 el-table 一致 -->
<b-table :columns="columns" :data="tableData">
</b-table>
```

### 2.2 实现对el-table的属性和事件的继承

> 利用vue中高阶组件传参的2个利器 [\$attrs](https://cn.vuejs.org/v2/api/#vm-attrs) 和 [\$listeners](https://cn.vuejs.org/v2/api/#vm-listeners)

```html
<el-table v-bind="$attrs" v-on="$listeners">
</el-table>
```

### 2.3 根据columns数组循环渲染el-table-column

```html
<!-- 模板: -->
<el-table v-bind="$attrs" v-on="$listeners">
    <template v-for="column in columns">
        <!-- 无插槽模式: column对象没有colSlot字段时, 将直接渲染列字段 -->
        <el-table-column v-if="!column.colSlot" :key="column.id" v-bind="column">
        </el-table-column>
        <!-- 插槽模式: column对象指定colSlot时, colSlot作为插槽的名字 表格列较为复杂或特殊时可启用插槽时, 将直接渲染插槽自定义内容 -->
        <el-table-column v-else :key="column.id" v-bind="column">
            <template slot-scope="scope">
                <slot :name="column.colSlot" v-bind="scope" />
            </template>
        </el-table-column>
    </template>
</el-table>
```

```javascript
// 数据:
columns: [
    { id: "selection", type: "selection", width: "55" },
    { id: "rname", label: "用户名称", colSlot: "rname" },
    { id: "remark", prop: "remark", label: "用户说明" },
    { id: "uname", prop: "uname", label: "用户账号" }
]
```

这里巧妙的使用了[v-bind](https://cn.vuejs.org/v2/api/#v-bind)="column"绑定对象的用法 (遍历对象所有属性, 将对象属性和值绑定在模板上)
绑定的对象为column, 因此el-table-column组件内部的props即指向column对象内的字段,
而当表格列较为复杂时, 需要启用插槽, 因此给column对象额外增加了一个字段colSlot以供具名插槽 `<slot :name="column.colSlot" />`使用
> <font color=red>**tips:** v-for的key需要手动指定, 且不要以index为key, 最好是给column对象额外添加id字段作为v-for的key</font>

> 详细分析请看我的另一篇文章[《v-for为什么要加key，能用index作为key么》](https://www.cnblogs.com/youhong/p/11327062.html)

### 2.4 添加表格默认属性
```html
<el-table v-bind="{ stripe: true, border: true, ...$attrs }" v-on="$listeners">
</el-table>
```
我们还可以预设一些默认属性, 方便表格的使用
使用es6的扩展运算符 ... 来展开\$attrs, 默认配置和用户传入值就可以完美拼接, 共同作用在模板上
这样写既可以在不传入stripe和border时提供默认值, 想自定义时也可以手动传入stripe和border
> 这里值得注意的是: stripe和border这些默认字段必须写在\$attrs前面, 否则, 当你手动传入stripe和border时, 将会因为字段被默认属性重写而失效

## 3. 封装表单组件


## 4. 打包并发布开源的npm包