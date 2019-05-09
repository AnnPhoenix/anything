<template>
    <div class="im-group-pull-request-message-item">
        <yd-group-avator class="avator" :user="message.group" :name="message.groupName" size=3 />
        <div class="main-content">
            <div class="full-name">{{message.groupName}}</div>
            <div class="content">{{message.content}}</div>
            <div class="request-user">邀请人: {{message.requestUser.fullName}}</div>
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
 * @Last Modified time: 2019-03-29 15:54:11
 * @desc 群组-群组拉人请求 消息模板 (messageType 14)
 */
import IMConstant from "@/service/im/constant.js";
import YdGroupAvator from "@/components/avator/group-avator.vue";
import YdButton from "@/components/button/button.vue";

export default {
    components: { YdGroupAvator, YdButton },

    props: {
        message: Object
    },

    data() {
        return {
            IMConstant
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
.im-group-pull-request-message-item {
    height: 140px;
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
            font-size: 26px;
            color: $tip_color;
        }

        .request-user {
            font-size: 24px;
            color: $tip_color;
        }
    }
}
</style>
