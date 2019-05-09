<template>
    <v-touch @tap="tap" class="iconfont icon-tianjiashangchuan"></v-touch>
</template>

<script>
/**
 * @Author: wangxin
 * @Date: 2018-07-04 10:32:36
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-01-31 15:05:36
 * @desc 新增操作列表
 */
import { CONTACT, IM } from "@/router/routerName";

export default {
    methods: {
        tap(e) {
            let options = [{
                label: "创建群组",
                iconfont: "icon-qunzu1",
                method: this.createGroup
            }, {
                label: "添加好友",
                iconfont: "icon-ren",
                method: this.addFriend
            }, {
                label: "扫一扫",
                iconfont: "icon-indexsaoyisao",
                method: this.scan
            }];

            this.$menus.top(e, options);
        },

        createGroup() {
            this.$router.push({
                name: CONTACT.GROUP_CREATE
            })
            this.$emit("success");
        },

        addFriend() {
            this.$router.push({
                name: CONTACT.FRIEND_ADD
            })
            this.$emit("success");
        },

        scan() {
            this.$emit("success");
        },
    }
};
</script>

<style lang="scss" scoped>
.iconfont {
    font-size: 52px;
}
</style>
