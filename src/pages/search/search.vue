<template>
    <!--搜索-->
    <div class="search">
        <uloading v-model="uLoadShow" />
        <div class="search-head">
            <span v-for="(item,index) in Artists.slice(0,5)"
                  :key="index"
                  @click="clickHot(item.first)">{{item.first}}</span>
            <input class="search-input"
                   type="text"
                   placeholder="音乐/歌手"
                   v-model.trim="searchValue"
                   @keyup.enter="onEnter">
        </div>
        <music-list ref="musicList"
                    :list="list"
                    :listType="2"
                    @select="selectItem"
                    @pullUp="pullUpLoad" />
    </div>
</template>

<script>
    import {mapGetters, mapActions, mapMutations} from 'vuex'
    import {search, searchHot, getMusicDetail} from 'api'
    import formatSongs from 'assets/js/song'
    import ucraftLoading from 'base/uloading/uloading'
    import MusicList from 'components/music-list/music-list'
    import {loadMixin} from "assets/js/mixin";
    
    export default {
        name: "search",
        mixins: [loadMixin],
        components: {
            ucraftLoading,
            MusicList
        },
        data() {
            return {
                Artists: [],//热搜数组
                list: [],//搜索数组
                page: 0,//分页
                lockUp: true,//是否锁定上拉加载事件,默认锁定
            }
        },
        computed: {
            ...mapGetters([
                'playing',
                'currentMusic',
            ])
        },
        watch: {
            list(newList, oldList) {
                if (newList.length !== oldList.length) {
                    this.lockUp = false
                } else if (newList[newList.length - 1].id !== oldList.length > 0 && oldList[oldList.length - 1].id) {
                    this.lockUp = false
                }
            }
        },
        created() {
            // 获取热搜
            searchHot()
            .then(res => {
                if (res.data.code === 200) {
                    this.Artists = res.data.result.hots;
                    this.uLoadShow = false
                }
            })
        },
        methods: {
            // 点击热搜
            clickHot(name) {
                this.searchValue = name;
                this.onEnter()
            },
            // 搜索事件
            onEnter() {
                if (this.searchValue.replace(/(^\s+)|(\s+$)/g, "") === '') {
                    this.$uToast('搜索内容不能为空！');
                    return
                }
                this.uLoadShow = true;
                this.page = 0;
                if (this.list.length > 0) {
                    this.$refs.musicList.scrollTo();
                }
                search(this.searchValue)
                .then(res => {
                    if (res.data.code === 200) {
                        this.list = formatSongs(res.data.result.songs);
                        this._hideLoad()
                    }
                })
            },
            //滚动加载事件
            pullUpLoad() {
                this.uLoadShow = true;
                this.page += 1;
                search(this.searchValue, this.page)
                .then(res => {
                    if (res.data.code === 200) {
                        if (!res.data.result.songs) {
                            this.$uToast('没有更多歌曲啦！');
                            this.uLoadShow = false;
                            return
                        }
                        this.list = [...this.list, ...formatSongs(res.data.result.songs)];
                        this._hideLoad();
                    }
                })
            },
            //播放歌曲
            async selectItem(music) {
                let image = await this._getMusicDetail(music.id);
                music.image = image;
                this.selectAddPlay(music)
            },
            // 获取歌曲详情
            _getMusicDetail(id) {
                return getMusicDetail(id)
                .then(res => {
                    if (res.data.code === 200) {
                        return res.data.songs[0].al.picUrl
                    }
                })
            },
            ...mapMutations({
                setPlaying: 'SET_PLAYING'
            }),
            ...mapActions([
                'selectAddPlay'
            ])
        }
    }
</script>

<style lang="less" scoped>
    .search {
        position: relative;
        width: 100%;
        height: 100%;
        .search-head {
            display: flex;
            height: 40px;
            padding: 10px 15px;
            overflow: hidden;
            background: @search_bg_coloe;
            span {
                line-height: 40px;
                margin-right: 15px;
                cursor: pointer;
                &:hover {
                    color: @text_color_active;
                }
                @media (max-width: 640px) {
                    & {
                        display: none;
                    }
                }
            }
            .search-input {
                flex: 1;
                height: 40px;
                box-sizing: border-box;
                padding: 0 15px;
                border: 1px solid @btn_color;
                outline: 0;
                background: transparent;
                color: @text_color_active;
                font-size: @font_size_medium;
                box-shadow: 0 0 1px 0 #fff inset;
                &::placeholder {
                    color: @text_color;
                }
            }
        }
        .musicList {
            width: 100%;
            height: calc(~'100% - 50px');
        }
    }
</style>
