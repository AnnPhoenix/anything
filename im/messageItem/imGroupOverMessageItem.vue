<template>
    <div class="im-group-over-item">
        <yd-group-avator class="avator" :user="message.opposite" :name="message.groupName" size=3 />
        <div class="main-content">
            <div class="full-name">{{message.groupName}}</div>
            <div class="content">{{message.content}}</div>
        </div>
    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-01-24 15:15:07 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-02-22 21:16:33
 * @desc 群组-解散群组 消息模板 (messageType 105)
 */
import YdGroupAvator from "@/components/avator/group-avator.vue";

export default {
    components: { YdGroupAvator },

    props: {
        message: Object
    },
}
</script>

<style lang="scss" scoped>
.im-group-over-item {
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
