<!--
 * @Description: 公司搜索页
 * @Date: 2019-10-22 13:46:55
 * @LastEditors: gzw
 * @LastEditTime: 2019-11-06 17:34:10
 -->

<template>
  <transition name="page-move">
    <div class="search-container" v-show="isVisible" ref="searchPage">
      <div class="input-control">
        <form @submit="startSearch">
          <cube-input
            v-model="companyName"
            type="search"
            ref="searchCompany"
            @input="query"
            @blur="inputBlurFix"
            placeholder="请输入查询企业"
          >
            <i class="el-icon-search" slot="prepend" role="button"></i>
            <i class="el-icon-error" slot="append" role="button"></i>
          </cube-input>
        </form>
        <span @click="cancel">取消</span>
      </div>
      <div class="company-list">
        <!-- <no-data :visible="companyList.length === 0" /> -->
        <cube-scroll :data="companyList" nest-mode="native">
          <div
            class="company-list_item border-bottom-1px"
            v-for="item in companyList"
            :key="item.companyName"
            v-html="item.highlight"
            @click="selectCompany(item.companyName)"
          ></div>
        </cube-scroll>
      </div>
    </div>
  </transition>
</template>
<script>
import inputFixMixin from "../../mixins/inputFixMixin.js";
import { initData } from "@/api";

const EVENT_TOGGLE = "toggle";
const EVENT_CHANGE = "change";
var timer = null;

export default {
  name: "SearchPage",
  props: {
    isVisible: Boolean,
    value: {
      type: String,
      default: ''
    },
    reqUrl: {
      type: String,
      required: true
    }
  },
  mixins: [inputFixMixin],
  data() {
    return {
      autofocus: false,
      companyName: "",
      companyList: [],
      loading: false
    };
  },
  watch: {
    isVisible(newval) {
      if (newval) {
        this.companyName = '';
        this.oldValue = this.value;
        document.body.appendChild(this.$refs.searchPage);
        setTimeout(() => {
          this.$refs.searchCompany.focus();
        }, 400);
      }
    }
  },
  model: {
    prop: "value",
    event: "change"
  },
  methods: {
    cancel() {
      this.companyName = this.oldValue;
      this.hide();
    },
    confirm(val) {
      this.oldValue = '';
      this.$emit(EVENT_CHANGE, val || this.companyName);
      this.hide();
    },
    hide() {
      this.$emit(EVENT_TOGGLE, false);
    },
    startSearch(e) {
      e.preventDefault();
      this.confirm();
    },
    selectCompany(val) {
      if (!val) return;
      this.confirm(val);
    },
    query() {
      this.getCompany();
    },
    nameHighLight(text) {
      if (!this.companyName) return text;
      const result = text.replace(
        new RegExp(this.companyName, "g"),
        `<span style="color: #5276B1;">${this.companyName}</span>`
      );
      return result;
    },
    getCompany() {
      if (!this.companyName) return;
      this.companyList = [];
      this.loading = true;
      if (timer) clearTimeout(timer);
      timer = setTimeout(() => {
        const { companyName, nameHighLight, reqUrl } = this;
        initData(reqUrl, { companyName }).then(res => {
          this.loading = false;
          if(!res) return;
          let _arr = [];
          if ( typeof res[0] === 'string') {
            _arr = res.map(item => {
              return {
                companyName: item
              }
            });
          } else {
            _arr = res;
          }
          this.companyList = _arr.map(item => {
            return {
              companyName: item.companyName,
              highlight: nameHighLight(item.companyName)
            }
          });
        });
      }, 700);
    }
  }
};
</script>

<style lang="scss" scoped>
@import "@/styles/functions.scss";
.search-container {
  position: fixed;
  z-index: 799;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #f5f7fa;
}
.input-control {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: rem(16);
  background-color: #ffffff;
  span {
    height: rem(21);
    font-size: rem(15);
    font-weight: 400;
    color: rgba(51, 51, 51, 1);
    line-height: rem(21);
    margin-left: rem(15);
  }
  /deep/ .cube-input-append {
    color: #dadada;
    padding-right: rem(16);
  }
  /deep/ .cube-input-prepend {
    color: #999999;
    padding-left: rem(16);
  }
  .el-icon-search {
    font-size: rem(15);
    font-weight: bold;
  }
  form {
    flex: 1;
  }
  .cube-input {
    border-radius: rem(17);
    background: rgba(246, 246, 246, 1);
    &::after {
      border: 0;
    }
  }
  /deep/ .cube-input-field {
    padding: rem(6.5) rem(10);
  }
}
.company-list {
  height: 90.5vh;
  background-color: #ffffff;
  &_item {
    font-size: rem(14);
    font-weight: 400;
    color: rgba(51, 51, 51, 1);
    line-height: rem(20);
    padding: rem(13) rem(16)
  }
}
</style>
