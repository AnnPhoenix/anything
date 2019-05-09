<template>
    <div class="input-resource">
        <mt-swipe :auto="0" :continuous="false">
            <mt-swipe-item v-for="(group, index) in itemGroups" :key="index">
                <div class="container">
                    <div class="item" v-for="(item, subIndex) in group" :key="subIndex" @click="item.method">
                        <i class="iconfont" :class="item.fontClass"></i>
                        <p>{{item.name}}</p>
                    </div>
                </div>
            </mt-swipe-item>
        </mt-swipe>
    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-02-26 14:26:23 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-04-09 10:32:25
 * @desc inputer.operation
 */
import { mapGetters } from "vuex";
import { CLOUDDISK } from "@/router/routerName";
import Native_cameraService from "@/service/_native/camera";
import Native_fileService from "@/service/_native/file";
import IMConstant from "@/service/im/constant.js";
import NamespaceConstant from '@/service/fileNameSpace/constant.js';

export default {
    props: {
        dialog: Object
    },

    data() {
        return {
        }
    },

    computed: {
        ...mapGetters("im", ["cloudFiles"]),

        items() {
            return [
                {
                    name: '拍摄',
                    fontClass: 'icon-shexiang',
                    method: this.camera
                }, {
                    name: '本地文件',
                    fontClass: 'icon-chuangjianwenjianjia',
                    method: this.nativeFile
                }, {
                    name: '云盘文件',
                    fontClass: 'icon-yunpanwenjian',
                    method: this.cloudFile
                }, {
                    name: '群组文件',
                    fontClass: 'icon-qunzuwenjianjia',
                    method: this.groupFile
                },
                // {
                //     name: '阅后即焚',
                //     fontClass: 'icon-yuehoujifen',
                //     method: this.onceRead
                // }
            ]
        },

        itemGroups() {
            let pageSize = 8,
                arrs = [],
                temp = Object.assign([], this.items);

            while (temp.length > 0) {
                arrs.push(temp.splice(0, pageSize));
            }

            return arrs;
        }
    },

    mounted() {
        // 是否选择了云盘文件, 选择了则上传
        this.cloudFilesHandler();
    },

    methods: {
        /* Cloud File Operations */
        cloudFilesHandler() {
            let sourceFiles = this.cloudFiles();
            sourceFiles.length > 0 && this.yunduService.imService.chatFile(this.dialog, sourceFiles, IMConstant.FILE_MESSAGE_FILETYPE.CLOUD).then(() => {
                this.dialog.scrollvm.scrollToBottom();
            });
        },

        /* Native File Operations */
        nativeFileHandler(filePath, file) {
            let namespaceInstance = null;
            let needInit = false;

            if (this.dialog.isGroup) {
                namespaceInstance = this.yunduService.nameSpaceService.open(NamespaceConstant.NAMESPACE.GROUP_CONTACT);
                needInit = namespaceInstance.getDeepLength() == 0;
                needInit && namespaceInstance.init(this.dialog.opposite);
            } else {
                namespaceInstance = this.yunduService.nameSpaceService.open(NamespaceConstant.NAMESPACE.CONTACT);
                needInit = namespaceInstance.getDeepLength() == 0;
                needInit && namespaceInstance.init();
            }

            // 选择本地文件
            this.$nextTick(() => {
                let dirId = this.yunduService.nameSpaceService.getCurrentDirId();
                let dirStack = this.yunduService.nameSpaceService.getCurrentDirStack();
                let namespace = this.yunduService.nameSpaceService.getCurrentNameSpace();

                // 格式化file文件
                file = Object.assign({}, file, {
                    fileName: file.name,
                    fileLength: file.size,
                    contentType: file.type,
                    isDir: 0,
                    filePath, dirId, dirStack
                })

                // 生成message消息
                this.yunduService.imService.chatFile(this.dialog, [file], IMConstant.FILE_MESSAGE_FILETYPE.NATIVE).then(() => {
                    this.$nextTick(() => {
                        this.dialog.scrollvm.scrollToBottom();
                    })
                });
            })
        },

        /* Operations */
        // 拍照
        camera() {
            Native_cameraService.takePhoto().then(({ filePath, file }) => {
                this.nativeFileHandler(filePath, file);
            });
        },

        // 本地文件
        nativeFile() {
            Native_fileService.open().then(({ file, filePath }) => {
                this.nativeFileHandler(filePath, file);
            }).catch(e => {
                log.error(e);
                this.$toast.error(e || '上传失败')
            });
        },

        // 选择云盘文件
        cloudFile() {
            this.$router.push({
                name: CLOUDDISK.FILE_BROWSER,
                params: {
                    dirDisabled: 0,
                    cb: ["commit", "im/setCloudFiles"]
                }
            })
        },

        // 选择群组文件
        groupFile() {
            this.$router.push({
                name: CLOUDDISK.FILE_BROWSER_GROUPLIST,
                params: {
                    dirDisabled: 0,
                    cb: ["commit", "im/setCloudFiles"]
                }
            })
        },

        // 阅后即焚
        onceRead() {
            // @TODO
        },
    }
}
</script>

<style lang="scss" scoped>
.input-resource {
    height: 548px;
    //   border-top: 1px solid #CECECE;
    padding: 0 20px;
    padding-bottom: 50px;
    .container {
        width: 100%;
        padding: 68px 0;
        display: flex;
        flex-wrap: wrap;
        justify-content: flex-start;
        .item {
            width: 25%;
            margin-bottom: 70px;
            flex-shrink: 0;
            text-align: center;
            .iconfont {
                width: 105px;
                height: 105px;
                line-height: 105px;
                margin-bottom: 18px;
                background: #fff;
                display: inline-block;
                font-size: 80px;
                // color: #878b91;
                // border: 1px solid #878b91;/* no */
                color: $tip_color;
                border: $border; /* no */
                border-radius: 20px;
            }
        }
    }
}
</style>

<style lang="scss">
.input-resource {
    .mint-swipe-indicators {
        .mint-swipe-indicator {
            width: 16px;
            height: 16px;
            opacity: 0.6;
            margin: 0 6px;
            background: $tip_color;
            &.is-active {
                background: $content_color;
            }
        }
    }
}
</style>
