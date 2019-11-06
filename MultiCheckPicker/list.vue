<template>
  <ul class="cube-tree-list">
    <!-- <li
      class="cube-tree-list_li"
    >
      <cube-checkbox
        v-model="checkAll"
        @input="handleCheckAll($event)"
        position="right"
        class="cube-tree-list_checkbox border-bottom-1px"
        >全部</cube-checkbox
      >
    </li> -->
    <li
      class="cube-tree-list_li"
      v-for="(node, nodeIndex) in list"
      :key="guid(node)"
      :class="{ 'cube-tree-list_li-active': activeList[level - 1] === node.id }"
    >
      <cube-checkbox
        @input="handleCheck($event, node)"
        v-model="node.checked"
        position="right"
        :hollowStyle="node.indeterminate"
        :option="{ disabled: node.disabled }"
        class="cube-tree-list_checkbox border-bottom-1px"
        v-if="!onlyLast || (onlyLast && node.isLeaf)"
        >{{ node[labelKey] }}</cube-checkbox
      >

      <div
        class="cube-tree-list_txt"
        :class="{ 'cube-tree-list_txt-checked': node.checked || node.indeterminate }"
        v-else
        @click="handleClick(node, nodeIndex, level)"
      >
        {{ node[labelKey] }}
      </div>
    </li>
  </ul>
</template>

<script>
export default {
  props: {
    activeList: {
      type: Array,
      default: () => []
    },
    list: {
      type: Array,
      default: () => []
    },
    level: {
      type: [Number, String]
    },
    labelKey: {
      type: String,
      default: "label"
    },
    expandTrigger: {
      type: String,
      default: "click"
    },
    onlyLast: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      checkAll: false,
      oldValue: {
        oldNode: null,
        oldLevelIndex: null,
        oldLevel: null
      }
    };
  },
  watch: {
    list: {
      deep: true,
      immediate: true,
      handler: function() {
        this.$forceUpdate();
      }
    }
  },
  methods: {
    handleClick(node, levelIndex, level) {
      this.$emit("handle-click", node, levelIndex, level);
    },
    handleCheck(v, node) {
      node.checked = v;
      this.$emit("handle-check", node);
    },
    handleCheckAll(v) {
      this.$emit("handle-check-all", this.list, v);
    },
    guid() {
      return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(
        c
      ) {
        let r = (Math.random() * 16) | 0;
        let v = c === "x" ? r : (r & 0x3) | 0x8;
        return v.toString(16);
      });
    }
  }
};
</script>
<style lang="scss">
@import "@/styles/functions.scss";
.cube-tree-list {
  margin-bottom: rem(30);
  &_li {
    padding: 0 rem(16);
    &-active {
      background-color: #F9F9F9;
    }
  }
  &_txt {
    padding: rem(11);
    margin-left: rem(-16);
    margin-right: rem(-16);
    line-height: 1.5;
    font-size: rem(14);
     overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    &-checked {
      color: #5276B1;
    }
  }
  &_checkbox {
    padding: 0;
    /deep/ .cube-checkbox-label {
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
  }
}
</style>
