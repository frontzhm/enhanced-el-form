<template lang="pug">
el-form(:model="model" :rules="rules" v-bind="options")
  el-form-item(label="活动名称" prop="name")
    el-input(v-model="model.name")
  el-form-item(label="年龄" prop="age")
    el-input(v-model="model.age")

</template>
<script>
export default {
  name: "enhanced-el-form",
  props: {
    options: {
      type: Object,
      default() {
        return {};
      }
    },
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
