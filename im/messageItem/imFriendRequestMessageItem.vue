<template>
    <div class="im-friend-request-message-item">
        <yd-avator class="avator" :user="showUser" :name="showUser.realName" size=3 />
        <div class="main-content">
            <div class="full-name">{{showUser.fullName}}</div>
            <div class="content">{{message.content}}</div>
        </div>
        <div class="buttons" v-if="message.handleStatus == IMConstant.HANDLE_STATUS.NO_HANDLE">
            <yd-button type="primary" size="small" @click.native="handle(IMConstant.HANDLE_STATUS.CONFIRM)">同意</yd-button>
            <yd-button type="default" size="small" @click.native="handle(IMConstant.HANDLE_STATUS.REJECT)">拒绝</yd-button>
        </div>
        <div v-else>{{message.handleStatus | handleStatusFilter}}</div>
    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-01-24 15:15:07 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-03-29 15:54:04
 * @desc 好友通知 消息模板 (messageType 12)
 */
import IMConstant from "@/service/im/constant.js";
import YdAvator from "@/components/avator/avator.vue";
import YdButton from "@/components/button/button.vue";

export default {
    components: { YdAvator, YdButton },
    props: {
        message: Object
    },
    data() {
        return {
            IMConstant
        }
    },
    computed: {
        showUser() {
            if (this.message.isRequestUser) {
                return this.message.oppositeUser;
            } else {
                return this.message.requestUser;
            }
        }
    },
    filters: {
        handleStatusFilter(status) {
            // 0=未处理， 1=已确认， -1=已拒绝, -2=处理错误
            return ['处理错误', '已拒绝', '未处理', '已确认'][status + 2]
        }
    },
    methods: {
        handle(handleStatus) {
            this.$emit("handle", this.message, handleStatus);
        }
    }
}
</script>

<style lang="scss" scoped>
.im-friend-request-message-item {
    height: 120px;
    padding: 20px 0;
    background: #fff;
    border-bottom: 1px solid #f2f2f2;
    display: flex;
    justify-content: space-between;
    align-items: center;

    .main-content {
        flex-grow: 1;
        height: 100%;
        padding: 0 20px;
        display: flex;
        flex-direction: column;
        justify-content: space-around;
        align-items: flex-start;
        font-size: 28px;

        .content {
            font-size: 24px;
            color: $tip_color;
        }
    }
}
</style>
