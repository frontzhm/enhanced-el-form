<template lang="pug">
el-form(:model="model" :rules="rules" v-bind="$attrs")
  el-form-item(v-for="config in schemaHandled" :label="config.label" :prop="config.modelKey" :key="config.modelKey")
    component(:is="'el-'+config.type" v-model="model[config.modelKey]" v-bind="config.props") {{config.text}}
      template(v-if="config.type==='radio-group'" )
        el-radio(v-for="item in config.props.options" :key="item.value" :label="item.value") {{ item.label }}
      template(v-if="config.type==='checkbox-group'" )
        el-checkbox(v-for="item in config.props.options" :key="item.value" :label="item.value") {{ item.label }}
      template(v-if="config.type==='select'" )
        el-option(v-for="item in config.props.options" :key="item.value" :value="item.value" :label="item.label")
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
    schemaHandled() {
      // - radio-group checkbox-group select upload，需要子组件
      const singleComponents = [
        "radio",
        "checkbox",
        "input",
        "input-number",
        "cascader",
        "switch",
        "slider",
        "time-select",
        "date-picker",
        "rate",
        "color-picker",
        "transfer"
      ];
      this.schema.forEach(item => {
        const isSingleComponent = singleComponents.includes(item.type);
        item.isSingleComponent = isSingleComponent;
      });
      return this.schema;
    },
    rules() {
      return this.schema.reduce((acc, cur) => {
        acc[cur.modelKey] = cur.rules;
        return acc;
      }, {});
    }
  }
};
</script>
