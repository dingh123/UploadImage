# UploadImage
基于ElementUI二次封装的图片上传组件，支持多文件上传，直接使用v-model取值回显

### 使用方式

在 ``script`` 中引用组件 

```javascript
import UploadImage from '@/components/UploadImage/index.vue';
export default {
    components: {UploadImage}
}
```

在 ``template`` 中使用组件

```html
<upload-image ref="uploadImage" v-model="formData.image" :limit="6" :multiple="true" description="建议上传600x600的图片，只能上传jpg/png/gif文件" />
```

如果动态设置回显，需要手动触发一下watchValue方法
```javascript
this.formData.image = res.image;

this.$nextTick(() => {
    this.$refs.uploadImage.watchValue();
});
```

### 属性说明

|属性名        |类型 |默认值  |说明                                           |
|---        |---- |---  |---                                            |
|multiple        |Boolean |false    |是否支持多选                                         |
|data     |Object |-    |上传时附带的额外参数                                       |
|showFileList      |Boolean |true    |是否显示已上传文件列表                                       |
|limit     |Number |-    |最大允许上传个数  |
|name      |String |file    |设定文件域的字段名  |
|accept        |String |image/jpeg, image/png, image/gif|接受上传的文件类型                                        |
|description  |String |-|文件上传描述                                        |
|size        |Number|5 * 1024  |文件大小限制，单位KB，默认5MB                                       |
