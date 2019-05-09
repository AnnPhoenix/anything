<template>
    <div class="im-group-notice-message-item">
        <span>{{message.simpleMessage}}</span>
    </div>
</template>

<script>
// 群通知消息
export default {
    props: {
        message: Object
    }
}
</script>

<style lang="scss" scoped>
.im-group-notice-message-item {
    touch-action: pan-y !important;
    padding: 20px;
    display: flex;
    justify-content: center;

    span {
        display: inline-block;
        padding: 10px 26px;
        font-size: 26px;
        line-height: 30px;
        max-width: 80%;
        color: #fff;
        background: #c0c0c0;
        border-radius: 26px;
    }
}
</style>
