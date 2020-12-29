<template lang="pug">
div 1
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
      type: Object,
      default() {
        return {};
      }
    }
  }
};
</script>
