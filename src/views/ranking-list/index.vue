<template>
    <list-view :busy="busy" :noMore="noMore" :showHint="showHint" :loading="loading" :more="more" style="width: 660px;">
        <el-card
            v-for="user in userList"
            :key="user.userId"
            :body-style="{ 'display': 'flex', 'align-items': 'center', 'height': '80px', 'padding': '15px' }"
            class="user-item">
            <el-avatar class="avatar" :style="{ 'background-color': theme }">{{ getFirstChar(user.username) }}</el-avatar>
            <div class="username">{{ user.username }}</div>
            <div class="coin">积分：{{ user.coinCount }}</div>
            <div class="level">等级：{{ user.level }}</div>
            <div class="rank">{{ user.rank }}</div>
        </el-card>
    </list-view>
</template>

<script>
import ListView from '@/components/ListView';
import { getFirstChar } from '@/utils/text';
import { getRankingList } from '@/api/ranking';
import { mapGetters } from "vuex";

export default {
    data() {
        return {
            busy: false,
            noMore: false,
            showHint: false,
            loading: false,
            currPage: 1,
            userList: [],
        }
    },
    created() {
        this.getRankingData('init', this.currPage);
    },
    components: {
        ListView,
    },
    methods: {
        getFirstChar,
        more() {
            this.getRankingData('more', this.currPage);
        },
        getRankingData(state, page) {
            if (state === 'init') {
                this.loading = true;
                getRankingList(page).then(res => {
                    this.loading = false;
                    if (res.data.datas != undefined && res.data.datas.length != 0) {
                        this.userList = res.data.datas;
                        this.currPage = page + 1;
                    } else {
                        this.showHint = true;
                    }
                })
            }
            if (state === 'more') {
                this.busy = true;
                getRankingList(page).then(res => {
                    this.busy = false;
                    if (res.data.datas != undefined && res.data.datas.length != 0) {
                        this.userList.push(...res.data.datas);
                        this.currPage = page + 1;
                    } else {
                        this.noMore = true;
                    }
                })
            }
        }
    },
    computed: {
        ...mapGetters([
            'theme',
        ])
    }
}
</script>

<style lang="scss" scoped>
.user-item {
    &:not(:first-child) {
        margin-top: 6px;
    }

    & > div {
        font-size: 15px;
    }
}

.avatar {
    width: 45px;
    height: 45px;
}

.username {
    margin-left: 0.8rem;
}

.username::after,
.coin::after {
    content: '|';
    padding: 0 0.8rem;
    color: #909399;
}

.rank {
    margin-left: auto;
}
</style>