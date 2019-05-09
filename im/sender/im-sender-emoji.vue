<template>
    <div class="input-emoji">
        <mt-swipe :auto="0" :continuous="false">
            <mt-swipe-item v-for="(group, index) in itemGroups" :key="index">
                <div class="container">
                    <div class="emoji-item-container" v-for="(item, subIndex) in group" :key="subIndex" @click="tap(item, $event)">
                        <!-- <img class="emoji" :class="`f${item.name}`" :src="item.emoji" :alt="`f${item.name}`"/> -->
                        <span contenteditable="false" class="emoji" :class="`f${item.name}`" />
                    </div>
                </div>
            </mt-swipe-item>
        </mt-swipe>
    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-02-26 14:45:09 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-03-08 16:37:11
 * @desc inputer.emoji
 * @TODO 复制的问题, 因为contenteditable元素设置了禁止粘贴dom等元素(手机端特殊需求), 所以复制操作不会处理emoji表情. 如果需要同时粘贴复制, 需要约定特殊字符并转义, 暂不落地
 */
import emojis from '@/util/emoji'

export default {
    data() {
        return {
            emojis
        }
    },
    
    computed: {
        itemGroups() {
            let col = 7;
            let row = 3;
            let pageSize = col * row,
                arrs = [],
                temp = Object.assign([], this.emojis);

            while (temp.length > 0) {
                arrs.push(temp.splice(0, pageSize));
            }
            return arrs;
        }
    },

    methods: {
        tap(item, e) {
            let dom = '';
            dom = e.target.innerHTML && e.target.children[0] || e.target
            this.$emit('insertEmoji', `f${item.name}`)
            // this.$emit('insertEmoji', dom.cloneNode())
        }
    }
}
</script>

<style lang="scss">
@import '@/assets/scss/emoji.scss';
$emoji_width: (100/7) * 1%;

.input-emoji {
    height: 548px;
    padding: 0 20px;
    padding-bottom: 50px;
    .container {
        width: 100%;
        padding: 68px 0;
        display: flex;
        flex-wrap: wrap;
        justify-content: flex-start;
        .emoji-item-container {
            width: $emoji_width;
            margin-bottom: 70px;
            flex-shrink: 0;
            text-align: center;
        }
    }
}
</style>


<style lang="scss">
.input-emoji {
    .mint-swipe-indicators {
        .mint-swipe-indicator {
            width: 16px;
            height: 16px;
            opacity: 0.6;
            margin: 0 6px;
            background: $tip_color;
            &.is-active {
                background: $content_color;
            }
        }
    }
}
</style>