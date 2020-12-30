<template lang="pug">
div#app
  enhanced-el-form(:model="model" :schema="schema"  label-width="100px" class="demo-ruleForm")
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
          type: "date-picker",
          modelKey: "date1",
          label: "活动时间",
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
