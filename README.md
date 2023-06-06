# CronEdit
基于Vben Admin 的 Cron 表达式编辑器

## 使用方式
可以直接在FormSchema中使用
```tsx
import { useComponentRegister } from '/@/components/Form';
import { CronEdit } from '/@/components/CronEdit/index';

useComponentRegister('CronEdit', CronEdit);

export const formSchema: FormSchema[] = [
  {
    field: 'cronExpr',
    label: '表达式',
    required: true,
    component: 'CronEdit',
  },
];
```
可也直接使用， 用法和Input 一样
```
<CronEdit v-model:value="cronexpression">
```
