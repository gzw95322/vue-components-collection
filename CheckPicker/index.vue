<!--
 * @Description: 基础多选
 * @Date: 2019-10-22 13:46:55
 * @LastEditors: gzw
 * @LastEditTime: 2019-11-06 13:50:32
 -->

<template>
  <div class="check-picker">
    <div class="picker-zone" @click="show()">
      <span :class="{ 'has-content': checkListTxt.length > 0 }">{{ checkListTxt.join(',') || title }}</span>
      <arrow-toggle :is-top="isVisible" />
    </div>
    <transition name="cube-picker-fade">
      <!-- Transition animation need use with v-show in the same template. -->
      <cube-popup
        type="picker"
        ref="checkPicker"
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
            <div class="cube-picker-content c-picker-content">
              <cube-scroll :data="data" ref="chkScroll">
                <cube-checkbox-group v-model="checkList" :options="data" @input="checkChangeHandler" />
              </cube-scroll>
            </div>
            <div class="cube-picker-footer"></div>
          </div>
        </transition>
      </cube-popup>
    </transition>
  </div>
</template>
<script>
import arrowToggle from "../ArrowToggle";
import popupMixin from "cube-ui/src/common/mixins/popup";
import pickerMixin from "cube-ui/src/common/mixins/picker";

const EVENT_SELECT = "select";
const EVENT_VALUE_CHANGE = "value-change";
const EVENT_CANCEL = "cancel";
const EVENT_CHANGE = "change";

export default {
  name: "CheckPicker",
  components: {
    arrowToggle
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
      default: () => {
        return []
      }
    }
  },
  data() {
    return {
      isVisible: false,
      checkList: this.value,
      oldValues: []
    };
  },
  watch: {
    isVisible(newVal) {
      if (newVal) {
       this.$nextTick(() => {
         this.$refs.chkScroll.refresh();
       });
      }
    },
    value: {
      deep: true,
      immediate: true,
      handler(newVal, oldVal) {
        if (newVal !== oldVal) {
          this.checkList = newVal;
        }
      }
    }
  },
  model: {
    prop: 'value',
    event: 'change'
  },
  computed: {
    checkListTxt() {
      if (this.checkList && this.checkList.length > 0) {
        let _arrTxt = [];
        if (typeof this.data[0] !== 'string') {
          this.data.forEach(item => {
            if (this.checkList.includes(item.value)) _arrTxt.push(item.label);
          });
        } else {
          this.data.forEach(item => {
            if (this.checkList.includes(item)) _arrTxt.push(item);
          });
        }
        return _arrTxt;
      } else {
        return []
      }
    }
  },
  mounted() {
    document.body.appendChild(this.$refs.checkPicker.$el);
  },
  beforeDestroy() {
    document.body.removeChild(this.$refs.checkPicker.$el);
  },
  methods: {
    checkChangeHandler() {
      const _firstVal = this.data[0].value || this.data[0] || '';
      if (_firstVal === '全国' || _firstVal === '全部行业') {
        this.chkListRecheck(_firstVal);
      }
      this.$emit(EVENT_VALUE_CHANGE, this.checkList);
    },
    confirm() {
      this.hide();
      this.$emit(EVENT_CHANGE, this.checkList);
    },
    maskClick() {
      this.maskClosable && this.cancel();
    },
    cancel() {
      this.hide();
      let checkList = [];
      this.data.forEach(item => {
        if (this.oldValues.includes(item.value)) {
          checkList.push(item.value);
        };
      });
      this.checkList = checkList;
      this.$emit(EVENT_CANCEL);
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
    // 特例
    chkListRecheck(str) {
      let { checkList } = this;
      if (checkList.includes(str) && checkList.length > 1) {
        if (checkList.indexOf(str) === 0) {
          checkList.shift();
          this.checkList = checkList;
        }
        if (checkList.indexOf(str) > 0) {
          this.checkList = [str]
        }
      }
    }
  }
};
</script>

<style lang="scss">
@import "@/styles/functions.scss";
.c-picker-content {
  height: 229px;
  top: 0;
   /deep/ .cube-checkbox-wrap {
    flex-direction: row-reverse;
    justify-content: space-between;
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
