<template lang="pug">
div#app
  enhanced-el-form(:model="model" :options="options" :schema="schema")
  div {{model}}

</template>

<script>
import EnhancedElForm from "./components/EnhancedElForm.vue";

export default {
  name: "App",
  components: {
    EnhancedElForm
  },
  data() {
    return {
      model: {
        name: "",
        age: "",
        likes: ""
      },
      schema: [
        {
          type: "input",
          modelKey: "name",
          label: "姓名",
          props: {
            maxlength: 20
          },
          rules: [{ required: true, message: "请输入姓名", trigger: "blur" }]
        },
        {
          type: "input",
          modelKey: "age",
          label: "年龄",
          props: {
            maxlength: 5
          },
          rules: [{ required: true, message: "请输入年龄", trigger: "blur" }]
        },
        {
          type: "select",
          modelKey: "likes",
          label: "爱好",
          props: {
            options: [{ label: "x", value: "xx" }]
          },
          rules: [{ required: true, message: "请选择", trigger: "blur" }]
        }
      ],
      options: {
        inline: true
      }
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
