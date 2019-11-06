<!--
 * @Description: 基础多选
 * @Date: 2019-10-22 13:46:55
 * @LastEditors: gzw
 * @LastEditTime: 2019-11-06 13:51:29
 -->

<template>
  <div class="check-picker">
    <div class="picker-zone" @click="show()">
      <span :class="{ 'has-content': selectedLabels.length > 0 }">{{ selectedLabels.join(',') || title }}</span>
      <arrow-toggle :is-top="isVisible" />
    </div>
    <transition name="cube-picker-fade">
      <!-- Transition animation need use with v-show in the same template. -->
      <cube-popup
        type="picker"
        ref="checkMutliPicker"
        :mask="true"
        :center="false"
        :z-index="zIndex"
        v-show="isVisible"
        @touchmove.prevent
        @mask-click="maskClick"
      >
        <transition name="cube-picker-move">
          <div
            class="cube-picker-panel cube-safe-area-pb"
            v-show="isVisible"
            @click.stop
          >
            <div class="cube-picker-choose border-bottom-1px">
              <span class="cube-picker-cancel" @click="cancel">{{
                _cancelTxt
              }}</span>
              <span class="cube-picker-confirm" @click="confirm">{{
                _confirmTxt
              }}</span>
              <div class="cube-picker-title-group">
                <h1 class="cube-picker-title" v-html="title"></h1>
                <h2
                  v-if="subtitle"
                  class="cube-picker-subtitle"
                  v-html="subtitle"
                ></h2>
              </div>
            </div>
            <div class="cube-picker-content cm-picker-content" @click.stop>
              <div class="cm-picker-content_item cm-picker-content_item-first">
                <cube-scroll ref="firstScroll">
                  <render-list
                    :only-last="onlyLast"
                    :list="root.childNodes"
                    :level="1"
                    :active-list="activeList"
                    @handle-click="handleClick"
                    @handle-check="handleCheck"
                    :label-key="labelKey" />
                </cube-scroll>
              </div>
            <template v-for="item in maxLevellist">
              <div
                :class="`cm-picker-content_item cm-picker-content_item-${item.id}`"
                :key="item.id"
                v-if="item.rendered"
                v-show="activeList.length >= item.id"
              >
                <cube-scroll :ref="`childScroll_${item.id}`">
                  <render-list
                    :only-last="onlyLast"
                    :list="showData[item.id]"
                    :level="item.id + 1"
                    :active-list="activeList"
                    @handle-click="handleClick"
                    @handle-check="handleCheck"
                    @handle-check-all="handleCheckAll"
                    :label-key="labelKey" />
                </cube-scroll>
              </div>
            </template>
            </div>
            <div class="cube-picker-footer"></div>
          </div>
        </transition>
      </cube-popup>
    </transition>
  </div>
</template>
<script>
import TreeStore from '@/components/MultiCascader/lib/Tree.js'
import renderList from './list.vue'
import _ from 'lodash'
import { _findByObj } from '@/components/MultiCascader/tool/unit'

import arrowToggle from "../ArrowToggle";
import popupMixin from "cube-ui/src/common/mixins/popup";
import pickerMixin from "cube-ui/src/common/mixins/picker";

const EVENT_SELECT = "select";
const EVENT_VALUE_CHANGE = "value-change";
const EVENT_CANCEL = "cancel";
const EVENT_CHANGE = "change";

export default {
  name: "MultiCheckPicker",
  components: {
    arrowToggle,
    renderList
  },
  mixins: [popupMixin, pickerMixin],
  props: {
    cancelTxt: {
      type: String,
      default: "取消"
    },
    confirmTxt: {
      type: String,
      default: "完成"
    },
    data: {
      type: Array,
      default() {
        return []
      }
    },
    value: {
      type: Array,
      default: () => []
    },
    valueKey: {
      type: String,
      default: 'value'
    },
    labelKey: {
      type: String,
      default: 'label'
    },
    childrenKey: {
      type: String,
      default: 'children'
    },
    isTwoDimensionValue: {
      type: Boolean,
      default: false
    },
  },
  data() {
    return {
      onlyLast: true,
      isVisible: false,
      selectedLabels: [],
      selectedIds: [],
      selectedNodes: [],
      store: {},
      root: [],
      maxLevellist: [],
      showData: {},
      activeList: [],
      oldValue: []
    };
  },
  watch: {
    isVisible(newVal) {
      if (newVal) {
        document.body.appendChild(this.$refs.checkMutliPicker.$el);
        this.$nextTick(() => {
          this.$refs['firstScroll'].refresh();
        });
      }
    },
    data: {
      deep: true,
      handler () {
        this.init()
      }
    },
    value: {
      deep: true,
      handler () {
        // this.init()
        this.updateSelect(this.value, true, true)
      }
    },
    selectedNodes () {
      let _val = {
        value: this.selectedNodes.map(o => o[this.isTwoDimensionValue ? '_idArr' : this.valueKey]),
        nodes: this.selectedNodes
      };
      this.$emit(EVENT_VALUE_CHANGE, _val)
    }
  },
  model: {
    prop: 'value',
    event: 'change'
  },
  mounted () {
    this.init();
    document.body.appendChild(this.$refs.checkMutliPicker.$el);
  },
  beforeDestroy() {
    document.body.removeChild(this.$refs.checkMutliPicker.$el);
  },
  methods: {
    confirm() {
      this.$emit(EVENT_SELECT);
      this.hide();
    },
    maskClick() {
      this.maskClosable && this.cancel();
    },
    cancel() {
      this.$emit(EVENT_CANCEL);
      this.$emit(EVENT_CHANGE, this.oldValues);
      this.hide();
    },
    show() {
      if (this.isVisible) {
        return;
      }
      setTimeout(() => {
        this.oldValues = JSON.parse(JSON.stringify(this.value));
        this.isVisible = true;
      }, 100);
    },
    hide() {
      if (!this.isVisible) {
        return;
      }
      this.isVisible = false;
    },
    handleClick (node, levelIndex, level) {
      if (this.maxLevellist[level - 1]) {
        this.maxLevellist[level - 1].rendered = true
      }
      let tempList = [...this.activeList]
      if (level < tempList.length) {
        tempList.splice(level)
      }
      tempList[level - 1] = node.id
      this.showData[level] = node.childNodes
      this.activeList = tempList
      let _refName = "childScroll_" + level;
      setTimeout(() => {
        this.$refs[_refName][0].refresh();
      }, 100);
    },
    handleCheck (node) {
      node.check(node.checked)
      this.selectedIds = this.store.selectedIds
      this.updateSelect(this.store.selectedIds)
      this.$emit(EVENT_CHANGE, this.selectedNodes.map(o => o[this.isTwoDimensionValue ? '_idArr' : this.valueKey]))
    },
    handleCheckAll (parent, checked) {

    },
    updateSelect (data = [], needCheckNode = false, setValue = false) {
      let tempSelectedNodes = []
      let tempSelectedLabels = []
      let tempSelectedIds = []
      this.store.nodeList.forEach(node => {
        node.checked && node.check(false)
      })
      data.forEach(o => {
        let targetNode
        if (setValue) {
          targetNode = _findByObj(this.store.nodeList, { [this.isTwoDimensionValue ? '_idArr' : this.valueKey]: o }) || {}
          // targetNode = _.find(this.store.nodeList, { [this.isTwoDimensionValue ? '_idArr' : this.valueKey]: o }) || {}
          tempSelectedIds.push(targetNode.id)
        } else {
          targetNode = this.store.nodesMap[o]
          tempSelectedIds.push(o)
        }
        if (targetNode) {
          needCheckNode && Object.keys(targetNode).length > 0 && targetNode.check(true)
          tempSelectedNodes.push(targetNode)
          tempSelectedLabels.push(targetNode.showLabel)
        }
      })
      this.selectedNodes = tempSelectedNodes
      this.selectedLabels = tempSelectedLabels
      this.selectedIds = tempSelectedIds
    },
    init () {
      this.store = new TreeStore({
        data: this.data,
        separator: '-',
        valueKey: this.valueKey,
        labelKey: this.labelKey,
        childrenKey: this.childrenKey,
        showLeafLabel: true
      })
      this.root = this.store.root
      this.maxLevellist = Array.from({ length: this.store.maxLevel - 1 }, (v, i) => {
        this.showData[i + 1] = []
        return {
          id: i + 1,
          rendered: false
        }
      })
      this.updateSelect(this.value, true, true)
    }
  }
};
</script>

<style lang="scss">
@import "@/styles/functions.scss";
.check-picker {
  /deep/ .cube-checkbox-wrap {
    flex-direction: row-reverse;
    justify-content: space-between;
  }
}
.cm-picker-content {
  height: rem(229);
  top: 0;
  display: flex;
  background-color: #F9F9F9;
  &_item {
    flex: 1;
    background-color: #F9F9F9;
    &-first {
      width: rem(87);
      flex: unset;
      background-color: #fff;
    }
  }

}
.picker-zone {
  display: flex;
  align-items: center;
  justify-content: space-between;
  span {
    font-size: rem(14);
    font-weight: 400;
    color: rgba(102,102,102,1);
    overflow: hidden;
    height: rem(20);
    max-width: rem(75);
    line-height: rem(20);
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    margin-right: rem(5);
    &.has-content {
      color: rgba(51, 51, 51, 1);
    }
  }
}
</style>
