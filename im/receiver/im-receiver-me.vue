<template>
    <div class="im-receiver-me">
        <div class="message-send-status">
            <svg class="fail icon" aria-hidden="true" v-if="message.sendStatus == IMConstant.MESSAGE_SEND_STATUS.FAIL" @click="resend">
                <use xlink:href="#icon-tanhao"></use>
            </svg>
            <!-- <div class="loading" v-if="message.sendStatus == IMConstant.MESSAGE_SEND_STATUS.READY || message.sendStatus == IMConstant.MESSAGE_SEND_STATUS.SENDING"> -->
            <div class="loading" v-if="message.sendStatus == IMConstant.MESSAGE_SEND_STATUS.SENDING">
                <img :src="LoadingIcon" alt="loading">
            </div>
        </div>
        <section class="message-container" :class="message.messageType == IMConstant.MESSAGE_TYPE.CHAT ? 'chat' : 'file'">
            <div :is="message | compFilter" :dialog="dialog" :message="message" @remove="$emit('remove')"></div>
            <i class="triangle"></i>
        </section>
        <yd-avator class="avator" :name="user.realName" size="3" />
    </div>
</template>

<script>
import YdAvator from "@/components/avator/avator.vue";
import ImChatMessageItem from "@/components/_modules/im/receiver/messageItem/imChatMessageItem.vue";
import ImFileMessageItem from "@/components/_modules/im/receiver/messageItem/imFileMessageItem.vue";
import imGroupDescMessageItem from "@/components/_modules/im/receiver/messageItem/imGroupDescMessageItem.vue";
import IMConstant from "@/service/im/constant.js";

export default {
    components: {
        YdAvator,
        ImChatMessageItem, ImFileMessageItem, imGroupDescMessageItem,
    },

    props: {
        dialog: Object,
        message: Object
    },

    computed: {
        user() {
            return this.message.sendUser;
        },
    },

    data() {
        return {
            IMConstant,
            LoadingIcon: require("@/assets/images/loading.gif")
        }
    },

    filters: {
        compFilter(message) {
            let comp = "";
            let messageType = message.messageType;

            switch (messageType) {
                // 1-聊天
                case IMConstant.MESSAGE_TYPE.CHAT:
                    comp = "ImChatMessageItem";
                    break;

                // 2-发送文件
                case IMConstant.MESSAGE_TYPE.SEND_FILE:
                    comp = "ImFileMessageItem";
                    break;

                // 110-改群公告 [聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_CHANGE_DESC:
                    comp = "imGroupDescMessageItem"
                    break;
            }

            return comp;
        }
    },

    methods: {
        resend(){
            let messageType = this.message.messageType;
            switch (messageType) {
                // 1-聊天
                case IMConstant.MESSAGE_TYPE.CHAT:
                    this.yunduService.imService.reChat(this.dialog, this.message).then(() => {
                        this.dialog.scrollToBottom();
                    })
                    break;

                // 2-发送文件
                case IMConstant.MESSAGE_TYPE.SEND_FILE:
                    this.yunduService.imService.reChatFile(this.dialog, this.message).then(() => {
                        this.dialog.scrollToBottom();
                    })
                    break;
            }
        }
    }
}
</script>

<style lang="scss" scoped>
.im-receiver-me {
    display: flex;
    align-items: flex-start;
    justify-content: flex-end;
    width: 100%;
    padding: 30px 30px 30px 100px;

    .message-send-status {
        flex-shrink: 0;
        width: 40px;
        margin-top: 20px;

        > .fail {
            width: 50px;
            height: 50px;
        }

        > .loading {
            width: 50px;
            height: 50px;
            padding: 6px;
            img {
                width: 100%;
                height: auto;
            }
        }
    }

    .message-container {
        display: flex;
        min-width: 70px;
        min-height: 70px;
        margin: 10px 40px 0px 20px;
        border-radius: 20px;
        position: relative;

        &.chat {
            color: #fff;
            background: #36adff;
            i.triangle {
                border-color: #36adff transparent transparent;
            }
        }

        &.file {
            background: #fff;
            i.triangle {
                border-color: #fff transparent transparent;
            }
        }

        i.triangle {
            position: absolute;
            top: 20px;
            right: -20px;
            display: inline-block;
            width: 0;
            height: 0;
            border-width: 20px 20px 0 20px;
            border-style: solid;
        }
    }

    .avator {
        flex-shrink: 0;
        border: 2px solid #fff; /* no */
    }
}
</style>
