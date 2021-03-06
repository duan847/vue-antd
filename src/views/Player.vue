<template>
    <div>
        <a-divider orientation="left"><h3>{{name}}</h3></a-divider>
        <a-row type="flex" justify="space-around" align="middle">
            <a-col :xs="22" :sm="18" :md="18" :lg="18" :xl="16">
                <div id="dplayer" v-show="true" class="dplayer-hide-controller"></div>
            </a-col>
        </a-row>

        <!--播放线路、集数。少于五集，在同一个tab中显示-->
        <a-tabs v-if="urlList!== undefined &&urlList!== null && urlList.length <= 5">
            <a-tab-pane tab="播放源" key="1">
                <a-button v-for="(url,index) in urlList" :key="index" @click="switchUrl(url.url,url.name,index)"
                          class="episodes">{{url.name}}
                </a-button>
            </a-tab-pane>
            <a-button slot="tabBarExtraContent" :loading="loading" @click="updateAllInfoById">更新集数</a-button>
        </a-tabs>
        <a-tabs v-else>
            <a-tab-pane v-for="(line,index) in this.reversedMessage" :key="index">

                <span slot="tab">播放源{{index}}</span>
                <a-button v-for="(url,index) in line" :key="index" @click="switchUrl(url.url,url.name,index)"
                          class="episodes">{{url.name}}
                </a-button>
            </a-tab-pane>
            <a-button slot="tabBarExtraContent" :loading="loading" @click="updateAllInfoById">更新集数</a-button>
        </a-tabs>
        <!--下载地址-->
        <a-tabs v-if="downUrlList!== undefined &&downUrlList!== null && downUrlList.length >0">
            <a-tab-pane tab="下载源" key="1">
                <div>温馨提示：可将链接复制到百度网盘，使用百度网盘的离线下载功能高速下载（最快1秒下载完成）</div>
                <a-button v-for="(downUrl,index) in downUrlList" :key="index" @click="down(downUrl.url)"
                          class="episodes">{{downUrl.name}}
                </a-button>
            </a-tab-pane>
        </a-tabs>
        <a-row>
            <a-divider dashed orientation="left">简 介</a-divider>
            <div>{{videoDetail.synopsis}}</div>
        </a-row>
    </div>
</template>
<script>
    import {getDetailById, selectDownUrlPageById, selectUrlPageById, updateAllInfoById} from '@/api/video'

    export default {
        name: 'player',
        data() {
            return {
                Player: true,
                dp: '',
                dpshow: false,
                videoDetail: {},
                urlSize: -1,
                urlList: null,
                downUrlList: null,
                id: this.$route.params.id || this.$route.query.id,
                name: null,
                player: null,
                loading: false,
                urlIndex: 0,
                videoPlayStatus: true
            }
        },
        created() {
            Promise.all([selectUrlPageById(this.id, {size: this.urlSize}), getDetailById(this.id)]).then(resp => {
                this.videoDetail = resp[1]
                this.name = resp[1].name
                document.title = this.name
                this.urlList = resp[0].records
                this.initVideo()
            })
            selectDownUrlPageById(this.id, {size: this.urlSize}).then(resp => {
                this.downUrlList = resp.records
            })
        },
        computed: {
            reversedMessage: function () {
                let arr = []
                const map = {}
                const list = this.urlList

                if (!list) return {}
                for (let i = 0; i < list.length; i++) {
                    if (i !== 0 && list[i - 1].line !== list[i].line) {
                        map[list[i - 1].line] = arr
                        arr = []
                    }
                    arr.push(list[i])
                }
                map[list[list.length - 1].line] = arr
                return map
            }, title: function () {
                if (!this.name) {
                    return ''
                }
                return this.name + ' - ' + this.urlList[this.urlIndex].name
            }
        },
        methods: {
            //根据id更新视频所有信息
            updateAllInfoById() {
                this.loading = true
                updateAllInfoById(this.videoDetail.id).then(resp => {
                    if (resp) {
                        this.$message.success('更新集数成功');
                        selectUrlPageById(this.id, {size: this.urlSize}).then(resp => {
                            this.urlList = resp.records
                        })
                    }
                    this.loading = false
                }).catch(() => {
                    this.$message.error('抱歉，由于集数较多或网络不好，更新较慢，请稍等后手动刷新本页面');
                    this.loading = false
                })
            },
            //切换到下一集
            switchNext() {
                this.urlIndex = this.urlIndex + 1
                const url = this.urlIndex < this.urlList.length ? this.urlList[this.urlIndex].url : null
                if (url) {
                    this.player.notice('播放：' + this.name + " - " + this.urlList[this.urlIndex].name, 5000)
                    this.switchUrl(url, this.urlList[this.urlIndex].name, this.urlIndex)
                } else {
                    this.$message.error('已经是最后一集');
                }
            },
            switchUrl(url, name, index) {
                this.player.switchVideo({url: url});
                this.player.play();
                this.urlIndex = index
                document.title = this.title
            },
            initVideo() {
                const that = this
                that.player = new DPlayer({
                    container: document.getElementById('dplayer'),
                    preload: 'auto', //预加载
                    screenshot: false, // 屏幕截图
                    autoplay: true, //自动播放
                    mutex: true, //互斥，阻止多个播放器同时播放
                    volume: 0.7, //默认音量
                    video: {
                        url: this.urlList[this.urlIndex].url
                    },
                    danmaku: {
                        id: this.urlList[this.urlIndex].url,
                        api: 'https://api.prprpr.me/dplayer/',
                        // token: 'tokendemo',
                        // maximum: 1000,
                        // addition: ['https://api.prprpr.me/dplayer/v3/bilibili?aid='+ this.urlList[this.urlIndex].id],
                        // user: 'DIYgod',
                        // bottom: '15%',
                        // unlimited: true
                    }, contextmenu: [
                        {
                            text: '当前：' + this.name + " - " + this.urlList[this.urlIndex].name
                        }
                    ]
                });

                //播放结束后播放下一集
                this.player.on("ended", function () {
                    that.switchNext()
                })

            },
            down(url) {
                window.open(url)
            }
        },
        mounted() {


        },
    }

</script>

<style scoped>
    .episodes {
        margin: 0.12em;
    }

    .dplayer {
        width: 100%;
    }

    .dplayer-controller {
        /* 底部控制条 */
        display: none;
    }

    .dplayer-notice {
        /* 快退进提示 */
        display: none;
    }

    .dplayer-menu.dplayer-menu-show {
        /* 右键信息 */
        display: none;
    }

    .dplayer-mask {
        /* 悬浮层 */
        display: inline-block;
    }
</style>
