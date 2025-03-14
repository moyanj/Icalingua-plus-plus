<template>
    <div class="groupmember-root">
        <div class="groupmember-head-container">
            <div class="groupmember-head">
                <el-input v-model="searchContext" placeholder="搜索" prefix-icon="el-icon-search" clearable />
                <span class="el-icon-refresh-right groupmember-refresh" @click="refresh" />
            </div>
        </div>

        <div class="groupmember-content">
            <ContactEntry
                v-for="i in groupmembers"
                :key="i.user_id"
                :id="i.user_id"
                :remark="i.card"
                :group="i.group_id"
                :role="i.role"
                type="groupmember"
                :name="i.name"
                v-show="PinyinMatch(i.sc, searchContext) || !searchContext"
                @dblclick="$emit('dblclick', i.user_id, i.card)"
            />
        </div>
    </div>
</template>

<script>
import { ipcRenderer } from 'electron'
import ContactEntry from './ContactEntry.vue'
import PinyinMatch from 'pinyin-match'

export default {
    components: { ContactEntry },
    props: {
        groupmemberShown: { type: Boolean, required: true },
        gin: { type: Number, required: true },
    },
    data() {
        return {
            groupmembers: [],
            searchContextEdit: '',
        }
    },
    computed: {
        searchContext: {
            get() {
                return this.searchContextEdit
            },
            set(val) {
                this.searchContextEdit = val.toUpperCase()
            },
        },
    },
    async created() {
        const ownerMembers = []
        const adminMembers = []
        const normalMembers = []
        const memberinfo = await ipcRenderer.invoke('getGroupMembers', this.gin)
        console.log(memberinfo)
        if (memberinfo) {
            for (const element of memberinfo) {
                const member = {
                    user_id: element.user_id,
                    group_id: element.group_id,
                    name: element.nickname,
                    card: element.card || element.nickname,
                    sc: (element.card + element.nickname + element.user_id.toString()).toUpperCase(),
                    role: element.role,
                }
                if (member.role === 'owner') ownerMembers.push(member)
                else if (member.role === 'admin') adminMembers.push(member)
                else normalMembers.push(member)
            }
            this.groupmembers = [...ownerMembers, ...adminMembers, ...normalMembers]
        }
    },
    methods: {
        async refresh() {
            this.groupmembers = []
            const ownerMembers = []
            const adminMembers = []
            const normalMembers = []
            const memberinfo = await ipcRenderer.invoke('getGroupMembers', this.gin)
            if (memberinfo) {
                for (const element of memberinfo) {
                    const member = {
                        user_id: element.user_id,
                        group_id: element.group_id,
                        name: element.nickname,
                        card: element.card || element.nickname,
                        sc: (element.card + element.nickname + element.user_id.toString()).toUpperCase(),
                        role: element.role,
                    }
                    if (member.role === 'owner') ownerMembers.push(member)
                    else if (member.role === 'admin') adminMembers.push(member)
                    else normalMembers.push(member)
                }
                this.groupmembers = [...ownerMembers, ...adminMembers, ...normalMembers]
            }
            this.$message.success('已刷新')
        },
        PinyinMatch(str, search) {
            return PinyinMatch.match(str, search)
        },
    },
}
</script>

<style>
.el-collapse-item__header {
    padding-left: 12px;
    border-bottom: var(--chat-border-style);
    background-color: var(--panel-background);
    color: var(--panel-color-name);
}

.el-collapse-item__content {
    padding-bottom: 0;
    background-color: var(--panel-background);
}

.el-collapse-item__wrap {
    border-bottom: var(--chat-border-style);
}

.el-collapse-item__wrap > div > div:last-child > div > div {
    border-bottom: unset !important;
}

.el-tabs__header {
    margin: unset !important;
}

.el-tabs__item {
    padding: 0;
    color: var(--panel-color-name);
}

.el-tabs__nav-wrap::after {
    background-color: var(--panel-color-navi-bottom-bar);
}

.groupmember-root {
    height: 75vh;
    display: flex;
    flex-direction: column;
}

.groupmember-head-container {
    background-color: var(--panel-header-bg);
}

.groupmember-head {
    margin-right: 12px;
    display: flex;
    align-items: center;
}

.groupmember-content {
    overflow: auto;
}

.groupmember-refresh {
    cursor: pointer;
    font-size: larger;
    color: #909399;
    margin-left: 10px;
}
</style>
