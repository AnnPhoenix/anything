<template>
    <div class="im-receiver-opposite">
        <yd-avator class="avator" :name="user.realName" size="3" />
        <section class="message-container" :class="message.messageType == IMConstant.MESSAGE_TYPE.CHAT ? 'chat' : 'file'">
            <i class="triangle"></i>
            <div class="message" :is="message | compFilter" :dialog="dialog" :message="message" @remove="$emit('remove')"></div>
        </section>
        <div class="message-send-status"></div>
    </div>
</template>

<script>
import YdAvator from "@/components/avator/avator.vue";
import ImChatMessageItem from "@/components/_modules/im/receiver/messageItem/imChatMessageItem.vue";
import ImFileMessageItem from "@/components/_modules/im/receiver/messageItem/imFileMessageItem.vue";
import imGroupDescMessageItem from "@/components/_modules/im/receiver/messageItem/imGroupDescMessageItem.vue";
import IMConstant from "@/service/im/constant.js"

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
            IMConstant
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
                    comp = "imGroupDescMessageItem";
                    break;
            }

            return comp;
        }
    },

}
</script>

<style lang="scss" scoped>
.im-receiver-opposite {
    display: flex;
    align-items: flex-start;
    width: 100%;
    padding: 30px 100px 30px 30px;

    .avator {
        flex-shrink: 0;
        border: 2px solid #fff; /* no */
    }

    .message-container {
        display: flex;
        min-width: 70px;
        min-height: 70px;
        margin: 10px 20px 0px 40px;
        border-radius: 20px;
        position: relative;

        &.chat,
        &.file {
            background: #fff;
            i.triangle {
                border-color: #fff transparent transparent;
            }
        }

        i.triangle {
            position: absolute;
            top: 20px;
            left: -20px;
            display: inline-block;
            width: 0;
            height: 0;
            border-width: 20px 20px 0 20px;
            border-style: solid;
        }
    }

    .message-send-status {
        flex-shrink: 0;
        width: 40px;
        margin-top: 20px;
    }
}
</style>
