<template>
    <div class="im-receiver-messagelist" @scroll="scrollEventHandler">
        <mt-loadmore :top-method="!allLoaded && loadmore || null" ref="loadmore" topPullText="查看更多消息" topDropText="查看更多消息" topLoadingText="查看更多消息">
            <div v-for="(message, index) in dialog.messageStore.messages" :key="`message_${message.messageId}`" :id="`message_${message.messageSeq}`">
                <div class="datetime" v-if="dateTimeShow(index)">{{message.updateTime | momentFilter}}</div>
                <div :is="message | compFilter" :dialog="dialog" :message="message"></div>
            </div>
        </mt-loadmore>
    </div>
</template>

<script>
import ImReceiverMe from "@/components/_modules/im/receiver/im-receiver-me.vue";
import ImReceiverOpposite from "@/components/_modules/im/receiver/im-receiver-opposite.vue";
import ImGroupNoticeMessageItem from "@/components/_modules/im/receiver/messageItem/imGroupNoticeMessageItem.vue";
import IMConstant from "@/service/im/constant.js"
import IM_dialogService from '@/service/im/dialog/dialogService.js';

export default {
    components: {
        ImReceiverMe,
        ImReceiverOpposite,
        ImGroupNoticeMessageItem
    },

    props: {
        dialog: Object
    },

    data() {
        return {
            allLoaded: false
        }
    },

    filters: {
        compFilter(message) {
            let comp = "";
            let messageType = message.messageType;

            switch (messageType) {
                // 1-聊天
                case IMConstant.MESSAGE_TYPE.CHAT:
                // 2-发送文件
                case IMConstant.MESSAGE_TYPE.SEND_FILE:
                // 110-改群公告 [聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_CHANGE_DESC:
                    comp = message.isSendUser ? "ImReceiverMe" : "ImReceiverOpposite";
                    break;

                // 101-新建群组 [系统][聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_CREATE:
                // 102-添加管理员 [系统][聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_ADD_MANAGER:
                // 103-删除管理员-这个逻辑较为复杂，目前没有此消息的发送 [系统][聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_REMOVE_MANAGER:
                // 104-转让群组 [系统][聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_CHANGE_OWNER:
                // 106-群组新进人员 [聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_ADD_MEMBER:
                // 107-群组删除人员 [系统][聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_REMOVE_MEMBER:
                // 108-群成员主动离群 [聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_MEMBER_LEAVE:
                // 109-改群名称 [系统] [聊天]
                case IMConstant.MESSAGE_TYPE.GROUP_CHANGE_NAME:
                    comp = "ImGroupNoticeMessageItem"
                    break;
            }

            return comp;
        }
    },

    mounted() {
        this.dialog.registeScroll(this);
        this.scrollToBottom();
    },

    beforeDestroy() {
        this.dialog.destoryScroll();
    },

    methods: {
        /* Scroll Methods */

        // scroll.event.handler
        scrollEventHandler() {
            // 当前滚动条是否在最底端 (+5 修正是否触底)
            let currentOnBottom = this.$el.scrollHeight <= (this.$el.scrollTop + this.$el.offsetHeight) + 5;
            this.dialog.updateCurrentOnBottom(currentOnBottom);

            // 判断是否未读消息已在视野内
            this.dialog.unreadCount > 0 && this.unreadIsLoaded();

            // 群聊?
            if (this.dialog.opposite >= IMConstant.FROM_ID_COND.CHAT && this.dialog.opposite < IMConstant.FROM_ID_COND.GROUP_CHAT) {
                this.dialog.unreadAtMessages.length > 0 && this.unreadAtIsLoaded();
                this.dialog.unreadNoticeMessages.length > 0 && this.unreadNoticeIsLoaded();
            }
        },

        // 滚动到指定seq, 未读, at, 群公告等
        scrollToSeq(seq) {
            let messageItemDOM = this.$el.querySelector(`#message_${seq}`);
            messageItemDOM && messageItemDOM.scrollIntoViewIfNeeded();
        },

        // 是否滚动到未读(seq:0时有bug)
        unreadIsLoaded() {
            let seq = this.dialog.lastReadSequence;

            // scroll top
            let scrollTop = this.$el.scrollTop;

            // offset top
            let messageItemDOM = this.$el.querySelector(`#message_${seq}`);
            let offsetTop = messageItemDOM.offsetTop;

            // compare && read
            if (scrollTop <= offsetTop) {
                // @TODO messageItem特效
                IM_dialogService.read(this.dialog);
            }
        },

        // 是否滚动到未读at
        unreadAtIsLoaded() {
            // scroll top
            let scrollTop = this.$el.scrollTop;

            // messages
            let unreadAtMessages = this.dialog.unreadAtMessages;
            let seqIndexMapper = unreadAtMessages.map((message, index) => {
                return {
                    seq: message.messageSeq,
                    index
                }
            });

            let hasReadIndexs = [];
            seqIndexMapper.forEach(item => {
                let seq = item.seq;

                // offset top
                let messageItemDOM = this.$el.querySelector(`#message_${seq}`);
                let offsetTop = messageItemDOM.offsetTop;

                if (scrollTop <= offsetTop) {
                    hasReadIndexs.push(item.index);
                }
            });

            this.dialog.unreadAtMessages = this.dialog.unreadAtMessages.filter((message, index) => {
                return hasReadIndexs.indexOf(index) == -1;
            })
        },

        // 是否滚动到未读群通知
        unreadNoticeIsLoaded() { },

        // 滚动 Bottom
        scrollToBottom() {
            setTimeout(() => {
                this.$nextTick(() => {
                    this.$el.scrollTop = this.$el.scrollHeight + 1000;
                })
            }, 100)
        },

        // 滚动条固定, 加载消息时使用
        scrollToFixed(preScrollHeight) {
            this.$nextTick(() => {
                let currentScrollHeight = this.$el.scrollHeight;
                this.$el.scrollTop = this.$el.scrollTop + currentScrollHeight - preScrollHeight - 100;
            })
        },

        // Loadmore
        loadmore(callback) {
            let preScrollHeight = this.$el.scrollHeight;

            return this.dialog.loadmore().then(allLoaded => {
                // scroll fixed
                this.scrollToFixed(preScrollHeight);

                // update params
                this.allLoaded = allLoaded;
                this.$refs.loadmore.onTopLoaded();
            })
        },

        // 间隔5分钟一展示
        dateTimeShow(index) {
            if (index > 0) {
                let messages = this.dialog.messageStore.messages;
                let current = messages[index];
                let last = messages[index - 1];

                let currentTime = new Date(Date.parse(current.updateTime)).getTime()
                let lastTime = new Date(Date.parse(last.updateTime)).getTime()
                let cond = 5 * 60 * 1000;

                return currentTime - lastTime > cond
            }

            return false;
        },
    }
}
</script>

<style lang="scss" scoped>
.im-receiver-messagelist {
    flex-grow: 1;
    overflow: scroll;

    .datetime {
        padding: 20px 0;
        text-align: center;
    }
}
</style>
