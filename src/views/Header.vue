<template>
    <div>
        <a-row>
            <a-col :xs="6" :sm="7" :md="7" :lg="6" :xl="9" >
                <img src="../assets/logo.png" class="logo" alt="首页" @click="home"/>
            </a-col>
            <a-col :xs="14" :sm="11" :md="13" :lg="9" :xl="6">
                <a-input-search placeholder="搜索 电影名/电视剧名" @search="onSearch"/>
            </a-col>
            <a-col :xs="4" :sm="6" :md="4" :lg="3" :xl="9">
                <span style="float:right">
                    <a-avatar shape="square" icon="user"/>
                </span>
            </a-col>
            <a-divider/>
        </a-row>
    </div>
</template>

<script>
    import bus from '../eventBus.js'
    export default {
        methods: {
            home(){
                this.$router.push({name: "home"})
                document.title = "看片-免费/无广告"
            },
            onSearch (text) {
                if(text) {
                    bus.$emit('eventBus', text)
                    this.$router.push({path: "search", query: {name: text}})
                }
            },
        },
    }
</script>

<style scoped>
    .ant-layout-header {
        background: #7dbcea;
        padding:0 20px;
    }
    .logo{
        width: 80px;
        cursor:pointer
    }
</style>
