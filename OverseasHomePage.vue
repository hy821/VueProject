<template>
  <div class="overseasHomePage">
    <div class="header">
      <img class="l-logo" src="../assets/img/movies/img_logo.png" />
      <p class="nick">昵称</p>
      <img class="m-logo" src="../assets/img/movies/img_logo.png" />
      <a class="history" @click="linkMore">观看历史</a>
    </div>
    <div class="main">
      <mt-navbar v-model="selected">
        <mt-tab-item v-for="item in tabs" :key="item.id" :id="item.id">{{item.name}}</mt-tab-item>
      </mt-navbar>

      <mt-tab-container v-model="selected">
        <mt-tab-container-item v-for="item in tabs" :key="item.id" :id="item.id">
          <ul
            :class="'verticalUl page'+item.id"
            v-infinite-scroll="fetch"
            infinite-scroll-disabled="loading"
            infinite-scroll-distance="80"
          >
            <router-link
              tag="li"
              v-for="item1 in item.list"
              :key="item1.id"
              :to="'/OverseasVideoDetailPage?id='+item1.shareId+'&type=note'"
              @click.native="counts()"
            >
              <div class="img-group">
                <img :src="item1.cover.src" />
                <div v-if="item.id!=7" class="img-gradeLab">7.7分</div>
                <div v-if="item.id!=7" class="img-stateLab">更新中</div>
              </div>

              <template v-if="item.id==7">
                <div class="info">
                  <img class="sv-icon" src="../assets/img/movies/img_logo.png" />
                  <div class="ps">
                    <p class="p1">影视大魔王</p>
                    <p class="p2">一个正经Up主...</p>
                  </div>
                </div>
              </template>
              <template v-else>
                <p class="titles">{{item1.title}}</p>
                <p class="desc">一句话简介</p>
              </template>
            </router-link>
          </ul>

          <div class="loadingMore">
            <p v-show="item.status==1">加载中...</p>
            <p v-show="item.status==2">暂无数据</p>
            <p v-show="item.status==4">你到达我的底线了</p>
          </div>
        </mt-tab-container-item>
      </mt-tab-container>
    </div>
  </div>
</template>

<script>
  import ajaxAPI from "../assets/js/ajaxAPI";
  import config from "../assets/js/config";
  import { Storage } from "../assets/js/util";
  import Toast from "mint-ui/packages/toast/src/toast";
  import {
    InfiniteScroll,
    Indicator,
    Navbar,
    TabItem,
    TabContainer,
    TabContainerItem
  } from "mint-ui";

  export default {
    data() {
      return {
        selected: 1,
        tabs: [
          {
            id: 1,
            channelId: 1,
            name: "电影",
            list: [], // 当前标签页的数据集合
            api: "getOsHomePageList", // 当前标签页获取数据的api接口
            cursor: 0, // cursor分页
            pageIndex: 0, // 当前标签页的数据页码索引
            pageSize: 12, // 当前标签页的每页数据条数
            status: 0 // 当前标签页的数据状态(0:'未加载',1:'加载中',2:'暂无数据',3:'还有更多',4:'已全部加载',5:'初始加载中')
          },
          {
            id: 2,
            channelId: 2,
            name: "电视剧",
            list: [],
            api: "getOsHomePageList",
            cursor: 0,
            pageIndex: 0,
            pageSize: 12,
            status: 0
          },
          {
            id: 3,
            channelId: 3,
            name: "综艺",
            list: [],
            api: "getOsHomePageList",
            cursor: 0,
            pageIndex: 0,
            pageSize: 12,
            status: 0
          },
          {
            id: 4,
            channelId: 4,
            name: "动漫",
            list: [],
            api: "getOsHomePageList",
            cursor: 0,
            pageIndex: 0,
            pageSize: 12,
            status: 0
          },
          {
            id: 5,
            channelId: 5,
            name: "伦理",
            list: [],
            api: "getOsHomePageList",
            cursor: 0,
            pageIndex: 0,
            pageSize: 12,
            status: 0
          },
          {
            id: 6,
            channelId: 6,
            name: "会员专区",
            list: [],
            api: "getOsHomePageList",
            cursor: 0,
            pageIndex: 0,
            pageSize: 12,
            status: 0
          },
          {
            id: 7,
            channelId: 7,
            name: "短片",
            list: [],
            api: "getSubjectSearch",
            cursor: 0,
            pageIndex: 0,
            pageSize: 12,
            status: 0
          }
        ],
        channel: "",
        loading: false
      };
    },
    watch: {
      selected() {
        console.log("slected changed");
        Storage.setItem("OverseasHomePage_selected", this.selected);
        let curPage = this.tabs[this.selected - 1];
        this.loading = curPage.status == 1 || curPage.status == 5;
        curPage.status == 0 && this.fetch();
      }
    },
    created() {
      this.channel = this.$route.query.channel || "wx";
      this.selected = Storage.getItem("OverseasHomePage_selected") * 1 || 1;
    },
    mounted() {
      //this.fetch();
    },
    methods: {
      openIndicator() {
        Indicator.open({
          text: "加载中...",
          spinnerType: "fading-circle"
        });
      },
      linkMore() {
        window.open("http://h5.ikuaishou.com/#/wxDownload?type=movies");
      },
      fetch() {
        let curPage = this.tabs[this.selected - 1];
        let oldStatus = curPage.status;
        switch (curPage.status) {
          case 0:
            curPage.status = 5; // 初始加载状态
            this.openIndicator();
            break;
          case 1:
            return console.log("加载中，请稍后");
          case 2:
            return console.log("暂无数据");
          case 3:
            curPage.status = 1; // 加载状态(非初始)
            break;
          case 4:
            return console.log("已加载全部");
          case 5:
            return console.log("初始加载中，请稍后");
        }
        this.loading = true;

        if (curPage.id == 7) {
          //短片
          ajaxAPI[curPage.api](
            "",
            curPage.pageIndex,
            curPage.pageSize,
            this.channel
          )
            .then(res => {
              Indicator.close();
              this.loading = false;
              let list = res.data.data.data;
              if (res.data.code == 10000 && list) {
                curPage.pageIndex++;
                curPage.list = curPage.list.concat(list);
                if (list.length > 0) {
                  curPage.status = 3; // 还有更多
                } else {
                  if (oldStatus == 0) {
                    //初始加载时，没数据
                    curPage.status = 2; //暂无数据
                  } else {
                    curPage.status = 4; //没有更多了
                  }
                }
              } else {
                curPage.status = oldStatus;
                Toast("搜索出错，请重试");
              }
            })
            .catch(res => {
              console.error(res);
              Indicator.close();
              this.loading = false;
              curPage.status = oldStatus;
              Toast("搜索出错，请重试");
            });
        } //非短片 根据cursor分页
        else {
          ajaxAPI[curPage.api](
            curPage.channelId,
            curPage.cursor,
            curPage.pageSize
          )
            .then(res => {
              Indicator.close();
              this.loading = false;
              let list = res.data.data.data;
              if (res.data.code == 10000 && list) {
                if (list.length > 0) {
                  curPage.cursor = list[list.length - 1].cursor;
                  curPage.status = 3; // 还有更多
                } else {
                  if (oldStatus == 0) {
                    //初始加载时，没数据
                    curPage.status = 2; //暂无数据
                  } else {
                    curPage.status = 4; //没有更多了
                  }
                }

                curPage.list = curPage.list.concat(list);
              } else {
                curPage.status = oldStatus;
                Toast("搜索出错，请重试");
              }
            })
            .catch(res => {
              console.error(res);
              Indicator.close();
              this.loading = false;
              curPage.status = oldStatus;
              Toast("搜索出错，请重试");
            });
        }
      }
    }
  };
</script>

<style lang="less">
  @import "../assets/css/mixin";

  /*Indicator 加载中...*/
  .mint-indicator {
    .mint-indicator-wrapper {
      z-index: 30;
      width: 35%;
      background: none;

      .mint-indicator-text {
        color: #000000;
      }
    }

    .mint-indicator-mask {
      z-index: 20;
      background: #ffffff;
      opacity: 0.9;
    }
  }

  .overseasHomePage {
    .font(0.24rem, #131313, normal);
    position: fixed;
    background: #f8f8f8;
    width: 100%;
    height: 100%;
    overflow: hidden;

    .header {
      display: flex;
      height: 0.8rem;
      background: #fff;
      align-items: center;
      padding: 0 0.1rem;

      .l-logo {
        box-sizing: border-box;
        width: 0.5rem;
        height: 0.5rem;
        border: 1px solid #000;
        border-radius: 50%;
      }

      .nick {
        width: 1.5rem;
        font-size: 0.25rem;
        margin: 0 0.05rem;
      }

      .m-logo {
        width: 2rem;
        margin-right: auto;
      }

      .history {
        width: 1rem;
        font-size: 0.25rem;
        line-height: 0.5rem;
      }
    }

    .main {
      position: absolute;
      left: 0;
      right: 0;
      bottom: 0;
      top: 0.8rem;

      .mint-navbar {
        height: 0.4rem;
      }

      .mint-navbar .mint-tab-item {
        padding: 0 0 18px;
      }

      .mint-navbar .mint-tab-item.is-selected {
        border: 0;

        .mint-tab-item-label {
          border-color: #ffe141;
          color: #000000;
          font-weight: bold;
        }
      }

      .mint-tab-item-label {
        .font(0.18rem, #4a4a4a, normal);
        display: inline;
        border-bottom: 3px solid #ffffff;
        padding: 5px;
      }

      .mint-tab-container {
        position: absolute;
        left: 0;
        right: 0;
        top: 0.5rem;
        bottom: 0;
        margin-bottom: 5px;
        background: #fff;
        padding: 0.15rem;

        .mint-tab-container-wrap,
        .mint-tab-container-item {
          width: 100%;
          height: 100%;
          overflow: scroll;
          -webkit-overflow-scrolling: touch;
        }

        .titles {
          .font(0.25rem, #090909, normal);
          line-height: 0.3rem;
          text-align: center;
        }

        .desc {
          .font(0.23rem, #9b9b9b, normal);
          text-align: center;
        }

        .verticalUl {
          display: flex;
          justify-content: space-between;
          flex-wrap: wrap;

          li {
            width: 32%;
            margin-bottom: 0.2rem;

            .img-group {
              position: relative;

              .bdrs(0.1rem);
              width: 100%;
              height: 2.8rem;
              margin: 0;

              img {
                .bdrs(0.1rem);
                width: 100%;
                height: 100%;
                margin: 0;
              }

              .img-gradeLab {
                .font(0.05rem, #ffffff, normal);

                position: absolute;
                z-index: 2;
                left: 6px;
                bottom: 4px;
              }

              .img-stateLab {
                .font(0.05rem, #ffffff, normal);

                position: absolute;
                z-index: 2;
                right: 6px;
                bottom: 4px;
              }
            }
          }
        }

        .page7 {
          li {
            width: 100%;

            .img-group {
              //position: relative;  //默认的是position:static 这里不需要，多余
              //display: block; // div容器默认样式
              //width: 100%; // div容器默认样式
              //margin: 0; // div容器默认样式

              // img {
              //   //width: 100%; // 前面已经有了，无需覆盖
              //   height: 3rem;
              //   //margin: 0; // 前面已经有了，无需覆盖
              // }

              /* 直接在上面用v-if就可以了
                .img-gradeLab {
                  display: none;
                }

                .img-stateLab {
                  display: none;
                }*/
              height: 3rem;
            }

            .info {
              display: flex;
              height: 0.8rem;
              align-items: center;

              .sv-icon {
                // border-color: black;
                // border-width: 1px;
                // border-style: solid;
                border: 1px solid #000; // 上面的这样写就可以了

                width: 0.5rem;
                height: 0.5rem;
                .bdrs(50%);
              }

              .ps {
                margin-left: 0.1rem;

                p {
                  text-align: left;
                }
              }
            }
          }
        }

        .loadingMore {
          .font(0.25rem, #000000, normal);
          text-align: center;
          padding-bottom: 0.2rem;
        }
      }
    }
  }
</style>
