<template>
    <div>
        <el-upload
            ref="elUpload"
            action="#"
            list-type="picture-card"
            :data="data"
            :multiple="multiple"
            :limit="limit"
            :fileList="fileList"
            :show-file-list="showFileList"
            :name="name"
            :accept="accept"
            :before-upload="beforeUpload"
            :on-preview="handlePictureCardPreview"
            :on-success="handleImageSuccess"
            :on-remove="handleRemove"
            :on-exceed="handleExceed"
            class="upload-image"
            :class="{ 'hide-upload': hideUpload }"
            :http-request="uploadImgRequest"
        >
            <div class="upload-tip">
                <i class="el-icon-plus"></i>
                <div class="el-upload__tip" v-if="description">
                    {{ description }}
                </div>
            </div>
        </el-upload>

        <el-image-viewer v-if="showViewer" :on-close="closeViewer" :url-list="viewImageUrl" />
    </div>
</template>
<script>
import { uploadImg } from '@/api/file';
import emitter from 'element-ui/src/mixins/emitter';
import ElImageViewer from 'element-ui/packages/image/src/image-viewer';
export default {
    name: 'UploadImage',
    mixins: [emitter],
    components: {
        ElImageViewer,
    },
    props: {
        value: {
            type: String,
            default: '',
        },
        // 是否支持多选
        multiple: {
            type: Boolean,
            default: false,
        },
        // 上传时附带的额外参数
        data: {
            type: Object,
        },
        // 是否显示已上传文件列表
        showFileList: {
            type: Boolean,
            default: true,
        },
        // 最大允许上传个数
        limit: {
            type: Number,
        },
        // 设定文件域的字段名
        name: {
            type: String,
            default: 'file',
        },
        // 接受上传的文件类型
        accept: {
            type: String,
            default: 'image/jpeg, image/png, image/gif',
        },
        // 文件上传描述
        description: {
            type: String,
            default: '',
            // default: '请上传图片，只能上传jpg/png/gif文件',
        },
        validateEvent: {
            type: Boolean,
            default: true,
        },
        // 文件大小限制，单位KB，默认5MB
        size: {
            type: Number,
            default: 5 * 1024,
        },
    },
    data() {
        return {
            tempUrl: '',
            hideUpload: false,
            fileList: [],
            showViewer: false,
            viewImageUrl: [],
        };
    },
    inject: {
        elForm: {
            default: '',
        },
        elFormItem: {
            default: '',
        },
    },
    created() {
        this.watchValue();
    },
    computed: {
        // width() {
        //     return `${this.province}${this.city}${this.area}`;
        // },
    },
    watch: {
        value(val) {
            // this.watchValue();
            // console.log(val);
        },
    },
    methods: {
        watchValue() {
            console.log(this.value);
            let valueArray = this.value ? this.value.split(',') : [];
            this.fileList = valueArray.map(item => {
                return {
                    name: item,
                    url: item,
                };
            });
            this.update();
        },
        uploadImgRequest(f) {
            let param = new FormData(); //创建form对象
            param.append(this.name, f.file); //通过append向form对象添加数据
            uploadImg(param)
                .then(res => {
                    f.onSuccess(res);
                })
                .catch(() => {
                    f.onError();
                });
        },
        update() {
            let valueArray = this.value ? this.value.split(',') : [];
            if (valueArray.length >= this.limit) {
                this.hideUpload = true;
            } else {
                this.hideUpload = false;
            }
        },
        emitInput(value) {
            this.$emit('input', value);
            this.dispatch('ElFormItem', 'el.form.change', value);
            this.$nextTick(() => {
                this.update();
            });
            // if (this.$parent.$options.name === 'ElFormItem') {
            //     if (this.validateEvent) {
            //         this.$parent.$emit('el.form.change', [value]);
            //     }
            // }
        },
        beforeUpload(file) {
            const fileMax = file.size / 1024 < this.size;

            if (!fileMax) {
                this.$message.error(`上传图片大小不能超过 ${this.renderSize(this.size / 1024)}!`);
            }
            return !!fileMax;
        },
        handlePictureCardPreview(file) {
            this.viewImageUrl = [file.url];
            this.showViewer = true;
        },
        closeViewer() {
            this.showViewer = false;
        },
        handleRemove(file, fileList) {
            // console.log('Remove', file, fileList);
            const fileArray = fileList.map(item => {
                return item.response.url;
            });
            this.emitInput(fileArray.join(','));
        },
        handleImageSuccess(res, file, fileList) {
            // console.log('Success', res, file, file);
            this.emitInput(this.value ? this.value + ',' + res.url : res.url);
        },
        handleExceed() {
            this.$message.error(`最多上传${this.limit}张图片`);
        },
        clearFiles() {
            this.$refs.elUpload.clearFiles();
        },
        renderSize(value) {
            if (null == value || value == '') {
                return '0 Bytes';
            }
            let unitArr = new Array('Bytes', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB');
            let index = 0;
            let srcsize = parseFloat(value);
            index = Math.floor(Math.log(srcsize) / Math.log(1024));
            let size = srcsize / Math.pow(1024, index);
            size = size.toFixed(2); //保留的小数位数
            return size + unitArr[index];
        },
    },
};
</script>

<style lang="scss" scoped>
.hide-upload /deep/ .el-upload--picture-card {
    display: none;
}
.upload-tip {
    height: 100%;
    line-height: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    padding: 10px;
}
.el-upload__tip {
    line-height: 20px;
    margin-top: 10px;
}
</style>
