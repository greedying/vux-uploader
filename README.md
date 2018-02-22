# vux-uploader

## vux-uploader是什么

vux-uploader是一个vue的上传组件，是对[vux](https://github.com/airyland/vux)组件库的一个补充。  
因为vux并没有对[weui](https://github.com/weui/weui)的[uploader组件](https://weui.io/#uploader)进行封装，理由见[vux issue 682](https://github.com/airyland/vux/issues/682)，但这又是一个常见需求。在使用vux过程中，作者实现了这个组件，现整理开源出来，命名为vux-uploader。

## vux-uploader的特点

+ 基于vux，适合vux项目
+ 暂时不支持vux `$t`方式的多语言功能
+ 增加了删除按钮，点击则删除第一张图片
+ 内置图片上传、增加、删除功能，但暂时每次只能操作一张图片
+ 接上，允许用户监听事件，自己实现上传、增加、删除功能
+ 使用[axios](https://github.com/mzabriskie/axios)进行图片上传


## 快速使用

1. 检查项目是否符合使用条件
  * 需要是使用`vux2`的项目
  * 需要安装`babel`对`ES6`部分语法进行转码
  * 需要安装`less-loader`正确编译less源码
  * 只支持`webpack`的方式，暂不提供`umd`版

2. 安装`vux-uploader`
```javascript
npm install vux-uploader --save
```
3. 使用`vux-uploader`
```javascript
// 引入组件
import Uploader from 'vux-uploader'
```
```javascript
// 子组件
export default {
  ...
  components: {
    ...
    Uploader,
    ...
  }
  ...
}
```

```javascript
  // 使用组件
  <uploader
    :max="varmax"
    :images="images"
    :handle-click="true"
    :show-header="false"
    :readonly="true"
    :upload-url="uploadUrl"
    name="img"
    :params="params"
    size="small"
    @preview="previewMethod"
    @add-image="addImageMethod"
    @remove-image="removeImageMethod"
  ></uploader>
```

### props说明

* images
  * `类型`: Array
  * `默认值`: []，空数组
  * `含义`: 图片数组，格式为 `[ { url: '/url/of/img.ong' }, ...]` 
  * `备注`: 变量为数组时，数据流为双向，在组件内部改变数组的值影响父组件

* max
  * `类型`: Number
  * `默认值`: 1
  * `含义`: 图片最大张数
  * `备注`: 图片达到max值时，新增按钮消失

* showHeader
  * `类型`: Boolean
  * `默认值`: true
  * `含义`: 是否显示头部
  * `备注`: 控制整个头部的显示

* title
  * `类型`: String 
  * `默认值`: 图片上传
  * `含义`: 头部的标题
  * `备注`: 不显示则留空

* showTip
  * `类型`: Boolean
  * `默认值`: true
  * `含义`: 是否显示头部数字部分，即`1/3`部分
  * `备注`: 当`showHeader`为`false`时，header整体隐藏，此时本参数无效

* readonly
  * `类型`: Boolean
  * `默认值`: false
  * `含义`: 是否只读
  * `备注`: 只读时，新增和删除按钮隐藏

* handleClick
  * `类型`: Boolean
  * `默认值`: false
  * `含义`: 是否接管新增按钮的点击事件，如果是，点击新增按钮不再自动出现选择图片界面
  * `备注`:  `true`时，点击新增按钮，则`$emit('add-image')`，可以在此事件内进行自定义的选择图片等操作,`此后图片的新增、上传、删除都由使用者接管`

* autoUpload
  * `类型`: Boolean
  * `默认值`: true
  * `含义`: 选择图片后是否自动上传。是，则调用内部上传接口。否，则`$emit('upload-image', formData)', `formData`为图片内容，用户可监听事件自己上传
  * `备注`: handleClick为`true`时，无法进行图片选择，故此参数无效。此参数为`false`时，选择图片后,`$emit('upload-image', formData)', `formData`为图片内容

* uploadUrl
  * `类型`: String
  * `默认值`: ''
  * `含义`: 接收上传图片的url
  * `备注`: 需要返回如下格式的json字符串，否则请设置`autoUpload`为`false`自行上传

```javascript
  { 
    result: 0,
    message: "result不是0时候的错误信息",
    data: {
      url: "http://image.url.com/image.png"
    }
  }
```
* name
  * `类型`: String
  * `默认值`: `img`
  * `含义`: 默认上传的时候，图片使用的表单name
  * `备注`:  无

* params
  * `类型`: Object
  * `默认值`: null
  * `含义`: 上传文件时携带参数
  * `备注`: 无

* size
  * `类型`: String
  * `默认值`: normal
  * `含义`: 尺寸类型
  * `备注`: `normal`为`weui`默认尺寸，`small`为作者定义的小一些的尺寸

* capture
  * `类型`: String
  * `默认值`: ''
  * `含义`: input 的capture属性
  * `备注`: 可以设置为`camera`，此时点击新增按钮，部分机型会直接调起相机，注意，各型号手机表现不同，请谨慎使用。`handleClick`为true时，此属性无效

### emit事件说明

* add-image
  * `emit时机`: 当`handleClick`参数为`true`时，点击新增按钮
  * `参数`: 无
  * `备注`: 无

* remove-image
  * `emit时机`: 当`handleClick`参数为`true`时，点击删除按钮
  * `参数`: 无
  * `备注`: 当`handleClick`为`false`时，点击删除按钮，组件内部删除第一张图片；否则，用户需要监听本事件，并进行相应删除操作

* preview
  * `emit时机`: 点击任意一张图片的时候
  * `参数`: 图片对象，格式为 `{ url: 'imgUrl' }`
  * `备注`: 暂时没有实现自动预览功能，如果需要用户监听事件自行实现

* upload-image
  * `emit时机`: `handleClick和`autoUpload`都为`false`时，选择图片
  * `参数`: formData,图片内容生成的formData
  * `备注`: 可以通过`vm.$refs.input`获取图片的input元素


## 待实现事项，欢迎PR

- [ ] 上传进度实时显示
- [ ] 上传错误时，图片显示错误样式
- [ ] 一次选择，多图片上传
- [ ] 其他未实现事项
- [ ] formData 中 'img' 可以配置
- [x] 上传图片时，附带post参数[+16](https://github.com/greedying/vux-uploader/pull/16)

## 感谢与参考

+ [vux](https://github.com/airyland/vux)
+ [weui](https://github.com/weui/weui)
+ [axios](https://github.com/mzabriskie/axios)
+ [webpack-simple template](https://github.com/vuejs-templates/webpack-simple)

## 疑问与讨论

+ 请先到[issue](https://github.com/greedying/vux-uploader/issues)进行搜索
+ 如果上面没有答案，欢迎提[issue](https://github.com/greedying/vux-uploader/issues/new)
+ 欢迎加qq群交流，[点此加入](//shang.qq.com/wpa/qunwpa?idkey=b23dade3b260202233283989212dca63bcbb3af8621e850e021cdcb9726d95e2")
