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
