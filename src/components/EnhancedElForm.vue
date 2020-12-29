<template lang="pug">
div
  //- el-form(:model="model")
  div {{rules}}

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
