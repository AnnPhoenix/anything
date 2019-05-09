<template>
    <div class="download">
        <yd-popup :show.sync="popupShow" title="存到本地" @cancel="cancel">
            <div class="popup-content">
                <div class="popup-item">
                    <i class="iconfont icon-xiazai1" />
                    <div class="label" @click="saveToDefault()">
                        <span class="title">另存到:</span>
                        <span>手机/数据云保存的文件</span>
                    </div>
                </div>
                <div class="popup-item" @click="saveToCustom()">
                    <i class="iconfont" />
                    <div class="label">
                        自定义下载位置
                    </div>
                </div>
            </div>
        </yd-popup>

        <c-native-explorer ref="nativeExplorer"></c-native-explorer>
    </div>
</template>

<script>
/**
 * @Author: wangxin
 * @Date: 2018-06-26 19:13:04
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-04-02 23:24:15
 * @desc 下载
 */
import CDownload from '@/components/_modules/common/download.vue';
import TransferConstant from "@/service/fileTransfer/constant.js";

export default {
    extends: CDownload,
    methods: {
        /**
         * 云盘文件下载
         * @param {array} fileItems
         * @param {object} params
         * @param {boolean} params.isVersion
         */
        execute(fileItems, params = {}) {
            this.fileItems = Object.assign([], fileItems);
            this.args.module = TransferConstant.TRANSFER_MODULE.IM;
            this.args.params = params;
            this.popupShow = true;
        }
    }
};
</script>

<style lang="scss" scoped>
.download {
    .popup-content {
        flex-grow: 1;
        display: flex;
        flex-direction: column;

        .popup-item {
            display: flex;
            align-items: center;
            height: 132px;

            .iconfont {
                width: 100px;
                font-size: 64px;
                color: $main_color;
            }

            .label {
                font-size: 30px;
                display: flex;
                flex-direction: column;
                justify-content: space-around;
                span:last-child {
                    margin-top: 20px;
                }
            }

            &:last-child {
                height: 90px;
                border-top: 1px solid #d5d5d5; /* no */
            }
        }
    }
}
</style>