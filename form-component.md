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
<enhanced-el-form :model="model" :schema="schema" ></enhanced-el-form>
```

这边借鉴[cube-ui的form属性](https://didi.github.io/cube-ui/#/en-US/docs/form)

- model属性，表单数据对象，`{name:'颜酱',age:18}`
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

综上，`enhanced-el-form`组件的大概就出来了。

注意，`rules`属性是`el-form`有的，这里通过`schema`得到

![e-el-form2](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form2.png)

## el-form-item的处理

现在有一个很简单的表单，需要填写`姓名`和`年龄`。

`App.vue`里可以简单写下：

![e-el-form3](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form3.png)

而`enhanced-el-form`内部，没有`schema`的情况下，长这样

```pug
el-form(:model="model" :rules="rules")
  el-form-item(label="姓名" prop="name")
    el-input(v-model="model.name" maxlength="20")
  el-form-item(label="年龄" prop="age")
    el-input(v-model="model.age" maxlength="20")
```

当然，有了`schema`，直接就循环了

```pug
el-form(:model="model" :rules="rules")
  el-form-item(:label="config.label" :prop="config.modelKey" v-for="config in schema" :key="config.modelKey")
    el-input(v-model="model[config.modelKey]" v-bind="config.props")
```

## 匹配除了input之外的元素

表单组件当然不止`input`，查看[官网的侧边栏](https://element.eleme.cn/#/zh-CN/component/radio)发现有以下组件，其中需要配合子组件使用的放在末尾，其他的都可以单独使用。

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

`enhanced-el-form`内部，可以使用`component`匹配不同组件，需要子组件的，单独匹配。
![e-el-form5](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form5.png)

> 注意：有选项的时候，`options`可以是`['上海','北京']`，也可以是`[{label:'上海',value:'shanghai'},{label:'北京',value:'北京'}]`。  
`option项`里没有设定`label`和`value`的时候，默认两者一致。

## 演示实例 - 添加表单项

接下来，通过改写`schema`来显示表单。

将[官方的例子](https://element.eleme.cn/#/zh-CN/component/form#biao-dan-yan-zheng)，用`schema`改写下：
![e-el-form6](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form6.png)

> 注意：多选的时候，`model`里面的相应项的默认值必须是`数组`

![e-el-form7](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form7.png)

## 优化

- `el-form`自身也有很多属性，这里通过简单的`v-bind="$attrs"`，将`enhanced-el-from`上面的属性自动到`el-form`。  
- 同理，`v-on="$listeners"`
- `el-form`上面的方法，稍微麻烦点，通过手动赋值
- 一般提交按钮不需要配置项，直接插入即可，这里增加`slot`
- 部分表单项，需要定制，通过`slotName`属性，表示不参与内部循环

```js
// el-form(ref="elForm" :model="model" :rules="rules" v-bind="$attrs" v-on="$listeners")
  mounted() {
    const methods = [ "validate", "validateField", "resetFields", "clearValidate" ];
    methods.forEach(method => (this[method] = this.$refs.elForm[method]));
  }
```

## 代码

### 代码：组件概况

```vue
<template lang="pug">
  el-form(:model="model" :rules="rules")
</template>
<script>
export default {
  name: "enhanced-el-form",
  props: {
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

### 代码：el-form-item的处理

App.vue

```vue

<template lang="pug">
div#app
  enhanced-el-form(:model="model" :schema="schema")
  div {{model}}
</template>

<script>
import EnhancedElForm from "./components/EnhancedElForm.vue";
export default {
  name: "App",
  components: { EnhancedElForm },
  data() {
    return {
      model: { name: "", age: "" },
      schema: [
        {
          type: "input", modelKey: "name", label: "姓名",
          props: { maxlength: 20 },
          rules: [{ required: true, message: "请输入姓名", trigger: "blur" }]
        },
        {
          type: "input", modelKey: "age", label: "年龄",
          props: { maxlength: 5 },
          rules: [{ required: true, message: "请输入年龄", trigger: "blur" }]
        }
      ]
    };
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>

```

### 代码：除input之外的元素

EnhancedElForm.vue

```vue
<template lang="pug">
el-form(:model="model" :rules="rules")
  el-form-item(v-for="config in schema" :label="config.label" :prop="config.modelKey" :key="config.modelKey")
  
    el-radio-group(v-if="config.type==='radio-group'"   v-model="model[config.modelKey]" v-bind="config.props")
      el-radio(v-for="(item,index) in config.props.options" :key="index" :label="typeof item==='object'?item.value:item") {{ typeof item==='object'?item.label:item }}
      
    el-checkbox-group(v-else-if="config.type==='checkbox-group'"   v-model="model[config.modelKey]" v-bind="config.props")
      el-checkbox(v-for="(item,index) in config.props.options" :key="index" :label="typeof item==='object'?item.value:item") {{ typeof item==='object'?item.label:item }}
      
    el-select(v-else-if="config.type==='select'"   v-model="model[config.modelKey]" v-bind="config.props")
      el-option(v-for="(item,index) in config.props.options" :key="index" :value="typeof item==='object'?item.label:item" :label="typeof item==='object'?item.value:item")

    component(v-else :is="'el-'+config.type" v-model="model[config.modelKey]" v-bind="config.props") {{config.text}}
    
</template>
<script>
export default {
  name: "enhanced-el-form",
  props: {
    model: {
      type: Object,
      default() {
        return {};
      }
    },
    schema: {
      type: Array,
      default() {
        return {};
      }
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

</script>

```

### 代码：演示实例 - 添加表单项