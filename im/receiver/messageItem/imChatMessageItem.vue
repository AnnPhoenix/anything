<template>
    <v-touch @press="press($event, message)" class="im-chat-message-item">
        <div class="content" v-html="content"></div>
    </v-touch>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-04-04 09:34:53 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-04-22 12:04:58
 * @desc 聊天消息
 */
import Native_utilService from '@/service/_native/util.js'

export default {
    props: {
        dialog: Object,
        message: Object
    },

    computed: {
        content() {
            return this.message.content
        },
    },

    methods: {
        /* 长按操作 */
        press(e, message) {
            let options = [{
                label: '复制',
                method: this.copy,
                args: [message]
            }, {
                label: '转发',
                method: this.forward,
                args: [message]
            }, {}, {
                label: '删除',
                method: this.delete,
                args: [message]
            }, {
                label: '引用',
                method: this.reference,
                args: [message]
            }];

            if (message.isSendUser) {
                // 复制, 转发, 撤回, 删除, 引用
                options[2] = {
                    label: '撤回',
                    method: this.revoke,
                    args: [message]
                }
            } else {
                // 复制, 转发, 待办, 删除, 引用
                options[2] = {
                    label: '待办',
                    method: this.todo,
                    args: [message]
                }
            }

            this.$messageMenus(e, options);
        },

        copy(message) {
            Native_utilService.copy(this.content);
        },

        forward() {
            this.$picker.dialogPicker().then(opposites => {
                this.yunduService.imService.forwardChat(opposites, this.message);
                this.$toast.success("消息已发送")
            });
        },

        delete() {
            this.$confirm("删除后将不会出现在你的聊天记录中").then(() => {
                return this.dialog.deleteMessage(this.message)
            }).then(() => {
                this.$emit("remove")
            })
        },

        reference() {
            log.info("imChatMessageItem.reference")
        },

        revoke() {
            log.info("imChatMessageItem.revoke")
        },

        todo() {
            log.info("imChatMessageItem.todo")
        },
    }
}
</script>

<style lang="scss">
@import "@/assets/scss/emoji.scss";
.im-chat-message-item {
    touch-action: pan-y !important;
    padding: 14px;

    .content {
        font-size: 26px;
        word-break: break-all;
        line-height: 44px;
        padding: 0 6px;

        * {
            line-height: 44px;
        }

        img {
            width: 100%;
            border-radius: 10px;
            margin-bottom: 10px;
            &:last-child {
                margin-bottom: 0;
            }
        }

        .emoji {
            width: 48px;
            height: 48px;
        }
    }
}
</style>
