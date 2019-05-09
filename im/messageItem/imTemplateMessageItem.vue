<template>
    <div class="im-template-message-item">
        <!-- 时间 -->
        <div class="moment">
            {{time | momentFilter}}
        </div>

        <!-- 消息体 -->
        <div class="template-message-container">
            <div class="header">
                <yd-avator class="avator" :user="message.showUser" :name="avatorName" size="1" />
                <div class="name">{{sendUserName}}</div>
                <img :src="TipIcon" class="unread" v-if="message.handleStatus == 0">
            </div>
            <div class="thumb img-container" v-if="thumb">
                <img :src="thumb">
            </div>
            <div class="title">
                <span>[{{moduleName}}]</span>{{title}}
            </div>
            <div class="content">{{content}}</div>
            <div class="operate">
                <span>查看</span>
                <i class="iconfont icon-youjiantou"></i>
            </div>
        </div>

    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-01-24 15:15:07 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-03-05 10:49:44
 * @desc 模板消息 消息模板 (messageType 10000)
 */
import YdAvator from "@/components/avator/avator.vue";
import appFileService from "@/service/_server/app_file.js"

export default {
    components: { YdAvator },

    props: {
        message: Object
    },

    computed: {
        avatorName() {
            if (this.message.data.sendUserName) {
                return this.message.data.sendUserName.split("·")[0]
            }

            return '无'
        },

        sendUserName() {
            if (this.message.data.sendUserName) {
                let name = this.message.data.sendUserName.split("·")[0]
                let dept = this.message.data.sendUserName.split("·")[1]
                return dept ? `${name}·${dept}` : name
            }

            return '无'
        },

        moduleName() {
            return this.message.data.moduleName || 'module'
        },

        title() {
            return this.message.data.title || 'title'
        },

        thumb() {
            let imgUrl = this.message.data.picUrl
            if (imgUrl) {
                let item = {
                    fileName: '',
                    downloadUrl: imgUrl
                };

                let fileURL = appFileService.generateFileURL(item);
                let previewURL = appFileService.generatePreviewURL(item, fileURL);
                return previewURL;
            }

            return null;
        },

        content() {
            return this.message.data.content || '无'
        },

        time() {
            return this.message.data.time || new Date();
        }
    },

    data() {
        return {
            TipIcon: require("@/assets/images/dian.png")
        }
    }
}
</script>

<style lang="scss" scoped>
.im-template-message-item {
    padding: $padding;
    .moment {
        height: 80px;
        line-height: 80px;
        text-align: center;
        font-size: 22px;
        color: #666;
    }

    .template-message-container {
        padding: 20px;
        padding-bottom: 0;
        background: #fff;
        border-radius: $radius;

        .header {
            display: flex;
            align-items: center;
            height: 60px;
            margin-bottom: 20px;

            .avator {
                margin-right: 10px;
            }

            .name {
                flex-grow: 1;
                font-size: 20px;
                color: #333;
            }

            .unread {
                width: 16px;
                height: 16px;
            }
        }

        .thumb {
            margin-bottom: 20px;

            &.img-container {
                width: 100%;
                position: relative;
                padding-bottom: 34.06%;

                > img {
                    position: absolute;
                    width: 100%;
                    height: 100%;
                    left: 0;
                    top: 0;
                }
            }
        }

        .title {
            line-height: 30px;
            margin-bottom: 20px;
            font-size: 28px;
            color: $title_color;
            @include omit(680px);

            > span {
                color: $main_color;
            }
        }

        .content {
            margin-bottom: 20px;
            line-height: 36px;
            font-size: 26px;
            color: $tip_color;

            display: -webkit-box;
            text-overflow: ellipsis;
            -webkit-box-orient: vertical;
            -webkit-line-clamp: 3;
            overflow: hidden;
            word-wrap: break-word;
            font-family: inherit;
        }

        .operate {
            height: 64px;
            border-top: $border;
            font-size: 22px;
            color: $content_color;

            display: flex;
            align-items: center;
            justify-content: space-between;

            .iconfont {
                font-size: 36px;
                color: $tip_color;
            }
        }
    }
}
</style>
