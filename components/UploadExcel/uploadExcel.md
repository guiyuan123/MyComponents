# Excel解析组件

## 使用效果图

- **引用**：
```js
import UploadExcelComponent from '@/components/UploadExcel'
```

- **视图**：
```html
<div>
    <UploadExcelComponent btnName="导入项目计划" :on-success="handleSuccess" :before-upload="beforeUpload" />
</div>
```
```javascript
export default {
  components: {
    UploadExcelComponent
  },
  methods: {
    handleSuccess ({ results, header }) {
    },
    beforeUpload (file) {
    }
  }
}
```

## Attributes

参数 | 说明 | 类型 | 可选值 | 默认值
- | - | - | - | -
btnName | 按钮内容(必填) | String | ---- | ----

## Slot

参数 | 说明
- | -
---- | 默认tab布局内容

## Events

事件 | 说明 | 回调参数
- | - | -
before-upload | 解析前的回调函数,可对文件大小进行逻辑判断处理 | file
on-success | 解析成功返回的数据 | results, header