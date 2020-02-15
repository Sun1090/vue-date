<template>
  <div v-click-outside>
    <input type="text" :value="formatDate" class="contain-nav"  @focus="focus" @blur="blur"/>
    <div class="pannel" v-if="isVisible">
      <div class="pannel-nav">
        <span @click="prevMoYear">&lt;</span>
        <span @click="prevMonth">&lt;&lt;</span>
        <span>{{ time.year }}年</span>
        <span>{{ time.month + 1 }}月</span>
        <span @click="nextMonth">&gt;&gt;</span>
        <span @click="nextYear">&gt;</span>
      </div>
      <div class="pannel-content">
        <div class="days">
          <span v-for="j in 7" :key="`_` + j" class="cell">
            {{ weekDays[j - 1] }}
          </span>
          <div v-for="i in 6" :key="i">
            <span
              v-for="j in 7"
              :key="j"
              class="cell cell-days"
              @click="chooseDate(visibleDays[(i - 1) * 7 + (j - 1)])"
              :class="[
                {
                  notCurrentMonth:!isCurrentMonth(
                    visibleDays[(i - 1) * 7 + (j - 1)]
                  )
                },
                { today: isToday(visibleDays[(i - 1) * 7 + (j - 1)]) },
                { select: isSelect(visibleDays[(i - 1) * 7 + (j - 1)]) }
              ]"
            >
              {{ visibleDays[(i - 1) * 7 + (j - 1)].getDate() }}
            </span>
          </div>
        </div>
      </div>
      <div class="pannel-footer">
        今天
      </div>
    </div>
  </div>
</template>
<script>
// import {getYearMonthDay} from './util'
import * as utils from "./util";
export default {
  directives: {
    clickOutside: {
      //指令的生命周期
      bind(el, bindings, vnode) {
        //context
        let handler = e => {
          if (el.contains(e.target)) {
            //判断是否显示面板
            if (!vnode.context.isVisible) {
              vnode.context.focus();
            }
          } else {
            if (vnode.context.isVisible) {
              vnode.context.focus();
            }
          }
        };
        el.handler = handler;
        document.addEventListener("click", handler);
      },
      unbind(el) {
        document.removeEventListener("click", el.handler);
      }
    }
  },
  data() {
    let { year, month } = utils.getYearMonthDay(this.value);
    return {
      weekDays: ["日", "一", "二", "三", "四", "五", "六"],
      time: { year, month },
      isVisible: false //这个变量是用来控制这个面板是否可见
    };
  },
  props: {
    value: {
      type: Date,
      default: () => new Date() //返回的默认值必须是一个函数{}[]
    }
  },
  methods: {
    focus() {
      //显示面板
      this.isVisible = true;
    },
    blur() {
      this.isVisible = false;
    },
    //是否是当月
    isCurrentMonth(date) {
      let { year, month } = utils.getYearMonthDay(
        utils.getDate(this.time.year, this.time.month, 1)
      );
      let { year: y, month: m } = utils.getYearMonthDay(date);
      return year === y && month === m;
    },
    isToday(date) {
      let { year, month, day } = utils.getYearMonthDay(new Date());
      let { year: y, month: m, day: d } = utils.getYearMonthDay(date);
      return year === y && month === m && day === d;
    },
    chooseDate(date) {
      this.time = utils.getYearMonthDay(date)
      this.$emit("input", date)
      this.blur() //关闭碳层
    },
    isSelect(date) {
      let { year, month, day } = utils.getYearMonthDay(this.value);
      let { year: y, month: m, day: d } = utils.getYearMonthDay(date);
      return year === y && month === m && day === d;
    },
    prevMonth() {
      let d = utils.getDate(this.time.year, this.time.month, 1);
      d.setMonth(d.getMonth() - 1);
      this.time = utils.getYearMonthDay(d);
    },
    nextMonth() {
      let d = utils.getDate(this.time.year, this.time.month, 1);
      d.setMonth(d.getMonth() + 1);
      this.time = utils.getYearMonthDay(d);
    },
    prevMoYear() {
      let d = utils.getDate(this.time.year, this.time.month, 1);
      d.setFullYear(d.getFullYear() - 1);
      this.time = utils.getYearMonthDay(d);
    },
    nextYear() {
      let d = utils.getDate(this.time.year, this.time.month, 1);
      d.setFullYear(d.getFullYear() + 1);
      this.time = utils.getYearMonthDay(d);
    }
  },
  computed: {
    visibleDays() {
      //获取当前是周几
      let { year, month } = utils.getYearMonthDay(
        utils.getDate(this.time.year, this.time.month, 1)
      );
      let currentFirstDay = utils.getDate(year, month, 1);
      let week = currentFirstDay.getDay();
      let startDay = currentFirstDay - week * 60 * 60 * 1000 * 24;
      let arr = [];
      for (let i = 0; i < 42; i++) {
        //依次循环出42天
        arr.push(new Date(startDay + i * 60 * 60 * 1000 * 24));
      }
      return arr;
    },
    formatDate() {
      let { year, month, day } = utils.getYearMonthDay(this.value);
      return `${year}-${month + 1}-${day}`;
    }
  }
};
</script>

<style lang="stylus">
.contain-nav
    text-align center
    margin-left 44%
.pannel
    margin-top 20px
    margin-left 35%
    position absolute
    width 60*7px
    background #fff
    box-shadow  2px 2px 2px pink , -2px -2px 2px pink
    .pannel-nav
        display flex
        justify-content space-around
        height  70px
        span
          cursor  pointer
          user-select  none
    .pannel-content
        .cell
            display inline-flex
            justify-content center
            align-items center
            width  60px
            height  60px
            font-weight bold
            box-sizing  border-box
        .cell-days:hover,.select
            border  1px solid pink
     .pannel-footer
        margin-top 10px
        height  50px
        text-align  center
        font-weight bold
        font-size 16px
        color red
.notCurrentMonth
  color gray
.today
  background red
  color #fff
  border-radius  4px

</style>
