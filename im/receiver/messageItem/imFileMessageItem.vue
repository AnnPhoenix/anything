<template>
    <v-touch @press="press" class="im-file-message-item" :id="message.messageId">
        <template v-if="!fileTransfer">
            <div class="file-info" @click="clickEventHandler">
                <div class="file-icon-container">
                    <yd-img class="file-icon" :image="getIcon()" :defaultImg="getIconError()" />
                </div>
                <div class="file-container">
                    <h2 class="file-name">{{message.fileName}}</h2>
                    <div class="file-params">
                        <!-- 文件夹不展示filesize -->
                        <span class="file-size" v-if="message.isDir == 0">{{message.fileLengthStr | fileSizeFilter}}</span>
                        <span class="file-size" v-else></span>

                        <!-- 传输状态, message.isSend: true(已发送, 未发送), false(已下载, 未下载) -->
                        <span class="file-transfer-status">{{transferStatus}}</span>
                    </div>
                </div>
            </div>
        </template>
        <template v-else>
            <div class="file-info" @click="clickEventHandler">
                <div class="file-icon-container">
                    <yd-img class="file-icon" :image="getIcon()" :defaultImg="getIconError()" />
                </div>
                <div class="file-container">
                    <h2 class="file-name">{{message.fileName}}</h2>
                    <div class="file-params">
                        <!-- 文件夹不展示filesize -->
                        <span class="file-size" v-if="message.isDir == 0">{{message.fileLengthStr | fileSizeFilter}}</span>
                        <span class="file-size" v-else></span>

                        <!-- 传输状态, message.isSend: true(已发送, 未发送), false(已下载, 未下载) -->
                        <span class="file-transfer-status" v-if="fileTransfer.state != TransferConstant.TRANSFER_STATUS.COMPLETE">{{`${(fileTransfer.getSpeed() / 1024).toFixed(2)}KB/S`}}</span>
                        <span class="file-transfer-status" v-else>{{transferStatus}}</span>
                    </div>
                </div>
            </div>

            <!-- 进度条 -->
            <span class="transfer-progress" v-if="fileTransfer.state != TransferConstant.TRANSFER_STATUS.COMPLETE">
                <vm-progress class="t-progress" :percentage="progress" :show-text="false" :stroke-width=4 strokeColor="red" v-if="fileTransfer.state == TransferConstant.TRANSFER_STATUS.ERROR"></vm-progress>
                <vm-progress class="t-progress" :percentage="progress" :show-text="false" :stroke-width=4 v-else></vm-progress>
            </span>
        </template>

        <!-- download -->
        <im-download ref="download" @success="successCallback"></im-download>
    </v-touch>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2018-09-30 09:12:20 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-04-22 12:04:54
 * @desc 文件消息
 * @todo 也要做任务操作?
 * @constant contentType 文件类型 
 * 'Directory': -1, 'Picture': 0 图片, 'Word': 1, 'Excel': 2, 'PowerPoint': 3, 
 * 'Acrobat': 4 Pdf, 'Video': 5, 'PlainText': 6, 'Audio': 7, 'Unknow': 8
 */
import { mapGetters } from "vuex";
import { CLOUDDISK } from "@/router/routerName";
import { getIconInternet, getIconNative } from "@/util/ico";
import YdImg from '@/components/img/img.vue';
import VmProgress from 'vue-multiple-progress';
import ImDownload from '@/components/_modules/im/download.vue'
import imService from "@/service/_server/im.js"
import IMConstant from "@/service/im/constant.js";
import TransferConstant from "@/service/fileTransfer/constant.js";
import NamespaceConstant from '@/service/fileNameSpace/constant.js';

export default {
    components: { YdImg, VmProgress, ImDownload },

    props: {
        dialog: Object,
        message: Object
    },

    data() {
        return {
            TransferConstant
        }
    },

    computed: {
        ...mapGetters("transfer", ["fileTransferById"]),

        // 本条文件消息对应的transfer()
        fileTransfer() {
            if (this.message.fileTransferId) {
                let fileTransfer = this.fileTransferById(this.message.fileTransferId);
                return fileTransfer;
            }

            return null;
        },

        /**
         * 发送状态(已发送, 未发送, 已下载, 未下载)
         */
        transferStatus() {
            if (this.message.isSendUser) {
                let sendStatus = this.message.sendStatus;
                if (sendStatus == IMConstant.MESSAGE_SEND_STATUS.SUCCESS) {
                    return '已发送'
                } else {
                    return '未发送'
                }
            } else {
                let transferStatus = this.message.transferStatus;
                if (transferStatus == IMConstant.FILE_MESSAGE_TRANSFERSTATUS.NODOWNLOAD) {
                    return '未下载'
                } else {
                    return '已下载'
                }
            }
        },

        progress() {
            return this.fileTransfer.getProgress();
        },

        speed() {
            return `${(this.fileTransfer.getSpeed() / 1024).toFixed(2)}KB/S`
        },
    },

    methods: {
        /* 匹配图标 */
        getIcon() {
            let src = getIconInternet(this.message);

            if (this.message.ext.sourceFile
                && this.message.isSendUser
                && this.message.fileType == IMConstant.FILE_MESSAGE_FILETYPE.NATIVE) {
                src = getIconNative(this.message.ext.sourceFile);
            }

            return src
        },

        getIconError() {
            return getIconInternet(this.message, true)
        },

        /* namespace */
        openIMNamespace() {
            // 打开命名空间
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
        },

        /* imFile */
        getFileInfo() {
            return Promise.resolve(this.message.ext.imfile).then(file => {
                if (!file) {
                    return imService.file.info(this.message.fileId).then(fileInfo => {
                        return fileInfo;
                    })
                }

                return file;
            })
        },

        /* click */
        clickEventHandler() {
            // 判断当前文件消息的传输状态
            let transferStatus = this.message.transferStatus;
            switch (transferStatus) {
                // 上传(未上传), 无操作
                case IMConstant.FILE_MESSAGE_TRANSFERSTATUS.NOUPLOAD:
                    break;

                // 上传(已上传), 打开云盘预览
                case IMConstant.FILE_MESSAGE_TRANSFERSTATUS.UPLOAD:
                    this.openIMNamespace();
                    this.getFileInfo().then(previewItem => {
                        this.$store.commit('SET_PREVIEW_ITEM', previewItem);
                        this.$router.push({
                            name: CLOUDDISK.PREVIEW
                        })
                    });

                    break;

                // 下载(未下载), 则下载
                case IMConstant.FILE_MESSAGE_TRANSFERSTATUS.NODOWNLOAD:
                    this.download();
                    break;

                // 下载(已下载), 打开下载预览
                case IMConstant.FILE_MESSAGE_TRANSFERSTATUS.DOWNLOAD:
                    this.fileTransfer && this.$router.push({
                        name: CLOUDDISK.PREVIEW_DOWNLOAD,
                        params: {
                            fileTransfers: [this.fileTransfer]
                        }
                    })
                    break;
            }
        },

        /* press */
        press(e) {
            let options = [];

            if (this.message.isSendUser) {
                // 发送者
                if (this.message.sendStatus == IMConstant.MESSAGE_SEND_STATUS.SUCCESS) {
                    // 已发送(转存,转发,撤回)
                    options = [{
                        label: "转存",
                        method: this.save,
                    }, {
                        label: "转发",
                        method: this.forward,
                    }, {
                        label: "撤回",
                        method: null
                    }]
                } else {
                    // 未发送或发送失败(无)
                }
            } else {
                // 接收者
                if (this.message.transferStatus == IMConstant.FILE_MESSAGE_TRANSFERSTATUS.NODOWNLOAD) {
                    // 未下载(下载,转存,转发,删除)
                    options = [{
                        label: "下载",
                        method: this.download,
                    }, {
                        label: "转存",
                        method: this.save,
                    }, {
                        label: "转发",
                        method: this.forward,
                    }, {
                        label: "删除",
                        method: this.remove
                    }]
                } else {
                    // 已下载(转存,转发,删除)
                    options = [{
                        label: "转存",
                        method: this.save,
                    }, {
                        label: "转发",
                        method: this.forward,
                    }, {
                        label: "删除",
                        method: this.remove
                    }]
                }
            }

            options.length > 0 && this.$messageMenus(e, options);
        },

        // 下载
        download() {
            this.getFileInfo().then(fileInfo => {
                let params = {
                    hasOpposite: true,
                    opposite: this.dialog.opposite,
                    messageId: this.message.messageId,
                }

                this.$refs.download.execute([fileInfo], params);
            });
        },

        /* download */
        successCallback(fileTransfers) {
            fileTransfers.forEach(fileTransfer => {
                this.message.setFileTransfer(fileTransfer.fileTransferId)
            })
        },

        // 转存
        save() {
            this.openIMNamespace();
            this.getFileInfo().then(imFile => {
                this.$router.push({
                    name: CLOUDDISK.DIR_BROWSER,
                    params: {
                        operateType: 2,
                        operateItems: [imFile]
                    }
                });
            })
        },

        // 转发
        forward() {
            this.getFileInfo().then(file => {
                this.$picker.dialogPicker().then(opposites => {
                    this.yunduService.imService.forwardChatFile(opposites, [file], IMConstant.FILE_MESSAGE_FILETYPE.CLOUD);
                    this.$toast.success("消息已发送")
                });
            })
        },

        // 删除
        remove() {
            this.$confirm("删除后将不会出现在你的聊天记录中").then(() => {
                return this.dialog.deleteMessage(this.message)
            }).then(() => {
                this.$emit("remove")
            })
        },
    }
}
</script>

<style lang="scss" scoped>
.im-file-message-item {
    touch-action: pan-y !important;
    min-width: 400px;
    // width: 460px;
    padding: 20px;
    display: flex;
    flex-direction: column;

    .file-info {
        display: flex;
        margin: 10px 0;

        .file-icon-container {
            width: 100px;
            height: 100px;
            padding: 10px;
            background: $background_color;
            display: flex;
            justify-content: center;
            align-items: center;
            margin-right: 20px;
            img {
                width: 100%;
            }
        }

        .file-container {
            width: 300px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;

            .file-name {
                font-size: 28px;
                padding-bottom: 20px;
                color: $title_color;
                @include omit(280px);
            }

            .file-params {
                display: flex;
                justify-content: space-between;
                font-size: 24px;

                .file-size {
                    color: $tip_color;
                }

                .file-transfer-status {
                    color: $tip_color;
                }
            }
        }
    }
    // 进度条
    .transfer-progress {
        width: 100%;
        margin-top: 10px;
    }
}
</style>
