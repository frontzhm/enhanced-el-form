---
title: 让el-form更好用
tags: js
categories: js
---

`element-ui`虽然有[`el-form`组件](https://element.eleme.cn/#/zh-CN/component/form)，但是仍然需要手动写`el-form-item`。

![e-el-form1](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form1.png)

这里希望进一步抽离配置，在`el-form`的基础上封装个`enhanced-el-form`组件。

使用的时候希望这样，不再需要手动写里面的`el-form-item`：

```html
<enhanced-el-form :model="model" :options="options" :schema="schema" ></enhanced-el-form>
```

这边借鉴[cube-ui的form属性](https://didi.github.io/cube-ui/#/en-US/docs/form)

- model属性，表单数据对象，`{name:'颜酱',age:18}`
- options属性，表单属性配置，`{inline:true}`
- schema属性，每个表单项的配置数组，如下

```js
[
  {
    type: 'input',
    modelKey: 'name',
    label: '姓名',
    props: {
      maxlength: 20
    },
    rules: [
      { required: true, message: '请输入姓名', trigger: 'blur' }
    ]
  }
]
```

## 组件的概况

综上，`enhanced-el-form`组件的大概组成就出来了。

需要注意三点：

<!-- - `model`属性需要更改父组件的数据，为方便，配合`sync`修饰符 -->
- `rules`属性是`el-form`有的，这里通过`schema`得到
- `el-form`本身的其他属性，直接遍历`options`即可

![e-el-form2](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form2.png)

里面的`el-form-item`，如果没有schema的情况下，长这样

```pug
el-form(:model="model" :rules="rules" v-bind="options")
  el-form-item(label="活动名称" prop="name")
    el-input(v-model="model.name" maxlength="20")
  el-form-item(label="年龄" prop="age")
    el-input(v-model="model.age" maxlength="20")
```

当然，有了`schema`，循环就可以用了

```pug
el-form(:model="model" :rules="rules" v-bind="options")
  el-form-item(v-for="config in schema" :label="config.label" :prop="config.modelKey" :key="config.modelKey")
    el-input(v-model="model[config.modelKey]" v-bind="config.props")
```

## 匹配除了input之外的元素

要是只能`input`，也太不实用了，为了随意使用各种类型，需要在组件里各个匹配。

[官网的侧边栏](https://element.eleme.cn/#/zh-CN/component/radio)，看下所有的表单组件：

- radio
- checkbox
- input
- input-number
- cascader
- switch
- slider
- time-select
- date-picker
- rate
- color-picker
- transfer
- radio-group，需要子组件
- checkbox-group，需要子组件
- select，需要子组件
- upload，需要子组件




### 可输入的

- input
- input-number
- 

### 可选择的

- select
- cascader
- switch




radio checkbox 
input 


## 代码

### 组件概况代码

```vue
<template lang="pug">
el-form(:model="model" :rules="rules" v-bind="options")
</template>
<script>
export default {
  name: "enhanced-el-form",
  props: {
    options: {
      type: Object, default() { return {}; } 
    },
    model: {
      type: Object, default() { return {}; }
    },
    schema: {
      type: Array, default() { return {}; }
    }
  },
  computed: {
    rules() {
      return this.schema.reduce((acc, cur) => {
        acc[cur.modelKey] = cur.rules;
        return acc;
      }, {});
    }
  }
};
</script>

```




