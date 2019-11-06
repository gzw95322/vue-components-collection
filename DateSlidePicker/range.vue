<!-- 日期搜索框 -->
<template>
  <div class="date-range-container">
    <dateSlidePicker
      class="date-picker"
      :size="size"
      :select-range="selectRange"
      placeholder="开始日期"
      v-model="beginTime" />
    <dateSlidePicker
      class="date-picker"
      :size="size"
      :select-range="selectRange"
      placeholder="结束日期"
      v-model="endTime" />
  </div>
</template>

<script>
import dateSlidePicker from './index'
import config from '@/config'

export default {
  props: {
    value: Array,
    size: {
      type: String,
      default: 'small'
    },
    selectRange: {
      type: Array,
      default: () => config.dateRange
    }
  },
  components: {
    dateSlidePicker
  },
  data() {
    return {
      beginTime: '',
      endTime: ''
    };
  },
  watch: {
    value(newVal) {
      this.beginTime = !!newVal ? (newVal[0] || '') : '';
      this.endTime = !!newVal ? (newVal[1] || '') : '';
    },
    beginTime(newVal, oldVal) {
      const _beginTime = new Date(newVal.replace(/-/g,'/'));
      const _endTime = this.endTime ? new Date(this.endTime.replace(/-/g,'/')) : '';
      if (!_endTime) {
        return;
      }
      if (_beginTime > _endTime) {
         this.$message({
          message: '结束时间不早于开始时间!',
          type: 'error'
        });
        this.beginTime = '';
        return;
      }
      this.emitDateRange();
    },
    endTime(newVal) {
      const _beginTime = this.beginTime ? new Date(this.beginTime.replace(/-/g,'/')) : '';
      const _endTime = new Date(newVal.replace(/-/g,'/'));
      if (!_beginTime) {
        return;
      }
      if (_beginTime > _endTime) {
         this.$message({
          message: '结束时间不早于开始时间!',
          type: 'error'
        });
        this.endTime = '';
        return;
      }
      this.emitDateRange();
    }
  },
  model: {
    prop: 'value',
    event: 'updateVal'
  },
  methods: {
    emitDateRange() {
      const { beginTime, endTime } = this;

      this.$emit('updateVal', [beginTime, endTime]);
      // this.$emit('change', [beginTime, endTime]);
    }
  }
};
</script>
<style lang="scss" scoped>
.date-range-container {
  display: flex;
  justify-content: space-between;
}
  .date-picker {
    width: 48.5%;
  }
</style>
