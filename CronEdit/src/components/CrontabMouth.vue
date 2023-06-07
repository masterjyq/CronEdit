<template>
  <RadioGroup v-model:value="radioValue">
    <a-space direction="vertical">
      <Radio :value="1"> 月，允许的通配符[, - * /] </Radio>

      <Radio :value="2">
        周期从
        <InputNumber v-model:value="cycle01" :min="1" :max="12" /> -
        <InputNumber v-model:value="cycle02" :min="1" :max="12" /> 月
      </Radio>

      <Radio :value="3">
        从
        <InputNumber v-model:value="average01" :min="1" :max="12" /> 月开始，每
        <InputNumber v-model:value="average02" :min="1" :max="12" /> 月月执行一次
      </Radio>

      <div>
        <Radio :value="4" />
        指定
        <Select
          v-model:value="checkboxList"
          placeholder="可多选"
          mode="multiple"
          style="width: 200px"
          @change="checkboxChange"
          :options="[...Array(11)].map((_, i) => ({ value: i + 1 }))"
        />
      </div>
    </a-space>
  </RadioGroup>
</template>

<script>
  import { Form, Radio, InputNumber, Select } from 'ant-design-vue';

  export default {
    name: 'CrontabMouth',
    components: {
      Form,
      FormItem: Form.Item,
      Radio,
      InputNumber,
      Select,
      RadioGroup: Radio.Group,
    },
    props: ['check', 'cron', 'resolve'],
    data() {
      return {
        radioValue: 1,
        cycle01: 1,
        cycle02: 2,
        average01: 1,
        average02: 1,
        checkboxList: [],
        checkNum: this.check,
      };
    },
    computed: {
      // 计算两个周期值
      cycleTotal: function () {
        this.cycle01 = this.checkNum(this.cycle01, 1, 12);
        this.cycle02 = this.checkNum(this.cycle02, 1, 12);
        return this.cycle01 + '-' + this.cycle02;
      },
      // 计算平均用到的值
      averageTotal: function () {
        this.average01 = this.checkNum(this.average01, 1, 12);
        this.average02 = this.checkNum(this.average02, 1, 12);
        return this.average01 + '/' + this.average02;
      },
      // 计算勾选的checkbox值合集
      checkboxString: function () {
        let str = this.checkboxList.join();
        return str === '' ? '*' : str;
      },
    },
    watch: {
      radioValue(a, b) {
        this.radioChange();
      },
      cycleTotal: 'cycleChange',
      averageTotal: 'averageChange',
      checkboxString: 'checkboxChange',
    },
    mounted: function () {
      this.resolve('mouth');
    },
    methods: {
      // 单选按钮值变化时
      radioChange() {
        debugger;
        if (this.radioValue === 1) {
          this.$emit('update', 'mouth', '*');
          this.$emit('update', 'year', '*');
        } else {
          if (this.cron.day === '*') {
            this.$emit('update', 'day', '0', 'mouth');
          }
          if (this.cron.hour === '*') {
            this.$emit('update', 'hour', '0', 'mouth');
          }
          if (this.cron.min === '*') {
            this.$emit('update', 'min', '0', 'mouth');
          }
          if (this.cron.second === '*') {
            this.$emit('update', 'second', '0', 'mouth');
          }
        }
        switch (this.radioValue) {
          case 2:
            this.$emit('update', 'mouth', this.cycle01 + '-' + this.cycle02);
            break;
          case 3:
            this.$emit('update', 'mouth', this.average01 + '/' + this.average02);
            break;
          case 4:
            this.$emit('update', 'mouth', this.checkboxString);
            break;
        }
      },
      // 周期两个值变化时
      cycleChange() {
        if (this.radioValue === 2) {
          this.$emit('update', 'mouth', this.cycleTotal);
        }
      },
      // 平均两个值变化时
      averageChange() {
        if (this.radioValue === 3) {
          this.$emit('update', 'mouth', this.averageTotal);
        }
      },
      // checkbox值变化时
      checkboxChange() {
        if (this.radioValue === 4) {
          this.$emit('update', 'mouth', this.checkboxString);
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
