<template>
  <RadioGroup v-model:value="radioValue">
    <a-space direction="vertical">
      <Radio :value="1"> 周，允许的通配符[, - * / L #] </Radio>

      <Radio :value="2"> 不指定 </Radio>

      <Radio :value="3">
        周期从星期
        <InputNumber v-model:value="cycle01" :min="1" :max="7" /> -
        <InputNumber v-model:value="cycle02" :min="1" :max="7" />
      </Radio>

      <Radio :value="4">
        第
        <InputNumber v-model:value="average01" :min="1" :max="4" /> 周的星期
        <InputNumber v-model:value="average02" :min="1" :max="7" />
      </Radio>

      <Radio :value="5">
        本月最后一个星期
        <InputNumber v-model:value="weekday" :min="1" :max="7" />
      </Radio>

      <div>
        <Radio :value="6" />
        指定
        <Select
          v-model:value="checkboxList"
          placeholder="可多选"
          mode="multiple"
          style="width: 200px"
          @change="checkboxChange"
        >
          <SelectOption v-for="(item, index) of weekList" :key="index" :value="index + 1">{{
            item
          }}</SelectOption>
        </Select>
      </div>
    </a-space>
  </RadioGroup>
</template>

<script>
  import { Radio, Input, InputNumber, Select } from 'ant-design-vue';

  export default {
    name: 'CrontabWeek',
    components: {
      Radio,
      Input,
      InputNumber,
      Select,
      SelectOption: Select.Option,
      RadioGroup: Radio.Group,
    },
    props: ['check', 'cron', 'resolve'],
    data() {
      return {
        radioValue: 2,
        weekday: 1,
        cycle01: 1,
        cycle02: 2,
        average01: 1,
        average02: 1,
        checkboxList: [],
        weekList: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
        checkNum: this.check,
      };
    },
    computed: {
      // 计算两个周期值
      cycleTotal: function () {
        this.cycle01 = this.checkNum(this.cycle01, 1, 7);
        this.cycle02 = this.checkNum(this.cycle02, 1, 7);
        return this.cycle01 + '-' + this.cycle02;
      },
      // 计算平均用到的值
      averageTotal: function () {
        this.average01 = this.checkNum(this.average01, 1, 4);
        this.average02 = this.checkNum(this.average02, 1, 7);
        return this.average01 + '#' + this.average02;
      },
      // 最近的工作日（格式）
      weekdayCheck: function () {
        this.weekday = this.checkNum(this.weekday, 1, 7);
        return this.weekday;
      },
      // 计算勾选的checkbox值合集
      checkboxString: function () {
        let str = this.checkboxList.join();
        return str == '' ? '*' : str;
      },
    },
    watch: {
      radioValue: 'radioChange',
      cycleTotal: 'cycleChange',
      averageTotal: 'averageChange',
      weekdayCheck: 'weekdayChange',
      checkboxString: 'checkboxChange',
    },
    mounted: function () {
      this.resolve('week');
    },
    methods: {
      // 单选按钮值变化时
      radioChange() {
        if (this.radioValue === 1) {
          this.$emit('update', 'week', '*');
          this.$emit('update', 'year', '*');
        } else {
          if (this.cron.mouth === '*') {
            this.$emit('update', 'mouth', '0', 'week');
          }
          if (this.cron.day === '*') {
            this.$emit('update', 'day', '0', 'week');
          }
          if (this.cron.hour === '*') {
            this.$emit('update', 'hour', '0', 'week');
          }
          if (this.cron.min === '*') {
            this.$emit('update', 'min', '0', 'week');
          }
          if (this.cron.second === '*') {
            this.$emit('update', 'second', '0', 'week');
          }
        }
        switch (this.radioValue) {
          case 2:
            this.$emit('update', 'week', '?');
            break;
          case 3:
            this.$emit('update', 'week', this.cycle01 + '-' + this.cycle02);
            break;
          case 4:
            this.$emit('update', 'week', this.average01 + '#' + this.average02);
            break;
          case 5:
            this.$emit('update', 'week', this.weekday + 'L');
            break;
          case 6:
            this.$emit('update', 'week', this.checkboxString);
            break;
        }
      },
      // 根据互斥事件，更改radio的值

      // 周期两个值变化时
      cycleChange() {
        if (this.radioValue == '3') {
          this.$emit('update', 'week', this.cycleTotal);
        }
      },
      // 平均两个值变化时
      averageChange() {
        if (this.radioValue == '4') {
          this.$emit('update', 'week', this.averageTotal);
        }
      },
      // 最近工作日值变化时
      weekdayChange() {
        if (this.radioValue == '5') {
          this.$emit('update', 'week', this.weekday + 'L');
        }
      },
      // checkbox值变化时
      checkboxChange() {
        if (this.radioValue == '6') {
          this.$emit('update', 'week', this.checkboxString);
        }
      },
    },
  };
</script>
<style scoped>
  .ant-input-number {
    width: 80px;
    max-width: 100px;
  }
</style>
