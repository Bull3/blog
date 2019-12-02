> 本章内容旨在对管理系统的核心组件进行封装, 通常是表格组件和表单组件, 分为以下几块

> * 封装表格组件 (内嵌分页组件)
> * 封装表单组件
> * 打包并发布开源的npm包

> 二次封装的核心思想

> * 尽量少的改动原始组件的使用方法, 降低新组件的使用成本
> * 以数据配置为核心, 根据数据自动渲染模板, 而不是手动编写模板

## 封装表格组件 (内嵌分页组件)

### 1. 封装表格组件
```html
<!-- 期望的组件最终用法 -->
<b-table :columns="columns" :data="tableData">
</b-table>
<!-- columns是表格动态渲染所需的数据, 即表格列数据, 其他属性和事件用法均与el-table相同 -->
```
#### 1.1 基础功能

##### 1.1.1 为了贴近el-table的原始使用方法, 需要使用vue的 [\$attrs](https://cn.vuejs.org/v2/api/#vm-attrs) 和 [\$listeners](https://cn.vuejs.org/v2/api/#vm-listeners) 对属性和事件进行跨组件传递

```html
<el-table v-bind="$attrs" v-on="$listeners">
</el-table>
```

##### 1.1.2 根据数据循环渲染el-table-column

```html
模板:
<el-table v-bind="$attrs" v-on="$listeners">
    <template v-for="column in columns">
        <!-- 没有插槽colSlot时, 直接根据数据渲染模板 -->
        <el-table-column v-if="!column.colSlot" :key="column.id" v-bind="column">
        </el-table-column>
        <!-- 表格列较为复杂或特殊时可启用插槽时, 将直接渲染插槽自定义内容 -->
        <el-table-column v-else :key="column.id" v-bind="column">
            <template slot-scope="scope">
                <slot :name="column.colSlot" v-bind="scope" />
            </template>
        </el-table-column>
    </template>
</el-table>
```

```javascript
数据:
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

#### 1.2 添加默认配置
```html
<el-table v-bind="{ stripe: true, border: true, ...$attrs }" v-on="$listeners">
</el-table>
```
为了提高组件复用性, 设置了一些默认字段
使用es6的扩展运算符 ... 来展开\$attrs, 默认配置和用户传入值就可以完美拼接, 共同作用在模板上
这里值得注意的是: stripe: true, border: true这些默认字段必须写在前面, 若写在后面则将覆盖\$attrs传入的stripe和border

#### 1.3 嵌入分页组件



### 2. 封装表单组件