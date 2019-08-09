# Excel解析组件

## 使用效果图

![tab-layout](../../_media/layout/codeeditor-img.png)

- **引用**：
```js
import CodeEditor from '@/components/CodeEditor'
```

- **视图**：
```html
<div>
    <Code-Editor :value="jsonValue" @cancel="cancelBtn" @confirm="confirmBtn" />
</div>
```
```javascript
export default {
  components: {
    CodeEditor
  },
  data () {
      return {
        jsonValue: '',      
      }
  },
  methods: {
    cancelBtn () {
    },
    confirmBtn (value) {
    }
  }
}
```

## Attributes

参数 | 说明 | 类型 | 可选值 | 默认值
- | - | - | - | -
value | JOSN数据 | Object/Array/String | ---- | ----

## Slot

参数 | 说明
- | -
---- | 默认tab布局内容

## Events

事件 | 说明 | 回调参数
- | - | -
confirm |  点击保存按钮事件 | value
cancel | 点击取消按钮事件 | ----