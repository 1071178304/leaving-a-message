<template>
    <view>
        <!-- 留言 -->
        <scroll-view class="scrollView" :scroll-top="scrollTop" scroll-with-animation scroll-y="true" lower-threshold="100"
            :style="{ height: scrollViewH + 'px' }">
            <view class="recruit-details-header-box" v-for="(item,index) in detailsList" :key="index">
                <image :src="item.user.avatar" class="recruit-details-header"></image>
                <text class="recruit-details-header-name">{{item.user.nickname == '' ? item.user.username : item.user.nickname}}</text>
                <text class="recruit-details-header-time">{{item.create_time | ctime }}</text>
                <view class="recruit-details-header-text">{{item.content}}</view>
            </view>
        </scroll-view>
        <!-- 发表 -->
        <view style="height: 95rpx;"></view>
        <view class="help-star">
            <view class="recruit-details-input">
                <image src="../../../static/recruit-details/write.png" class="recruit-details-input-img"></image>
                <!-- confirm-type="search" v-model="titles" :placeholder="plasearch" @confirm="search" -->
                <input class="uni-input" :placeholder="plasearch" cursor-spacing="14" type="text" confirm-type="down"
                    @confirm="sendCom" />
            </view>
            <block v-if="!is_collection">
                <image src="../../../static/detail/star.png" @click="onstar(id,is_collection)" class="star-img"></image>
                <view class="recruit-details-ok"><text>收藏</text></view>
            </block>
            <block v-else>
                <image src="../../../static/detail/star2.png" @click="onstar(id,is_collection)" class="star-img"></image>
                <view class="recruit-details-ok"><text>已收藏</text></view>
            </block>

            <!-- btn -->
            <block v-if="statusValue == 0">
                <block v-if="!is_join">
                    <view class="recruit-details-btn" @click="joinbtn()">加入</view>
                </block>
                <block v-else>
                    <view class="recruit-details-btn2" @click="joinbtn()">取消加入</view>
                </block>
            </block>
            <block v-else>
                <block v-if="!is_join">
                    <view class="recruit-details-btn">加入</view>
                </block>
                <block v-else>
                    <view class="recruit-details-btn2">取消加入</view>
                </block>
            </block>
        </view>
    </view>
</template>

<script>
    import {
        formatDate
    } from '../../../common/js/time.js' // 过滤时间戳
    import appShare, {
        closeShare
    } from '../../../utils/share.js';
    export default {
        data() {
            return {
                // 分页
                page: 0, //页数
                limit: 10, //传的条数
                is_collection: '', //收藏
                //列表
                detailsList: [],
                plasearch: '留言',
                star: false,
                join: 2,
                recruitId: '',
                title: '',
                name: '',
                num: '',
                org_activity_user_count: '',
                address: '',
                statusValue: '',
                statusText: '',
                mobile: '',
                create_time: '',
                organizer: '',
                content: '',
                start_date: '',
                end_date: '',
                is_join: '',
                // qs
                show: false,
                imgTextMsg: {
                    content: ''
                },
                commentData: [],
                scrollViewH: 100,
                // page: 0,
                scrollTop: 0,
                imgTextH: 20,
                id: '',
                date: {},
                comment: [],
            };
        },
        onLoad(option) {
            var recruitId = option.id;
            this.recruitId = recruitId;
            console.log(recruitId, "111")
            this.recruit()
            this.recruitdetailsList()
        },
        onShow() {
            this.recruit()
        },
        onReady() {
            this.scrollViewH = 250;
        },
        // 当滑动到底部实现数组拼接
        onReachBottom() {
            this.recruitdetailsList()
        },
        filters: {
            //过滤时间戳
            formatDate(time) {
                var date = new Date(time * 1000);
                return formatDate(date, "yyyy-MM-dd hh:mm");
            },
            //过滤时间戳
            ctime(time) {
                var date = new Date(time * 1000);
                return formatDate(date, "yyyy-MM-dd");
            }
        },
        methods: {

            // 表单 少一个push上去
            sendCom(e, recruitId) {
                if (!this.$store.state.isLogin) {
                    uni.showModal({
                        title: '提示',
                        content: '请先登录',
                        confirmText: '去登录',
                        success(res) {
                            if (res.confirm) {
                                uni.navigateTo({
                                    url: '../../login/login',
                                })
                            } else if (res.cancel) {
                                return
                            }
                        }

                    })
                } else {

                    var UserInfo = uni.getStorageSync('userInfo');
                    const token = uni.getStorageSync('token')
                    var uid = UserInfo.user_id;
                    console.log(e.detail.value, "123")
                    console.log(UserInfo, "uid==============")
                    console.log(this.recruitId, 'id')
                    uni.request({
                        url: this.Livingurl + '#',
                        method: 'POST',
                        data: {
                            content: e.detail.value,
                            user_id: uid,
                            org_activity_id: this.recruitId
                        },
                        header: {
                            'token': token,
                        },
                        success: (res) => {
                            console.log(UserInfo.avatar, "img")
                            this.detailsList.push({
                                content: e.detail.value,
                                create_time: UserInfo.expiretime,
                                user: {
                                    "avatar": UserInfo.avatar,
                                    nickname: UserInfo.nickname,
                                    username: UserInfo.username
                                }
                            })
                            // user头像
                            // this.recruitdetailsList()
                        },
                        fail: (res) => {
                            console.log(res)
                        }
                    })
                }
            },
            // 按钮
            joinbtn(recruitId) {
                if (!this.$store.state.isLogin) {
                    uni.showModal({
                        title: '提示',
                        content: '请先登录',
                        confirmText: '去登录',
                        success(res) {
                            if (res.confirm) {
                                uni.navigateTo({
                                    url: '../../login/login',
                                })
                            } else if (res.cancel) {
                                return
                            }
                        }

                    })
                } else {
                    var UserInfo = uni.getStorageSync('userInfo');
                    const token = uni.getStorageSync('token')
                    var uid = UserInfo.user_id;
                    console.log(uid, "uid==============")
                    console.log(this.recruitId, "ssssid")
                    uni.request({
                        url: this.Livingurl + '#',
                        method: 'POST',
                        data: {
                            user_id: uid,
                            org_activity_id: this.recruitId
                        },
                        header: {
                            'token': token,
                        },
                        success: (res) => {
                            console.log(res)
                            this.recruit();

                        }

                    })
                }
            },
            // 收藏
            onstar(recruitId, is_collection) {
                if (!this.$store.state.isLogin) {
                    uni.showModal({
                        title: '提示',
                        content: '请先登录',
                        confirmText: '去登录',
                        success(res) {
                            if (res.confirm) {
                                uni.navigateTo({
                                    url: '../../login/login',
                                })
                            } else if (res.cancel) {
                                return
                            }
                        }

                    })
                } else {
                    if (is_collection === 0) {
                        var UserInfo = uni.getStorageSync('userInfo');
                        var uid = UserInfo.user_id;
                        // console.log(id ,"收藏信息id==============")
                        // console.log(uid,"id==============")
                        let poat_data = {
                            user_id: uid,
                            type: 3,
                            collection_id: this.recruitId
                        }
                        console.log(poat_data, 'poat_data==')
                        uni.request({
                            url: this.Livingurl + '#',
                            method: 'POST',
                            data: poat_data,
                            success: (res) => {
                                this.recruit();
                                console.log(res, "hello")
                                uni.showToast({
                                    title: "成功收藏",
                                    position: 'bottom',
                                    success() {}
                                })

                            },
                            fail: (res) => {
                                console.log(res)
                            }
                        })
                    } else {
                        console.log("收藏id", is_collection)
                        uni.request({
                            url: this.Livingurl + '#',
                            method: 'GET',
                            data: {
                                id: is_collection
                            },
                            success: (res) => {
                                this.recruit();
                                uni.showToast({
                                    title: "取消收藏",
                                    position: 'bottom',
                                    success() {}
                                })
                                console.log("取消了", is_collection, "1111111111");
                            }
                        })
                    }


                }
            },
            // 活动留言
            recruitdetailsList(recruitId) {
                const token = uni.getStorageSync('token')
                this.page++;
                console.log(this.page)
                uni.request({
                    url: this.Livingurl + '#',
                    method: 'GET',
                    data: {
                        org_activity_id: this.recruitId,
                        page: this.page, //页数
                        limit: this.limit, //次数
                    },
                    success: (res) => {
                        console.log(res, "123")
                        console.log(this.page)
                        if (res.data.code === 0) {
                            let data1 = res.data.data.data
                            this.detailsList = this.detailsList.concat(data1)
                            console.log(this.detailsList, "???")
                        }
                    }
                })
            },
            // 活动详情
            recruit(recruitId) {
                var UserInfo = uni.getStorageSync('userInfo');
                const token = uni.getStorageSync('token')
                var uid = UserInfo.user_id;
                console.log(UserInfo, "用于头像")
                uni.request({
                    url: this.Livingurl + '#',
                    method: 'GET',
                    data: {
                        id: this.recruitId,
                        uid: uid
                    },
                    success: (res) => {
                        if (res.data.code === 1) {
                            let data = res.data.data
                            console.log(data)
                            this.num = data.num
                            this.org_activity_user_count = data.org_activity_user_count
                            this.statusText = data.status.text
                            this.statusValue = data.status.value
                            this.organizer = data.org.title
                            this.content = data.content
                            this.title = data.title
                            this.mobile = data.mobile
                            this.name = data.name
                            this.start_date = data.start_date
                            this.is_collection = data.is_collection
                            this.create_time = data.create_time
                            this.end_date = data.end_date
                            this.is_join = data.is_join
                            this.address = data.address
                        }
                    }
                });
                console.log(this.recruitId, "1111")
            },
            back() {
                uni.navigateBack({
                    delta: 1
                });
            },
            onShare() {
                let shareData = {
                    shareUrl: 'https://kemean.com/',
                    shareTitle: '分享的标题',
                    shareContent: '分享的描述',
                    shareImg: 'http://qn.kemean.cn//upload/202004/18/1587189024467w6xj18b1.jpg',
                    appId: 'wxd0e0881530ee4444', // 默认不传type的时候，必须传appId和appPath才会显示小程序图标
                    appPath: 'pages/home/home',
                    appWebUrl: 'https://kemean.com/'
                };
                // 调用
                let shareObj = appShare(shareData, res => {
                    console.log('分享成功回调', res);
                    // 分享成功后关闭弹窗
                    // 第一种关闭弹窗的方式
                    closeShare();
                });
            },
        }
    }
</script>

<style lang="scss" scoped>

    .recruit-details-text {
        font-size: 28rpx;
        color: #606266;
        line-height: 45rpx;
        width: 668rpx;
        padding-bottom: 37rpx;
        margin: 0 auto;
    }

    // 留言
    .recruit-details-header-box {
        padding: 20rpx 40rpx;
        border-bottom: 1rpx solid #F8F8FA;
        height: 180rpx;

        .recruit-details-header {
            width: 60rpx;
            border-radius: 50%;
            float: left;
            height: 60rpx;
        }

        .recruit-details-header-name {
            font-size: 28rpx;
            color: #303133;
            float: left;
            padding-left: 15rpx;
            line-height: 60rpx;
            font-weight: bold;
        }

        .recruit-details-header-time {
            font-size: 24rpx;
            color: #909399;
            float: right;
            line-height: 60rpx;
        }

        .recruit-details-header-text {
            padding: 20rpx 0;
            float: right;
            width: 593rpx;
            font-size: 28rpx;
            color: #303133;
        }
    }

    .help-star {
        width: 100%;
        position: fixed;
        bottom: 0;
        background: white;
        height: 100rpx;

        .recruit-details-input {
            margin: 19rpx 40rpx;
            width: 240rpx;
            float: left;
            height: 60rpx;
            background: url(../../../static/recruit-details/background.png) no-repeat center;
            background-size: 100% 100%;

            .recruit-details-input-img {
                margin: 16rpx 20rpx;
                width: 32rpx;
                float: left;
                height: 28rpx;
            }

            .uni-input {
                padding-top: 12rpx;
                padding-left: 12rpx;
                float: left;
                font-size: 28rpx;
                color: #909299;
                width: 100rpx;
            }
        }

        .star-img {
            float: left;
            margin: 32rpx 11rpx 0rpx 25rpx;
            width: 36rpx;
            height: 35rpx;
        }

        .recruit-details-ok {
            float: left;
            color: #303133;
            padding-top: 32rpx;
            font-size: 28rpx;
        }
    }

    .recruit-details-btn {
        float: right;
        width: 220rpx;
        height: 88rpx;
        line-height: 88rpx;
        text-align: center;
        background: #4688FA;
        border-radius: 100rpx;
        font-size: 28rpx;
        color: white;
        margin: 8rpx 40rpx 0rpx 0rpx;
    }

    .recruit-details-btn2 {
        float: right;
        width: 220rpx;
        height: 88rpx;
        line-height: 88rpx;
        text-align: center;
        background: #f5f5f5;
        border-radius: 100rpx;
        font-size: 28rpx;
        color: #c0c4cc;
        margin: 8rpx 40rpx 0rpx 0rpx;
    }

    // 活动介绍
    .item {
        padding: 20upx 0;
        position: relative;
        color: #303133;

        .userMsg-time {
            line-height: 60upx;

            image {
                vertical-align: middle;
                width: 60upx;
                height: 60upx;
                border-radius: 50%;
            }

            .name {
                font-size: 28upx;
                margin-left: 15upx;
            }

            .time {
                color: #909399;
                font-size: 24upx;
                position: absolute;
                right: 0;
                top: 20upx;
                display: block;
                height: 60upx;
            }
        }

        p {
            padding-left: 70upx;
            font-size: 28upx;
        }
    }

    .bottom {
        height: 100upx;
        background-color: #fff;
        padding: 20upx 40upx;
        box-sizing: border-box;
        // background-color: #ccc;
        text-align: right;
        position: relative;

        .input {
            height: 60upx;
            line-height: 60upx;
            border-radius: 30upx;
            background-color: #f5f7fa;
            position: absolute;
            left: 40upx;
            width: 300rpx;
            top: 20upx;
            text-align: left;

            image {
                position: absolute;
                left: 20upx;
                top: 20upx;
                width: 32upx;
                height: 28upx;
            }

            input {
                position: absolute;
                left: 65upx;
                top: 50%;
                transform: translateY(-50%);
                width: 85%;
                height: 28upx;
                font-size: 28upx;
                line-height: 28upx;
                border: none;
                outline: none;
                margin: 0;
                padding: 0;
            }
        }

        .sendBtn {
            height: 60upx;
            border-radius: 30upx;
            font-size: 28upx;
            background-color: #f5f7fa;
            line-height: 60upx;
            position: absolute;
            right: 40upx;
            top: 20upx;
            text-align: left;
            color: grey;
            box-sizing: border-box;
            padding: 0 20upx;

            image {
                vertical-align: middle;
                width: 30upx;
                height: 30upx;
                margin-left: 8upx;
                transform: translateY(-6upx);
            }
        }

        .icons {
            position: absolute;
            right: 40upx;
            top: 20upx;

            view {
                display: inline-block;
                margin-right: 30upx;

                image {
                    width: 35upx;
                    height: 35upx;
                    vertical-align: middle;
                    margin-right: 8upx;
                    transform: translateY(-6upx);
                }

                font-size: 28upx;
            }

            .collect {
                margin-right: 0;
            }
        }
    }
</style>
