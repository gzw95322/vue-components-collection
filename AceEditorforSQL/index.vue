<!--
 * @Description: sql编辑器组件
 * @LastEditors: gzw
 * @Date: 2019-03-15 17:59:05
 * @LastEditTime: 2019-06-27 12:00:17
 -->
<template>
    <div class="editor-container">
      <div class="editor">
        <editor v-model="content" @init="editorInit" :options="options" lang="sql" theme="sqlserver" width="690" height="350" ref="editor"></editor>
      </div>
      <div class="editor-tip">备注：不允许写drop,truncate、update关键词</div>
    </div>
</template>

<script>
import editor from "vue2-ace-editor";

export default {
  name: "SqlEditor",
  props: ['value'],
  data() {
    return {
      content: this.value,
      options: {
        fontSize: 14, // 字体
        enableBasicAutocompletion: true, // 
        enableSnippets: true, // 提示代码块
        enableLiveAutocompletion: true, // 设置自动提示
        wrap: 'free' // 自动换行
      }
    };
  },
  methods: {
    editorInit: function() {
      require("brace/ext/language_tools"); // 语言工具
      require("brace/mode/sql"); //sql 语法
      require("brace/mode/mysql"); // mysql 语法
      require("brace/mode/text"); // 纯文本 语法
      require("brace/theme/sqlserver"); // 主题
      require("brace/snippets/sql"); // sql语法提示
      require("brace/snippets/mysql"); // mysql 语法提示
    }
  },
  watch: {
    // vlaue 绑定实现
    value(newVal) {
      this.content = newVal;
    },
    // 内容不正常内容规避
    content(newVal, oldVal) {
      let reg = /drop|truncate|delete|update/gm;
      this.content = newVal.match(reg) != null ? newVal.replace(reg, '') : newVal;
      this.$emit('input', this.content)
    }
  },
  components: {
    editor
  }
};
</script>

<style lang='less' scoped>
.editor {
  border: 1px solid #D9D9D9;
  width: 690px;
  height: 350px;
}
.editor-tip {
  color: #666666;
  font-size: 14px;
}
</style>
