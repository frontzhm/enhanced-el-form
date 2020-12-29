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
- options属性，表单属性配置，`{inline:false}`
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




