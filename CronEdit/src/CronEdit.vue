<template>
  <div>
    <InputSearch v-bind="$attrs" :value="contabValueRes" @change="handleChange" @search="openModal">
      <template #enterButton>
        <Button>编辑</Button>
      </template>
    </InputSearch>
    <BasicModal :visible="showModal" @ok="okModal" @cancel="cancelModal" title="表达式">
      <Tabs type="border-card">
        <TabPane key="second" tab="秒" v-if="shouldHide('second')" forceRender>
          <CrontabSecond 
          @update="updateContabValue" 
          :check="checkNumber"
          :resolve="resolveExpByName"
          ref="cronsecond" />
        </TabPane>
        <TabPane key="min" tab="分钟" v-if="shouldHide('min')">
          <CrontabMin
            @update="updateContabValue"
            :check="checkNumber"
            :resolve="resolveExpByName"
            :cron="contabValueObj"
            ref="cronmin"
          />
        </TabPane>

        <TabPane key="hour" tab="小时" v-if="shouldHide('hour')">
          <CrontabHour
            @update="updateContabValue"
            :check="checkNumber"
            :resolve="resolveExpByName"
            :cron="contabValueObj"
            ref="cronhour"
          />
        </TabPane>

        <TabPane key="day" tab="日" v-if="shouldHide('day')">
          <CrontabDay
            @update="updateContabValue"
            :check="checkNumber"
            :resolve="resolveExpByName"
            :cron="contabValueObj"
            ref="cronday"
          />
        </TabPane>

        <TabPane key="mouth" tab="月" v-if="shouldHide('mouth')" forceRender>
          <CrontabMouth
            @update="updateContabValue"
            :check="checkNumber"
            :resolve="resolveExpByName"
            :cron="contabValueObj"
            ref="cronmouth"
          />
        </TabPane>

        <TabPane key="week" tab="周" v-if="shouldHide('week')" forceRender>
          <CrontabWeek
            @update="updateContabValue"
            :check="checkNumber"
            :resolve="resolveExpByName"
            :cron="contabValueObj"
            ref="cronweek"
          />
        </TabPane>

        <TabPane key="year" tab="年" v-if="shouldHide('year')" forceRender>
          <CrontabYear
            @update="updateContabValue"
            :check="checkNumber"
            :resolve="resolveExpByName"
            :cron="contabValueObj"
            ref="cronyear"
          />
        </TabPane>
      </Tabs>

      <div class="popup-main">
        <div class="popup-result">
          <p class="title">时间表达式</p>
          <table>
            <thead>
              <th v-for="item of tabTitles" width="40" :key="item">{{ item }}</th>
              <th>crontab完整表达式</th>
            </thead>
            <tbody>
              <td>
                <span>{{ contabValueObj.second }}</span>
              </td>
              <td>
                <span>{{ contabValueObj.min }}</span>
              </td>
              <td>
                <span>{{ contabValueObj.hour }}</span>
              </td>
              <td>
                <span>{{ contabValueObj.day }}</span>
              </td>
              <td>
                <span>{{ contabValueObj.mouth }}</span>
              </td>
              <td>
                <span>{{ contabValueObj.week }}</span>
              </td>
              <td>
                <span>{{ contabValueObj.year }}</span>
              </td>
              <td>
                <span>{{ contabValueString }}</span>
              </td>
            </tbody>
          </table>
        </div>
        <CrontabResult :ex="contabValueString" />
      </div>
    </BasicModal>
  </div>
</template>

<script>
  import CrontabSecond from './components/CrontabSecond.vue';
  import CrontabMin from './components/CrontabMin.vue';
  import CrontabHour from './components/CrontabHour.vue';
  import CrontabDay from './components/CrontabDay.vue';
  import CrontabMouth from './components/CrontabMouth.vue';
  import CrontabWeek from './components/CrontabWeek.vue';
  import CrontabYear from './components/CrontabYear.vue';
  import CrontabResult from './components/CrontabResult.vue';
  import { BasicModal } from '/@/components/Modal/index';
  import { Input, Tabs, Button } from 'ant-design-vue';

  export default {
    name: 'CronEdit',
    components: {
      CrontabSecond,
      CrontabMin,
      CrontabHour,
      CrontabDay,
      CrontabMouth,
      CrontabWeek,
      CrontabYear,
      CrontabResult,
      Tabs,
      TabPane: Tabs.TabPane,
      InputSearch: Input.Search,
      BasicModal,
      Button,
    },
    props: ['hideComponent', 'value'],
    data() {
      return {
        tabTitles: ['秒', '分钟', '小时', '日', '月', '周', '年'],
        tabActive: 0,
        myindex: 0,
        showModal: false,
        contabValueObj: {
          second: '*',
          min: '*',
          hour: '*',
          day: '*',
          mouth: '*',
          week: '?',
          year: '',
        },
        contabValueRes: '',
      };
    },
    computed: {
      contabValueString: function () {
        let obj = this.contabValueObj;
        return this.getExp(obj);
      },
    },
    watch: {
      value(val) {
        this.contabValueRes = val
        this.resolveExp()
      },
      hideComponent(_) {
        // 隐藏部分组件
      },
      contabValueRes(val) {
        this.$emit('change', val);
      },
    },
    mounted: function () {
      this.resolveExp();
    },
    methods: {
      openModal() {
        this.showModal = true;
      },
      okModal() {
        this.contabValueRes = this.contabValueString;
        this.showModal = false;
      },
      cancelModal() {
        this.showModal = false;
      },
      handleChange(e) {
        this.contabValueRes = e.target.value;
      },
      shouldHide(key) {
        if (this.hideComponent && this.hideComponent.includes(key)) return false;
        return true;
      },
      getExp(obj) {
        let str =
          obj.second +
          ' ' +
          obj.min +
          ' ' +
          obj.hour +
          ' ' +
          obj.day +
          ' ' +
          obj.mouth +
          ' ' +
          obj.week +
          (obj.year == '' ? '' : ' ' + obj.year);
        return str;
      },
      resolveExpByName(name){
        this.changeRadio(name,this.contabValueObj[name])
      },
      resolveExp() {
        //反解析 表达式
        if (this.value) {
          let arr = this.value.split(' ');
          if (arr.length >= 6) {
            //6 位以上是合法表达式
            let obj = {
              second: arr[0],
              min: arr[1],
              hour: arr[2],
              day: arr[3],
              mouth: arr[4],
              week: arr[5],
              year: arr[6] ? arr[6] : '',
            };
            this.contabValueObj = {
              ...obj,
            };
            for (let i in obj) {
              if (obj[i]) this.changeRadio(i, obj[i]);
            }
          }
        } else {
          //没有传入的表达式 则还原
          this.clearCron();
        }
      },
      // tab切换值
      tabCheck(index) {
        this.tabActive = index;
      },
      // 由子组件触发，更改表达式组成的字段值
      updateContabValue(name, value, from) {
        'updateContabValue', name, value, from;
        this.contabValueObj[name] = value;
        if (from && from !== name) {
          console.log(`来自组件 ${from} 改变了 ${name} ${value}`);
          this.changeRadio(name, value);
        }
      },
      //赋值到组件
      changeRadio(name, value) {
        let arr = ['second', 'min', 'hour', 'mouth'],
          refName = 'cron' + name,
          insVlaue;

        if (!this.$refs[refName]) return;

        if (arr.includes(name)) {
          if (value === '*') {
            insVlaue = 1;
          } else if (value.indexOf('-') > -1) {
            let indexArr = value.split('-');
            isNaN(indexArr[0])
              ? (this.$refs[refName].cycle01 = 0)
              : (this.$refs[refName].cycle01 = indexArr[0]);
            this.$refs[refName].cycle02 = indexArr[1];
            insVlaue = 2;
          } else if (value.indexOf('/') > -1) {
            let indexArr = value.split('/');
            isNaN(indexArr[0])
              ? (this.$refs[refName].average01 = 0)
              : (this.$refs[refName].average01 = indexArr[0]);
            this.$refs[refName].average02 = indexArr[1];
            insVlaue = 3;
          } else {
            insVlaue = 4;
            this.$refs[refName].checkboxList = value.split(',');
          }
        } else if (name === 'day') {
          if (value === '*') {
            insVlaue = 1;
          } else if (value === '?') {
            insVlaue = 2;
          } else if (value.indexOf('-') > -1) {
            let indexArr = value.split('-');
            isNaN(indexArr[0])
              ? (this.$refs[refName].cycle01 = 0)
              : (this.$refs[refName].cycle01 = indexArr[0]);
            this.$refs[refName].cycle02 = indexArr[1];
            insVlaue = 3;
          } else if (value.indexOf('/') > -1) {
            let indexArr = value.split('/');
            isNaN(indexArr[0])
              ? (this.$refs[refName].average01 = 0)
              : (this.$refs[refName].average01 = indexArr[0]);
            this.$refs[refName].average02 = indexArr[1];
            insVlaue = 4;
          } else if (value.indexOf('W') > -1) {
            let indexArr = value.split('W');
            isNaN(indexArr[0])
              ? (this.$refs[refName].workday = 0)
              : (this.$refs[refName].workday = indexArr[0]);
            insVlaue = 5;
          } else if (value === 'L') {
            insVlaue = 6;
          } else {
            this.$refs[refName].checkboxList = value.split(',');
            insVlaue = 7;
          }
        } else if (name === 'week') {
          if (value === '*') {
            insVlaue = 1;
          } else if (value === '?') {
            insVlaue = 2;
          } else if (value.indexOf('-') > -1) {
            let indexArr = value.split('-');
            isNaN(indexArr[0])
              ? (this.$refs[refName].cycle01 = 0)
              : (this.$refs[refName].cycle01 = indexArr[0]);
            this.$refs[refName].cycle02 = indexArr[1];
            insVlaue = 3;
          } else if (value.indexOf('#') > -1) {
            let indexArr = value.split('#');
            isNaN(indexArr[0])
              ? (this.$refs[refName].average01 = 1)
              : (this.$refs[refName].average01 = indexArr[0]);
            this.$refs[refName].average02 = indexArr[1];
            insVlaue = 4;
          } else if (value.indexOf('L') > -1) {
            let indexArr = value.split('L');
            isNaN(indexArr[0])
              ? (this.$refs[refName].weekday = 1)
              : (this.$refs[refName].weekday = indexArr[0]);
            insVlaue = 5;
          } else {
            this.$refs[refName].checkboxList = value.split(',');
            insVlaue = 7;
          }
        } else if (name == 'year') {
          if (value == '') {
            insVlaue = 1;
          } else if (value == '*') {
            insVlaue = 2;
          } else if (value.indexOf('-') > -1) {
            insVlaue = 3;
          } else if (value.indexOf('/') > -1) {
            insVlaue = 4;
          } else {
            this.$refs[refName].checkboxList = value.split(',');
            insVlaue = 5;
          }
        }
        this.$refs[refName].radioValue = insVlaue;
      },
      // 表单选项的子组件校验数字格式（通过-props传递）
      checkNumber(value, minLimit, maxLimit) {
        //检查必须为整数
        value = Math.floor(value);
        if (value < minLimit) {
          value = minLimit;
        } else if (value > maxLimit) {
          value = maxLimit;
        }
        return value;
      },
      clearCron() {
        // 还原选择项
        ('准备还原');
        this.contabValueObj = {
          second: '*',
          min: '*',
          hour: '*',
          day: '*',
          mouth: '*',
          week: '?',
          year: '',
        };
        for (let j in this.contabValueObj) {
          this.changeRadio(j, this.contabValueObj[j]);
        }
      },
    },
  };
</script>
<style scoped>
  .pop_btn {
    text-align: center;
    margin-top: 20px;
  }
  .popup-main {
    position: relative;
    margin: 10px auto;
    background: #fff;
    border-radius: 5px;
    font-size: 12px;
    overflow: hidden;
  }
  .popup-title {
    overflow: hidden;
    line-height: 34px;
    padding-top: 6px;
    background: #f2f2f2;
  }
  .popup-result {
    box-sizing: border-box;
    line-height: 24px;
    margin: 25px auto;
    padding: 15px 10px 10px;
    border: 1px solid #ccc;
    position: relative;
  }
  .popup-result .title {
    position: absolute;
    top: -28px;
    left: 50%;
    width: 140px;
    font-size: 14px;
    margin-left: -70px;
    text-align: center;
    line-height: 30px;
    background: #fff;
  }
  .popup-result table {
    text-align: center;
    width: 100%;
    margin: 0 auto;
  }
  .popup-result table span {
    display: block;
    width: 100%;
    font-family: arial;
    line-height: 30px;
    height: 30px;
    white-space: nowrap;
    overflow: hidden;
    border: 1px solid #e8e8e8;
  }
  .popup-result-scroll {
    font-size: 12px;
    line-height: 24px;
    height: 10em;
    overflow-y: auto;
  }
</style>
