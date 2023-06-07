# CronEdit
基于Vben Admin 的 Cron 表达式编辑器

## 使用方式
将文件夹放置到 `src/components` 目录下
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

## 效果
### 编辑
![image](https://github.com/masterjyq/CronEdit/assets/29848365/62473676-22e8-4e67-a8f9-4cc9ed7a0b29)

### 打开
![image](https://github.com/masterjyq/CronEdit/assets/29848365/7312ef1d-db34-41af-99c4-57d4291ef384)

