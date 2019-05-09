<template>
    <div class="input">
        <!-- 文本输入, 切换资源面板 -->
        <div class="input-bar">
            <!-- @TODO 语音.暂不落地 -->
            <!-- <i class="iconfont icon-yuyin" /> -->

            <!-- content.input -->
            <!-- contenteditable="plaintext-only" 不允许粘贴dom元素 -->
            <!-- style: user-modify: read-write-plaintext-only; 不允许粘贴dom元素-->
            <div id="content" class="content" contenteditable="plaintext-only" @focus="contentFocusHandler" @blur="contentBlurHandler" @input="contentInputHandler"></div>

            <!-- 表情 && 文字 -->
            <i v-if="mode == 'text'" @click="emojiClickHandler" class="iconfont icon-xiaolian" />
            <i v-else @click="contentClickHandler" class="iconfont icon-wenzi" />

            <!-- 操作 && send.button -->
            <button v-if="dialog.draftNodeList.length > 0" class="send-button" @click="send">发送</button>
            <i v-else class="operation-button iconfont icon-tianjiashangchuan" @click="operationClickHandler" />
        </div>

        <!-- 资源选择 -->
        <!-- 资源选择.emoji -->
        <im-sender-emoji v-show="emojiFocus" class="resource-banner" @insertEmoji="insertEmoji"></im-sender-emoji>
        <!-- 资源选择.operation -->
        <!-- <im-sender-operation v-if="operationFocus" class="emoji-banner" :dialog="dialog" @inputOperation="inputOperation" ref="operation"></im-sender-operation> -->
        <im-sender-operation v-show="operationFocus" class="emoji-banner" :dialog="dialog"></im-sender-operation>
    </div>
</template>

<script>
/**
 * @Author: wangxin 
 * @Date: 2019-02-26 14:26:23 
 * @Last Modified by: wangxin
 * @Last Modified time: 2019-03-28 14:29:19
 * @desc dialog.chat.sender.input
 * @TODO 可以修改成, 先预置tempRange, 如果range为null, 则先操作tempRange
 * insert永远用tempRange实现
 */
import { IM } from "@/router/routerName";
import ImSenderEmoji from './im-sender-emoji.vue';
import ImSenderOperation from './im-sender-operation.vue';

export default {
    components: {
        ImSenderEmoji,
        ImSenderOperation,
    },

    props: {
        dialog: Object
    },

    data() {
        return {
            mode: 'text',
            contentDOM: null, // dom

            /* focus states */
            inputFocus: false,
            emojiFocus: false,
            operationFocus: false,

            /* input range */
            saveRange: null,
            tempRange: null
        }
    },

    computed: {
        operationRef() {
            return this.$refs.operation;
        }
    },

    beforeDestroy() {
        if (this.saveRange) {
            this.saveRange.detach();
            this.saveRange = null;
        }

        if (this.tempRange) {
            this.tempRange.detach();
            this.tempRange = null;
        }

        let sel = this.getSelection();
        sel.removeAllRanges();
    },

    mounted() {
        this.contentDOM = this.$el.querySelector('#content')

        // 草稿消息写入contentDOM
        if (this.dialog.draftNodeList.length > 0) {
            this.insertDraftNodeList(this.dialog.draftNodeList);
        }
    },

    methods: {
        /* Input Methods about Chat Message */
        // 发送
        send() {
            let content = this.contentDOM.innerHTML;
            this.yunduService.imService.chat(this.dialog, content, this.dialog.atUids).then(() => {
                this.dialog.scrollvm.scrollToBottom();
            });

            /* init params */
            this.hasContent = false;
            this.contentDOM.innerHTML = '';
            this.contentDOM.focus();

            /* 清空dialog中保存的临时消息数据 */
            this.dialog.clearDraftMessage();
        },

        // // 发送资源
        // inputOperation(type) {
        //     switch (type) {
        //         case this.operationRef.CAMERA:
        //             break;
        //         case this.operationRef.NATIVEFILE:
        //             this.sendNativeFile();
        //             break;
        //         case this.operationRef.CLOUDFILE:
        //             this.sendCloudFile();
        //             break;
        //         case this.operationRef.GROUPFILE:
        //             this.sendGroupFile();
        //             break;
        //         case this.operationRef.ONCEREAD:
        //             break;
        //     }
        // },

        // // 发送本地文件
        // sendNativeFile() {
        //     // // 聊天消息, 生成消息并保存, 等待fileTransfer的文件上传完成通知, 然后发送此消息
        //     // let messageFile = new MessageFile();
        //     // messageFile.setMessageId();

        //     // return uploadService(this.dialog, messageFile.getMessageId(), this.opposite).then(transfer => {
        //     //     log.info('[chatDialog.vue][sendNativeFile] transfer is ', transfer);

        //     //     // 补全消息
        //     //     let socket_message = messageFile.generate(this.opposite, transfer);
        //     //     messageFile.assignExt({ transferId: transfer.id });
        //     //     messageFile.init(socket_message);
        //     //     this.dialog.handleNative(messageFile).then(() => {
        //     //         this.messageListRef.scrollToBottom();
        //     //     })
        //     // })
        // },

        // // 发送云盘文件
        // sendCloudFile() {
        //     // @TODO vuex -> save dialog -> save files -> send
        //     // this.$router.push({
        //     //     name: CLOUDDISK.FILE_BROWSER
        //     // })
        // },

        // // 发送群组文件
        // sendGroupFile() {
        //     // @TODO
        //     // this.$router.push({
        //     //     name: CLOUDDISK.FILE_BROWSER_GROUPLIST
        //     // })
        // },

        /* Content Dom Event Handlers */
        // focus
        contentFocusHandler() {
            this.mode = 'text'
            this.inputFocus = true;
            this.emojiFocus = false;
            this.operationFocus = false;
            this.$emit('heightChange');
        },

        // blur
        contentBlurHandler() {
            // 保存离开输入框时的range对象
            this.saveRange = this.getRange();
        },

        // input
        contentInputHandler(e) {
            // 是否编辑了atSpan
            let selection = this.getSelection();

            // focusNode 是span.AtMember span.dataset.atuids
            let focusNode = selection.focusNode;
            let parentNode = focusNode.parentNode;
            if (focusNode && parentNode && parentNode.nodeType == 1 && "atuids" in parentNode.dataset) {
                parentNode.classList = [];
                this.dialog.removeAtUid(parentNode.dataset.atindex);
                delete parentNode.dataset.atuids;
                delete parentNode.dataset.atindex;
            }

            // at
            if (e.data == "@") {
                if (this.dialog.isGroup) {
                    // 处于群组中, 触发@功能
                    return this.insertAt();
                }
            }

            // normal
            this.updateLength();
        },

        /* Router Click Handlers */
        // router.content.click
        contentClickHandler() {
            if (this.saveRange) {
                this.contentDOM.focus();
            } else if (this.tempRange) {
                let selection = this.getSelection();
                selection.addRange(this.tempRange);
                this.tempRange.detach();
                this.tempRange = null;
            } else {
                this.contentDOM.focus();
            }
        },

        // router.emoji.click
        emojiClickHandler() {
            this.mode = 'emoji'
            this.inputFocus = false;
            this.emojiFocus = true;
            this.operationFocus = false;
            this.$emit('heightChange');
        },

        // router.operation.click
        operationClickHandler() {
            if (this.inputFocus) {
                this.contentDOM.blur();
                this.inputFocus = false;
                this.emojiFocus = false;
                this.operationFocus = true;
            } else if (this.emojiFocus) {
                this.mode = 'text';
                this.inputFocus = false;
                this.emojiFocus = false;
                this.operationFocus = true;
            } else if (this.operationFocus) {
                // @TODO 这里要不要contentDOM.focus
                this.contentDOM.focus();
            } else {
                this.inputFocus = false;
                this.emojiFocus = false;
                this.operationFocus = true;
            }

            this.$emit('heightChange');
        },

        /* Insert Methods */
        // 更新contentDOM.content.length, 判断是否存在待发送内容
        updateLength() {
            let hasContent = this.contentDOM.innerHTML.length > 0;

            if (hasContent) {
                this.formatContentChildNodes();
                this.dialog.updateDraftNodeList(this.contentDOM.childNodes);
            } else {
                this.dialog.updateDraftNodeList([]);
            }
        },

        // 刷新contentDOM子节点的格式
        formatContentChildNodes() {
            let childNodes = this.contentDOM.childNodes;
            let nullTextNodeList = [];
            for (let index = 0; index < childNodes.length; index++) {
                let nodeItem = childNodes.item(index);

                // 筛选出空文本节点
                if (nodeItem.nodeType == 3 && nodeItem.data == "") {
                    nullTextNodeList.push(nodeItem);
                }
            }

            // 去除空节点
            nullTextNodeList.forEach(nodeItem => {
                let newNode = document.createTextNode('\u200b');
                nodeItem.parentNode.replaceChild(newNode, nodeItem);
            })
        },

        // 初始化Content Input
        insertDraftNodeList(nodeList) {
            while (nodeList.length > 0) {
                this.insertDOM(nodeList[0]);
            }

            this.updateLength();
        },

        // 插入emoji
        insertEmoji(emojiName) {
            // 插入节点
            let emojiDOM = document.createElement("span");
            emojiDOM.contentEditable = false;
            emojiDOM.classList.add("emoji", `${emojiName}`);

            this.insertDOM(emojiDOM);
            this.updateLength();
        },

        // 插入at
        insertAt() {
            // 获取range, 将当前的@文本替换为span
            this.saveRange = this.getRange();
            let node = this.saveRange.endContainer;
            let end = this.saveRange.endOffset;
            this.saveRange.setStart(node, end - 1);
            this.saveRange.setEnd(node, end);

            // 插入节点
            let atDOM = document.createElement("span");
            // atDOM.contentEditable = false;
            atDOM.innerHTML = "@";

            this.dialog.updateAtDOM(atDOM);
            this.insertDOM(atDOM);
            this.updateLength();

            this.$router.push({
                name: IM.CHOSE_AT_GROUP_MEMBER,
                params: {
                    opposite: this.dialog.opposite,
                    dialog: this.dialog
                }
            })
        },

        // 插入内容
        insertDOM(content) {
            if (!this.saveRange) {
                let selection = this.getSelection();
                if (!this.tempRange) {
                    this.tempRange = document.createRange();
                    this.tempRange.selectNodeContents(this.contentDOM);
                    this.tempRange.setStart(this.contentDOM, 0);
                }

                this.tempRange.deleteContents();
                this.tempRange.collapse(true);
                this.tempRange.insertNode(content);
                this.tempRange.setStartAfter(content);
            } else {
                this.saveRange.deleteContents();
                this.saveRange.collapse(true);
                this.saveRange.insertNode(content);
                this.saveRange.setStartAfter(content);
            }
        },

        /* Selection && Range Services */
        // get selection
        getSelection() {
            let selection = window.getSelection();
            return selection;
        },

        // get range
        getRange() {
            let selection = this.getSelection();
            return selection.rangeCount > 0 ? selection.getRangeAt(0) : null;
        }
    }
}
</script>

<style lang="scss" scoped>
@import "@/assets/scss/emoji.scss";
.input {
    display: flex;
    flex-direction: column;
    background: #fff;

    .input-bar {
        display: flex;
        align-items: flex-end;
        justify-content: space-between;
        width: 100%;
        padding: 20px 10px 20px 20px;

        > * {
            margin-right: 10px;
        }

        > .content {
            user-modify: read-write-plaintext-only;
            -webkit-user-modify: read-write-plaintext-only;
            flex-grow: 1;
            overflow-y: scroll;
            min-height: 68px;
            max-height: 240px;
            line-height: 50px;
            word-break: break-all;
            font-family: inherit;
            font-size: 28px;
            padding-bottom: 10px;
            border-bottom: 1px solid $tip_color; /* no */
            outline: none;

            &:focus {
                border-bottom: 1px solid $main_color; /* no */
            }
        }

        .send-button {
            flex-shrink: 0;
            height: 58px;
            padding: 0 20px;
            margin-bottom: 5px;
            outline: none;
            border: 1px solid $main_color; /* no */
            border-radius: 10px;
            font-size: 28px;
            text-align: center;
            color: #fff;
            background: $main_color;
        }

        .iconfont {
            font-size: 68px;
            color: $tip_color;
        }
    }
}
</style>

<style lang="scss">
.input {
    .input-bar {
        > div.content {
            > .ql-container.ql-snow {
                width: 100%;
                font-size: 36px;
                border: none;
            }

            img[src=""],
            img:not([src]) {
                opacity: 0;
            }

            .emoji {
                width: 48px;
                height: 48px;
                display: inline-block;
                // margin-bottom: 6px;
                vertical-align: bottom;
            }

            .AtMember {
                background: #add7ff;
                padding-right: 4px; /* no */
                margin-right: 4px; /* no */
            }
        }
    }
}
</style>
