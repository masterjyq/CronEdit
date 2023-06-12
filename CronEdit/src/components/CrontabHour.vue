<template>
  <RadioGroup v-model:value="radioValue">
    <a-space direction="vertical">
      <Radio :value="1"> 小时，允许的通配符[, - * /] </Radio>

      <Radio :value="2">
        周期从
        <InputNumber v-model:value="cycle01" :min="0" :max="60" />
        -
        <InputNumber v-model:value="cycle02" :min="0" :max="60" />
        小时
      </Radio>

      <Radio :value="3">
        从
        <InputNumber v-model:value="average01" :min="0" :max="60" />
        小时开始，每
        <InputNumber v-model:value="average02" :min="0" :max="60" />
        小时执行一次
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
          :options="[...Array(23)].map((_, i) => ({ value: i }))"
        />
      </div>
    </a-space>
  </RadioGroup>
</template>

<script>
  import { Form, Radio, Input, InputNumber, Select } from 'ant-design-vue';

  export default {
    name: 'CrontabHour',
    components: {
      Form,
      FormItem: Form.Item,
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
        radioValue: 1,
        cycle01: 0,
        cycle02: 1,
        average01: 0,
        average02: 1,
        checkboxList: [],
        checkNum: this.check,
      };
    },
    computed: {
      // 计算两个周期值
      cycleTotal: function () {
        this.cycle01 = this.checkNum(this.cycle01, 0, 23);
        this.cycle02 = this.checkNum(this.cycle02, 0, 23);
        return this.cycle01 + '-' + this.cycle02;
      },
      // 计算平均用到的值
      averageTotal: function () {
        this.average01 = this.checkNum(this.average01, 0, 23);
        this.average02 = this.checkNum(this.average02, 1, 23);
        return this.average01 + '/' + this.average02;
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
      checkboxString: 'checkboxChange',
    },
    mounted: function () {
      this.resolve('hour');
    },
    methods: {
      // 单选按钮值变化时
      radioChange() {
        // if (this.radioValue === 1) {
        //   this.$emit('update', 'hour', '*', 'hour');
        //   this.$emit('update', 'day', '*', 'hour');
        // } else {
        //   if (this.cron.min === '*') {
        //     this.$emit('update', 'min', '0', 'hour');
        //   }
        //   if (this.cron.second === '*') {
        //     this.$emit('update', 'second', '0', 'hour');
        //   }
        // }
        switch (this.radioValue) {
          case 1:
            this.$emit('update', 'hour', '*', 'hour');
            break;
          case 2:
            this.$emit('update', 'hour', this.cycle01 + '-' + this.cycle02);
            break;
          case 3:
            this.$emit('update', 'hour', this.average01 + '/' + this.average02);
            break;
          case 4:
            this.$emit('update', 'hour', this.checkboxString);
            break;
        }
      },
      // 周期两个值变化时
      cycleChange() {
        if (this.radioValue == '2') {
          this.$emit('update', 'hour', this.cycleTotal);
        }
      },
      // 平均两个值变化时
      averageChange() {
        if (this.radioValue == '3') {
          this.$emit('update', 'hour', this.averageTotal);
        }
      },
      // checkbox值变化时
      checkboxChange() {
        if (this.radioValue == '4') {
          this.$emit('update', 'hour', this.checkboxString);
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
