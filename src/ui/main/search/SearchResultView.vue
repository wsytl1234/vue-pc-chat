<template>
    <section class="search-result-container"
             v-if="sharedSearchState.query.length"
             v-bind:class="{active:sharedSearchState.query}"
             v-v-on-click-outside="hideSearchView"
             @click="hideSearchView"
    >
        <div class="search-result">
            <ul>
                <li class="category-item" v-if="sharedSearchState.userSearchResult.length > 0">
                    <label>{{ sharedSearchState.searchDomainInfo ? `在 ${sharedSearchState.searchDomainInfo.name} 中的搜索结果` : $t('search.new_user') }}</label>
                    <ul>
                        <li v-for="(user, index) in toShowUserList" :key="index">
                            <div class="search-result-item contact" @click.stop="chatToContact(user)">
                                <img :src="user.portrait">
                                <span>{{ user.displayName }}</span>
                                <button @click.stop="addFriend(user)">{{ $t('common.add') }}</button>
                            </div>
                        </li>
                    </ul>
                    <div v-if="!shouldShowAllUser&& this.sharedSearchState.userSearchResult.length > 5"
                         class="show-all"
                         @click.stop="showAllUser">
                        {{ $t('search.view_all') + this.sharedSearchState.userSearchResult.length }}
                    </div>
                </li>
                <li class="category-item" v-if="sharedSearchState.channelSearchResult.length > 0">
                    <label>{{ $t('common.channel') }}</label>
                    <ul>
                        <li v-for="(channel, index) in toShowChannelList" :key="index">
                            <div class="search-result-item contact" @click.stop="chatToChannel(channel)">
                                <img :src="channel.portrait">
                                <span>{{ channel.name }}</span>
                            </div>
                        </li>
                    </ul>
                    <div v-if="!shouldShowAllChannel&& this.sharedSearchState.channelSearchResult.length > 5"
                         class="show-all"
                         @click.stop="showAllChannel">
                        {{ $t('search.view_all') + this.sharedSearchState.channelSearchResult.length }}
                    </div>
                </li>
                <li class="category-item" v-if="sharedSearchState.contactSearchResult.length > 0">
                    <label>{{ $t('common.contact') }}</label>
                    <ul>
                        <li v-for="(contact, index) in toShowContactList" :key="index">
                            <div class="search-result-item contact" @click.stop="chatToContact(contact)">
                                <img :src="contact.portrait">
                                <span>{{ contact._displayName }}</span>
                            </div>
                        </li>
                    </ul>
                    <div v-if="!shouldShowAllContact && this.sharedSearchState.contactSearchResult.length > 5"
                         class="show-all"
                         @click.stop="showAllContact">
                        {{ $t('search.view_all') + this.sharedSearchState.contactSearchResult.length }}
                    </div>
                </li>
                <li class="category-item" v-if="sharedSearchState.groupSearchResult.length > 0">
                    <label>{{ $t('contact.group') }}</label>
                    <ul>
                        <li v-for="(group, index) in toShowGroupList" :key="index">
                            <div class="search-result-item group" @click="chatToGroup(group)">
                                <img :src="group.portrait">
                                <span>{{ group.remark ? group.remark : group.name }}</span>
                            </div>
                        </li>
                    </ul>
                    <div v-if="!shouldShowAllGroup && this.sharedSearchState.groupSearchResult.length > 5"
                         class="show-all"
                         @click.stop="showAllGroup">
                        {{ $t('search.view_all') + this.sharedSearchState.groupSearchResult.length }}
                    </div>
                </li>
                <li class="category-item" v-if="sharedSearchState.conversationSearchResult.length > 0">
                    <label>{{ '会话' }}</label>
                    <ul>
                        <li v-for="(conv, index) in toShowConversationList" :key="index">
                            <div class="search-result-item group" @click="chatToConversation(conv.conversation)">
                                <img :src="conv.conversation._target.portrait">
                                <span>{{ conv.conversation._target._displayName }}</span>
                            </div>
                        </li>
                    </ul>
                    <div v-if="!shouldShowAllConversation && this.sharedSearchState.conversationSearchResult.length > 5"
                         class="show-all"
                         @click.stop="showAllConversation">
                        {{ $t('search.view_all') + this.sharedSearchState.conversationSearchResult.length }}
                    </div>
                </li>
                <li class="category-item" v-if="sharedMiscState.isElectron">
                    <label>{{ $t('search.message_history') }}</label>
                    <div class="search-result-item message" @click="showMessageHistoryPage">
                        <p>{{ $t('search.search_message_history') }} </p>
                    </div>
                </li>
            </ul>
        </div>
    </section>
</template>

<script>
import store from "../../../store";
import Conversation from "../../../wfc/model/conversation";
import ConversationType from "../../../wfc/model/conversationType";
import FriendRequestView from "../contact/FriendRequestView.vue";
import IpcEventType from "../../../ipcEventType";
import {ipcRenderer} from "../../../platform";
import wfc from "../../../wfc/client/wfc";
import {vOnClickOutside} from '@vueuse/components'

export default {
    name: "SearchResultView",
    props: [
        "query"
    ],
    data() {
        return {
            sharedSearchState: store.state.search,
            sharedMiscState: store.state.misc,
            shouldShowAllUser: false,
            shouldShowAllChannel: false,
            shouldShowAllContact: false,
            shouldShowAllGroup: false,
            shouldShowAllConversation: false,
        }
    },

    mounted() {
        // do nothing
        store.setSearchQuery(this.query)
    },

    beforeUnmount() {
        store.setSearchQuery('')
    },

    watch: {
        // "query":function (val, oldVal){
        //   console.log('searchView query changed:', val, oldVal)
        // }
        // or
        query() {
            console.log('searchView query changed:', this.query)
            store.setSearchQuery(this.query)
        }
    },

    methods: {
        isFriend(userId) {
            return wfc.isMyFriend(userId);
        },
        addFriend(user) {
            this.$modal.show(
                FriendRequestView,
                {
                    userInfo: user,
                },
                null,
                {
                    name: 'friend-request-modal',
                    width: 600,
                    height: 250,
                    clickToClose: false,
                }, {})
        },
        showAllUser() {
            this.shouldShowAllUser = true;
        },
        showAllChannel() {
            this.shouldShowAllChannel = true;
        },
        showAllContact() {
            this.shouldShowAllContact = true;
        },

        showAllGroup() {
            this.shouldShowAllGroup = true;
        },

        showAllConversation() {
            this.shouldShowAllConversation = true;
        },
        hideSearchView(e) {
            console.log('hideSearchView', e);
            if (e.target.id !== 'searchInput' && e.target.classList[0] !== 'show-all') {
                store.hideSearchView()
            }
        },

        chatToContact(contact) {
            if (this.$router.currentRoute.path !== '/home') {
                this.$router.replace("/home");
            }
            let conversation = new Conversation(ConversationType.Single, contact.uid, 0);
            store.setCurrentConversation(conversation);
            store.hideSearchView();
        },

        chatToChannel(channel) {
            if (this.$router.currentRoute.path !== '/home') {
                this.$router.replace("/home");
            }
            let conversation = new Conversation(ConversationType.Channel, channel.channelId, 0);
            store.setCurrentConversation(conversation);
            store.hideSearchView();
        },

        chatToGroup(group) {
            if (this.$router.currentRoute.path !== '/home') {
                this.$router.replace("/home");
            }
            let conversation = new Conversation(ConversationType.Group, group.target, 0);
            store.setCurrentConversation(conversation);
            store.hideSearchView();
        },

        chatToConversation(conversation) {
            if (this.$router.currentRoute.path !== '/home') {
                this.$router.replace("/home");
            }
            store.setCurrentConversation(conversation);
            store.hideSearchView();
        },

        showMessageHistoryPage() {
            let hash = window.location.hash;
            let url = window.location.origin;
            if (hash) {
                url = window.location.href.replace(hash, '#/message-history');
            } else {
                url += "/message-history"
            }
            ipcRenderer.send(IpcEventType.showMessageHistoryPage, {
                url: url,
            });
            console.log(IpcEventType.showMessageHistoryPage, url)
        }

    },

    computed: {
        toShowUserList: function () {
            return !this.shouldShowAllUser && this.sharedSearchState.userSearchResult.length > 5 ? this.sharedSearchState.userSearchResult.slice(0, 4) : this.sharedSearchState.userSearchResult;
        },
        toShowChannelList: function () {
            return !this.shouldShowAllChannel&& this.sharedSearchState.channelSearchResult.length > 5 ? this.sharedSearchState.channelSearchResult.slice(0, 4) : this.sharedSearchState.channelSearchResult;
        },
        toShowContactList: function () {
            return !this.shouldShowAllContact && this.sharedSearchState.contactSearchResult.length > 5 ? this.sharedSearchState.contactSearchResult.slice(0, 4) : this.sharedSearchState.contactSearchResult;
        },
        toShowGroupList: function () {
            return !this.shouldShowAllGroup && this.sharedSearchState.groupSearchResult.length > 5 ? this.sharedSearchState.groupSearchResult.slice(0, 4) : this.sharedSearchState.groupSearchResult;
        },
        toShowConversationList: function () {
            return !this.shouldShowAllConversation && this.sharedSearchState.conversationSearchResult.length > 5 ? this.sharedSearchState.conversationSearchResult.slice(0, 4) : this.sharedSearchState.conversationSearchResult;
        }
    },

    directives: {
        vOnClickOutside
    },

}
</script>

<style lang="css" scoped>

.search-result-container {
    display: none;
}

.search-result-container.active {
    display: block;
    z-index: 100;
    overflow: auto;
    /*background-color: red;*/
    background-color: #f3f3f3e5;
}

.search-result-container ul {
    list-style: none;
    background-color: white;
}


.category-item label {
    color: #b2b2b2;
    display: block;
    padding-top: 10px;
    padding-bottom: 2px;
    margin-left: 12px;
    font-size: 13px;
    border-bottom: 1px solid #eeeeee;
}

.search-result-item {
    background-color: white;
    padding: 10px 12px;
}

.search-result-item:active {
    background-color: #d9d9d9;
}

.search-result-item.contact {
    width: 100%;
    display: flex;
    align-items: center;
}

.search-result-item.contact img {
    width: 34px;
    height: 34px;
    border-radius: 2px;
}

.search-result-item.contact span {
    font-size: 14px;
    padding-left: 10px;
}

.search-result-item.contact button {
    margin-left: auto;
    padding: 3px 10px;
    border-radius: 3px;
    border: 1px solid #cccccc;
    outline: none;
}

.search-result-item.contact button:active {
    background: #cccccc;
}

.search-result-item.group {
    width: 100%;
    display: flex;
    align-items: center;
}

.search-result-item.group img {
    width: 34px;
    height: 34px;
    border-radius: 2px;
}

.search-result-item.group span {
    font-size: 14px;
    padding-left: 10px;
}

.search-result-item.message {
    height: 54px;
    display: flex;
    align-items: center;
    font-size: 13px;
}

.show-all {
    padding-left: 12px;
    color: #66789d;
    font-size: 12px;
}

</style>
