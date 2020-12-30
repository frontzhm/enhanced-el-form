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
- 一般提交按钮不需要配置项，直接插入即可，这里增加`slot#footer`
- 同理，表单的开始有可能有别的描述，这里增加`slot#header`
- 部分表单项，需要定制，通过`slotName`属性，表示不参与内部循环，需要自己写逻辑
- `date`系列的表单，可能需要多个组件拼接，一般有`children`，这种时候再需要自己定制的基础上，还需要处理`children`的rules

```js
// el-form(ref="elForm" :model="model" :rules="rules" v-bind="$attrs" v-on="$listeners")
  mounted() {
    const methods = [ "validate", "validateField", "resetFields", "clearValidate" ];
    methods.forEach(method => (this[method] = this.$refs.elForm[method]));
  }
```

以上逻辑将在下面的部分展示全部代码。

## 演示实例的优化

实例的效果：

![e-el-form8](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/e-el-form8.gif)

EnhancedElForm的代码如下

```vue
<template lang="pug">
el-form(ref="elForm" :model="model" :rules="rules" v-bind="$attrs" v-on="$listeners")
  slot(name="header")
  
  template(v-for="config in schema" )
    slot(v-if="config.slotName" :name="config.slotName" v-bind="config")
 
    el-form-item(v-else :label="config.label" :prop="config.modelKey" :key="config.modelKey")
      el-radio-group(v-if="config.type==='radio-group'"   v-model="model[config.modelKey]" v-bind="config.props")
        el-radio(v-for="(item,index) in config.props.options" :key="index" :label="typeof item==='object'?item.value:item") {{ typeof item==='object'?item.label:item }}
      el-checkbox-group(v-else-if="config.type==='checkbox-group'"   v-model="model[config.modelKey]" v-bind="config.props")
        el-checkbox(v-for="(item,index) in config.props.options" :key="index" :label="typeof item==='object'?item.value:item") {{ typeof item==='object'?item.label:item }}
      el-select(v-else-if="config.type==='select'"   v-model="model[config.modelKey]" v-bind="config.props")
        el-option(v-for="(item,index) in config.props.options" :key="index" :value="typeof item==='object'?item.value:item" :label="typeof item==='object'?item.label:item")

      component(v-else :is="'el-'+config.type" v-model="model[config.modelKey]" v-bind="config.props") {{config.text}}

  slot(name="footer")
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
        // 日期组件可能有children
        const hasChildren = cur.children && cur.children.length;
        hasChildren &&
          cur.children.forEach(child => (acc[child.modelKey] = child.rules));
        return acc;
      }, {});
    }
  },
  mounted() {
    // el-form上面的方法继承过来
    const methods = [
      "validate",
      "validateField",
      "resetFields",
      "clearValidate"
    ];
    methods.forEach(method => (this[method] = this.$refs.elForm[method]));
  }
};
</script>

```

App.vue的代码如下

```vue
<template lang="pug">
div#app
  enhanced-el-form(ref='ruleForm' :model="model" :schema="schema"  label-width="100px" @validate="validate")
    template(#date="config")
      el-form-item(:label="config.label")
        el-col(:span="11")
          el-form-item(:prop="config.children[0].modelKey")
            el-date-picker(v-model="model[config.children[0].modelKey]" v-bind="config.children[0].props")
        el-col(:span="2") --
        el-col(:span="11")
          el-form-item(:prop="config.children[1].modelKey")
            el-time-picker(v-model="model[config.children[1].modelKey]" v-bind="config.children[1].props")
    template(#footer)
      el-form-item
        el-button(type="primary" @click="submitForm('ruleForm')") 立即创建
        el-button(@click="resetForm('ruleForm')") 重置
  div {{model}}
</template>

<script>
import EnhancedElForm from "./components/EnhancedElForm.vue";
export default {
  name: "App",
  components: { EnhancedElForm },
  data() {
    return {
      model: { type: [] },
      schema: [
        {
          type: "input",
          modelKey: "name",
          label: "活动名称",
          rules: [
            { required: true, message: "请输入活动名称", trigger: "blur" },
            { min: 3, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" }
          ]
        },
        {
          type: "select",
          modelKey: "region",
          label: "活动区域",
          props: {
            placeholder: "请选择活动区域",
            options: [
              { label: "区域一", value: "shanghai" },
              { label: "区域二", value: "beijing" }
            ]
          },
          rules: [
            { required: true, message: "请选择活动区域", trigger: "change" }
          ]
        },
        {
          slotName: "date",
          label: "活动时间",
          children: [
            {
              modelKey: "date1",
              props: {
                type: "date",
                placeholder: "选择日期",
                style: "width:100%"
              },
              rules: [
                {
                  type: "date",
                  required: true,
                  message: "请选择日期",
                  trigger: "change"
                }
              ]
            },
            {
              modelKey: "date2",
              props: {
                placeholder: "选择时间",
                style: "width:100%"
              },
              rules: [
                {
                  type: "date",
                  required: true,
                  message: "请选择时间",
                  trigger: "change"
                }
              ]
            }
          ]
        },
        {
          type: "switch",
          modelKey: "delivery",
          label: "即时配送",
          props: {}
        },

        {
          type: "checkbox-group",
          modelKey: "type",
          label: "活动性质",
          props: {
            options: [
              "美食/餐厅线上活动",
              "地推活动",
              "线下主题活动",
              "单纯品牌曝光"
            ]
          },
          rules: [
            {
              type: "array",
              required: true,
              message: "请至少选择一个活动性质",
              trigger: "change"
            }
          ]
        },

        {
          type: "radio-group",
          modelKey: "resource",
          label: "特殊资源",
          props: {
            options: [
              { label: "线上品牌商赞助", value: "xianshang" },
              { label: "线下场地免费", value: "xianxia" }
            ]
          },
          rules: [
            { required: true, message: "请选择活动资源", trigger: "change" }
          ]
        },
        {
          type: "input",
          modelKey: "desc",
          label: "活动形式",
          props: {
            type: "textarea"
          },
          rules: [
            { required: true, message: "请填写活动形式", trigger: "blur" }
          ]
        }
      ]
    };
  },
  methods: {
    validate(...args) {
      console.log(...args);
    },
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          alert("submit!");
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    }
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

## 不足

为了方便，这里的组件`EnhancedElForm`，直接修改了父组件的`model`。
但这是违反规范的，回头有空再研究下，读者有思路也可以指导下~

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