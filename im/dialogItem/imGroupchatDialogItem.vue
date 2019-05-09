<template>
    <!-- 群组消息 -->
    <div class="im-groupChat-dialog-item" :class="dialogClass">
        <yd-group-avator class="dialog-avator" :group="dialog.json.entity" :name="dialog.title" size="4" />
        <div class="dialog-container">
            <div class="title-message">
                <span class="title">{{dialog.title}}</span>
                <span class="message" v-html="simpleMessage"></span>
            </div>
            <div class="moment-status">
                <span class="moment">{{dialog.lastUsedTime | momentFilter}}</span>
                <i v-if="dialog.disturbFlag != 1 && unreadCount > 0" class="unread" :class="unreadCount < 10 ? 'blue' : 'red'">{{unreadCount | unreadCountFilter}}</i>
                <i v-else-if="dialog.disturbFlag != 1 && unreadCount == 0">&nbsp;</i>
                <i v-else class="status iconfont icon-miandarao1" />
            </div>
        </div>
    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2018-09-11 11:34:40 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-03-09 14:22:12
 * @desc Dialog Item
 * from id:
 * 普通用户：(0 - 100000000)
 * 群组：[100000000 - 1000000000)
 * 系统消息：[1000000000 - ~)
 */
import YdGroupAvator from "@/components/avator/group-avator.vue";
import { momentCompare } from "@/util/formatDate"

export default {
    components: { YdGroupAvator },

    props: {
        dialog: {
            type: Object
        }
    },

    computed: {
        dialogClass() {
            return this.dialog.upFlag == 0 ? '' : 'up'
        },

        unreadCount() {
            return this.dialog.unreadCount;
        },

        simpleMessage() {
            return this.dialog.getSimpleMessage();
        }
    },

    filters: {
        // unreadCount > 99时,展示为99+
        unreadCountFilter(unreadCount) {
            return unreadCount > 99 ? '99+' : unreadCount
        }
    }
}
</script>

<style lang="scss">
@import "@/assets/scss/emoji.scss";
.im-groupChat-dialog-item {
    display: flex;
    align-items: center;
    width: 100%;
    height: 120px;
    padding: $padding;
    background: #fff;

    &.up {
        background: #eaf4fd;
    }

    .dialog-avator {
        margin-right: 20px;
        flex-shrink: 0;
    }

    .dialog-container {
        flex-grow: 1;
        display: flex;
        height: 100%;
        padding: 26px 0;
        border-bottom: $border;

        .title-message {
            flex-grow: 1;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;

            .title {
                font-size: 28px;
            }

            .message {
                max-width: 460px;
                margin-top: 8px;
                line-height: 30px;
                font-size: 24px;
                color: $tip_color;
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;

                .emoji {
                    width: 32px;
                    height: 32px;
                }

                // dialog 简要消息前缀
                .dialogSimpleMessage {
                    &.draft {
                        color: #8b1315;
                    }

                    &.at,
                    &.notice {
                        color: #ffb042;
                    }
                }
            }
        }

        .moment-status {
            // width: 200px;
            flex-shrink: 0;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: flex-end;

            .moment {
                font-size: 22px;
                color: $tip_color;
            }

            .unread {
                min-width: 30px;
                // height: 30px;
                // line-height: 30px;
                padding: 2px 12px;
                margin-top: 10px;
                color: #fff;
                border-radius: 16px;
                &.blue {
                    background: #88c5ff;
                }

                &.red {
                    background: #ff4044;
                }
            }

            .iconfont {
                color: $tip_color;
            }
        }
    }
}
</style>
