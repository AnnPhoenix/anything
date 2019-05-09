<template>
    <v-touch @tap="tap" class="im-notice-group-desc-item">
        <div class="message-info">
            <i class="iconfont icon-tongzhi"></i>
            <div class="content" v-html="content"></div>
        </div>
        <div class="title">通知公告</div>
    </v-touch>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-04-03 15:47:24 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-04-19 10:19:24
 * @desc 通知公告(群)
 */
import { CONTACT } from "@/router/routerName";

export default {
    props: {
        dialog: Object,
        message: Object
    },

    computed: {
        content() {
            return this.message.newDesc
        },
    },

    methods: {
        tap() {
            // 跳转到公告页
            this.$router.push({
                name: CONTACT.GROUP_DESC,
                params: {
                    groupId: this.dialog.opposite
                }
            })
        }
    }
}
</script>

<style lang="scss" scoped>
.im-notice-group-desc-item {
    touch-action: pan-y !important;
    width: 460px;
    display: flex;
    flex-direction: column;

    .message-info {
        display: flex;
        padding: 20px;
        .iconfont {
            flex-shrink: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100px;
            height: 100px;
            color: #fff;
            background: #8abf55;
            font-size: 60px;
        }

        .content {
            padding-left: 20px;
            font-size: 28px;
            line-height: 34px;
            color: $content_color;

            display: -webkit-box;
            text-overflow: ellipsis;
            -webkit-box-orient: vertical;
            -webkit-line-clamp: 3;
            overflow: hidden;
            word-wrap: break-word;
            font-family: inherit;
        }
    }

    .title {
        height: 50px;
        line-height: 50px;
        padding-left: 20px;
        color: $tip_color;
        background: #f8f8f8;
        border-top: $border; /* no */
        border-radius: 0 0 20px 20px;
    }
}
</style>
