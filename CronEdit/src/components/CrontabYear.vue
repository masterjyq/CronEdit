<template>
  <RadioGroup v-model:value="radioValue">
    <a-space direction="vertical">
      <Radio :value="1"> 不填，允许的通配符[, - * /] </Radio>

      <Radio :value="2"> 每年 </Radio>

      <Radio :value="3">
        周期从
        <InputNumber v-model:value="cycle01" :min="fullYear" /> -
        <InputNumber v-model:value="cycle02" :min="fullYear" />
      </Radio>

      <Radio :value="4">
        从
        <InputNumber v-model:value="average01" :min="fullYear" /> 年开始，每
        <InputNumber v-model:value="average02" :min="1" /> 年执行一次
      </Radio>

      <div>
        <Radio :value="5" />
        指定
        <Select
          v-model:value="checkboxList"
          placeholder="可多选"
          mode="multiple"
          style="width: 200px"
          @change="checkboxChange"
          :options="[...Array(8)].map((_, i) => ({ value: i + fullYear }))"
        />
      </div>
    </a-space>
  </RadioGroup>
</template>

<script>
  import { Radio, Input, InputNumber, Select } from 'ant-design-vue';

  export default {
    name: 'CrontabYear',
    components: {
      Radio,
      Input,
      InputNumber,
      Select,
      SelectOption: Select.Option,
      RadioGroup: Radio.Group,
    },
    props: ['check', 'mouth', 'cron', 'resolve'],
    data() {
      return {
        fullYear: 0,
        radioValue: 1,
        cycle01: 0,
        cycle02: 0,
        average01: 0,
        average02: 1,
        checkboxList: [],
        checkNum: this.check,
      };
    },
    computed: {
      // 计算两个周期值
      cycleTotal: function () {
        this.cycle01 = this.checkNum(this.cycle01, this.fullYear, this.fullYear + 100);
        this.cycle02 = this.checkNum(this.cycle02, this.fullYear + 1, this.fullYear + 101);
        return this.cycle01 + '-' + this.cycle02;
      },
      // 计算平均用到的值
      averageTotal: function () {
        this.average01 = this.checkNum(this.average01, this.fullYear, this.fullYear + 100);
        this.average02 = this.checkNum(this.average02, 1, 10);
        return this.average01 + '/' + this.average02;
      },
      // 计算勾选的checkbox值合集
      checkboxString: function () {
        let str = this.checkboxList.join();
        return str;
      },
    },
    watch: {
      radioValue: 'radioChange',
      cycleTotal: 'cycleChange',
      averageTotal: 'averageChange',
      checkboxString: 'checkboxChange',
    },
    mounted: function () {
      // 仅获取当前年份
      this.fullYear = Number(new Date().getFullYear());
      this.resolve('year');
    },
    methods: {
      // 单选按钮值变化时
      radioChange() {
        // if (this.cron.mouth === '*') {
        //   this.$emit('update', 'mouth', '0', 'year');
        // }
        // if (this.cron.day === '*') {
        //   this.$emit('update', 'day', '0', 'year');
        // }
        // if (this.cron.hour === '*') {
        //   this.$emit('update', 'hour', '0', 'year');
        // }
        // if (this.cron.min === '*') {
        //   this.$emit('update', 'min', '0', 'year');
        // }
        // if (this.cron.second === '*') {
        //   this.$emit('update', 'second', '0', 'year');
        // }
        switch (this.radioValue) {
          case 1:
            this.$emit('update', 'year', '');
            break;
          case 2:
            this.$emit('update', 'year', '*');
            break;
          case 3:
            this.$emit('update', 'year', this.cycle01 + '-' + this.cycle02);
            break;
          case 4:
            this.$emit('update', 'year', this.average01 + '/' + this.average02);
            break;
          case 5:
            this.$emit('update', 'year', this.checkboxString);
            break;
        }
      },
      // 周期两个值变化时
      cycleChange() {
        if (this.radioValue == '3') {
          this.$emit('update', 'year', this.cycleTotal);
        }
      },
      // 平均两个值变化时
      averageChange() {
        if (this.radioValue == '4') {
          this.$emit('update', 'year', this.averageTotal);
        }
      },
      // checkbox值变化时
      checkboxChange() {
        if (this.radioValue == '5') {
          this.$emit('update', 'year', this.checkboxString);
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
