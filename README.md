# vue-awesome-swipe
自定义双向绑定，网易云音乐标题

## 效果图
![](/img1.gif)	
![](/img2.gif)	
![](/img3.gif)	
## code
```
<template>
  <div :data-id="id" data-scrolltype="a" class="artist" @scroll="
myScroll($event)" :style="opacity <= 1 ? 'overflow-y: scroll': 'overflow-y: hidden'">
    <div class="container">
      <div class="playlist-box" :style="{'opacity': opacity > 0 ? opacity : '', 'background': `url(${getArtistAlbum.artist.picUrl}) no-repeat`}" >
        <div class="nav-bar" :style="{'opacity': opacity > 0 ? opacity : ''}">
          <div class="btnBar">
            <BtnBack :icon-color="'#fff'" :icon-size="'1.625rem'" />
          </div>
          <div class="btnBar">
            <span>
              <i class="iconfont iconfenxiangpt"></i>
            </span>
            <span>
              <i class="iconfont iconmore-nav"></i>
            </span>
          </div>
        </div>
        <div class="count-bar">
          <div class="icon-box">
            <span class="name">
              {{getArtistAlbum.artist.name}}
            </span>
          </div>
          <div class="icon-box">
            <span class="btnCircle"><i class="iconfont iconadd-1"></i>关注</span>
          </div>
        </div>
      </div>
      <div class="tracks">
        <div class="title" v-if="getArtistDesc.briefDesc" :style="color && color.b ? 'background:' + color.b : ''">
          <span class="icon"><i class="iconfont iconyouhuiquan"></i></span>
          <span class="text">{{getArtistDesc.briefDesc}}</span>
        </div>
        <div class="list" ref="tracksTitle" data-scrolltype="b" @scroll="myScroll($event)" :style="opacity >= 1 ? 'overflow-y: scroll': 'overflow-y: hidden'">
          <swiper class="swiper-title" :options="swiperTitle" ref="myswiper3">
            <slot v-for="item in titles">
              <swiper-slide>{{item}}</swiper-slide>
            </slot>
          </swiper>
          <swiper class="swiper-list" :options="swiperList" ref="myswiper4">
            <slot v-for="(item, index) in titles">
              <swiper-slide :data-index="index">
                <div v-if="index === 0">
                  <Loading :show="getArtistTopSong" />
                  <SongList v-if="demoSong && demoSong[0]" :title="'播放热门50'" :item="demoSong" :more-item="moreSongs" :show-num="true" :show-btn-play="false" :show-btn-icon="true" :padding="true" :slideIndex="1" @myslideto="myslideto" />
                  <Separator :show="demoAblum && demoAblum[0]" />
                  <AlbumList v-if="demoAblum && demoAblum[0]" :padding="true" :title="'最新音乐'" :item="demoAblum" :more-item="moreAblum" :slideIndex="2" @myslideto="myslideto" />
                  <Separator :show="demoVideo && demoVideo[0]" />
                  <VideoList v-if="demoVideo && demoVideo[0]" :padding="true" :title="'最新视频'" :item="demoVideo" :more-item="moreVideo" :slideIndex="3" @myslideto="myslideto" />
                  <Separator :show="demoDesc && demoDesc[0]" />
                  <DescLsit :padding="true" v-if="demoDesc && demoDesc[0]" :title="'歌手简介'" :item="demoDesc" :ellipsis="true" :more-item="moreDesc" :slideIndex="4" @myslideto="myslideto" />
                </div>
                <div v-else-if="index === 1">
                  <Loading :show="getArtistTopSong" />
                  <SongList v-if="getArtistTopSong && getArtistTopSong.songs" :padding="true" :item="getArtistTopSong.songs" :show-num="true" :show-btn-play="false" :show-btn-icon="false" />
                </div>
                <div v-else-if="index === 2">
                  <Loading :show="getArtistAlbum" />
                  <AlbumList v-if="getArtistAlbum && getArtistAlbum.hotAlbums" :padding="true" :item="getArtistAlbum.hotAlbums" />
                </div>
                <div v-else-if="index === 3">
                  <Loading :show="getArtistMv" />
                  <VideoList v-if="getArtistMv && getArtistMv.mvs" :padding="true" :item="getArtistMv.mvs" />
                </div>
                <div v-else-if="index === 4">
                  <Loading :show="getArtistDesc" />
                  <DescLsit v-if="getArtistDesc && getArtistDesc.introduction" :padding="true" :item="getArtistDesc.introduction" />
                </div>
              </swiper-slide>
            </slot>
          </swiper>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import { mapGetters, mapMutations } from 'vuex';
  import BtnBack from "../myComponents/BtnBack";
  import { swiper, swiperSlide } from 'vue-awesome-swiper';
  import Loading from "../myComponents/Loading";
  import AlbumList from "../myComponents/AlbumList";
  import SongList from "../myComponents/SongList";
  import VideoList from "../myComponents/VideoList";
  import DescLsit from "../myComponents/DescLsit";
  import Separator from "../myComponents/Separator";

  let vm = null;
  export default {
    name: "Artist",
    components: {
      swiper,
      swiperSlide,
      BtnBack,
      Loading,
      AlbumList,
      SongList,
      VideoList,
      DescLsit,
      Separator,
    },
    data() {
      return {
        id: this.$route.params.id || '',
        playlist: '',
        vipCount: null,
        color: '',
        opacity: 0,
        titles: ['主页', '歌曲', '专辑', '视频', '简介'],
        swiperTitle: {
          freeMode: true,
          freeModeMomentumRatio: 0,
          slidesPerView: 'auto',
          slidesPerGroup: 3,
          normalizeSlideIndex: false,
          allowTouchMove: false,
          on: {
            tap() {
              let swiperWidth = this.wrapperEl.clientWidth,
                  maxTranslate = this.slides[0].swiperSlideSize,
                  maxWidth = -maxTranslate + swiperWidth / 2;

              let realIndex = this.clickedIndex,
                  slide = this.slides[realIndex],
                  slideLeft = slide.offsetLeft,
                  slideWidth = slide.clientWidth,
                  slideCenter = slideLeft + slideWidth / 2;

              this.setTransition(300);
              console.log(slideCenter, swiperWidth/2, maxWidth)
              if (slideCenter < swiperWidth / 2) {
                this.setTranslate(0);
              } else if (slideCenter < maxWidth) {
                this.setTranslate(maxTranslate);
              } else {
                let nowTranslate = slideCenter - swiperWidth / 2;
                this.setTranslate(-nowTranslate);
              }

              let prevIndex = realIndex - 1 < 1 ? this.slides.length - 1 : realIndex - 1;
              let nextIndex = realIndex + 1 > this.slides.length - 1 ? 0 : realIndex + 1;
              const classNames = ['swiper-slide-prev', 'swiper-slide-active', 'swiper-slide-next'];
              for (let i = 0; i < this.slides.length; i++) {
                this.slides[i].classList.remove(...classNames);
              }

              this.slides[prevIndex].classList.add('swiper-slide-prev');
              this.slides[realIndex].classList.add('swiper-slide-active');
              this.slides[nextIndex].classList.add('swiper-slide-next');

              vm.swiperlistAr.slideTo(realIndex);
            }
          }
        },
        swiperList: {
          slidesPerView: 1,
          slidesPerGroup: 1,
          // notNextTick: true,
          autoHeight: true,
          on: {
            slideChange() {
              vm.swipernavAr.slideTo(this.activeIndex);
            }
          }
        },
        moreSongs: {
          more: true,
          moreText: '更多热歌'
        },
        moreAblum: {
          more: true,
          moreText: '更多专辑'
        },
        moreVideo: {
          more: true,
          moreText: '更多视频'
        },
        moreDesc: {
          more: true,
          moreText: '更多信息'
        },
        swiperWidth: 0,
        maxTranslate: 0,
        maxWidth: 0,
      }
    },
    computed: {
      ...mapGetters('song',[
        'getSongStatus'
      ]),
      ...mapGetters('artist', [
        'getArtistDesc',
        'getArtistTopSong',
        'getArtistAlbum',
        'getArtistMv'
      ]),
      ...mapGetters('user', [
        'getUserFolloweds'
      ]),
      swipernavAr() {
        return vm.$refs.myswiper3.swiper
      },
      swiperlistAr() {
        return vm.$refs.myswiper4.swiper
      },
      demoAblum() {
        let arr = [];
        arr.push(vm.getArtistAlbum.hotAlbums[0]);
        return arr;
      },
      demoSong() {
        let arr = [...vm.getArtistTopSong.songs];
        arr = arr.slice(0, 5);
        return arr;
      },
      demoVideo() {
        let arr = [];
        arr.push(vm.getArtistMv.mvs[0]);
        return arr;
      },
      demoDesc() {
        let arr = [];
        arr.push(vm.getArtistDesc.introduction[0]);
        return arr;
      }
    },
    created() {
      vm = this;
    },
    mounted() {
      vm.$nextTick(() => {
        let id = vm.id;
        // 用户粉丝列表
        vm.$api.userFolloweds(vm, {uid:id});
        // 歌手描述
        vm.$api.artistDesc(vm, {id:id});
        // 歌手热门50首
        vm.$api.artistTopSong(vm, {id:id});
        // 歌手专辑
        vm.$api.artistAlbum(vm, {id:id});
        // 歌手MV
        vm.$api.artistMv(vm, {id:id});
        // 更新swiper
        vm.swiperlistAr.updateAutoHeight(500);
      })
    },
    methods: {
      myslideto(index) {
        vm.swipernavAr.slideTo(index);
        vm.swiperlistAr.slideTo(index);
      },
      myScroll(el) { // 父子滚动条切换
        const type = el.target.dataset.scrolltype;
        const offsetTop = vm.$refs.tracksTitle.offsetTop;
        const scrollTop = el.target.scrollTop;
        let opacity = scrollTop / offsetTop;
        switch (type) {
          case 'a':
            vm.opacity = opacity;
            break;
          case 'b':
            if (scrollTop === 0) {
              vm.opacity = opacity;
            }
            break;
        }
      },
      playSong(t) { // 播放歌曲
        if (t.fee === 1) { // fee:1 为VIP歌曲，开会员才能听
          return false
        }
        let song =  { ...vm.getSongStatus };

        for (let s of song.songList) { // 歌曲是否存在列表中
          if (s.id === t.id) {
            song.currentSong = s;
            // vm.$api.lyric(vm, {id: s.id}); // 请求频繁容易被封IP 所以使用本地数据
          }
        }

        // 显示音乐播放器
        song.show = true;
        // 播放歌曲
        // song.play = true;
        // 歌曲专辑
        // song.currentAlbum = t.al;

        // 音乐全屏播放器的数据
        song.currentAlbum = t;
        // 歌曲名字
        // song.currentName = t.name;
        song.currentName = '这么久没见';
        vm.$store.dispatch('song/setSongStatus', song);
      },
      ...mapMutations('song',[
        'setSongStatus'
      ]),
    }
  }
</script>

<style lang="stylus" scoped>
  .btnCircle {
    width 3.75rem
    height 1.875rem
    display inline-flex
    align-items center
    justify-content center
    background #FA3737
    border-radius 4rem
    font-size 0.75rem
    color #eee
    i {
      font-size 0.75rem !important
      margin-right 0.25rem !important
    }
  }
  .artist {
    width 100vw
    height 100vh
    .iconarrow-right {
      font-size 0.75rem
    }
    .container {
      .nav-bar {
        color #fff
        padding 0.5rem
        height 3rem
        display flex
        align-items center
        justify-content space-between
        font-size 1.125rem
        .btnBar {
          width 50%
          height inherit
          display inline-flex
          align-items center
          justify-content flex-start
          font-size 1.25rem
          font-weight bold
        }
        .btnBar:last-child {
          justify-content flex-end
          span {
            width 3rem
            height 3rem
            display inline-flex
            align-items center
            justify-content center
            i {
              font-size 1.625rem
            }
          }
        }
      }
      .playlist-box {
        width 100%
        height 19.375rem
        background-size cover !important
        position relative
        z-index 0
        .count-bar {
          position absolute
          bottom 5rem
          padding 0 1.25rem
          width calc(100% - 2.5rem)
          height 5rem
          display flex
          align-items center
          justify-content space-between
          color #eee
          .iconfont {
            font-size 1.5rem
          }
          .icon-box {
            width 50%
            height inherit
            display inline-flex
            flex-direction column
            align-items flex-start
            justify-content space-evenly
            .name {
              font-size 1.25rem
              font-weight bold
            }
          }
          .icon-box:last-child {
            align-items flex-end
          }
        }
      }

      .tracks {
        margin-top -4.75rem
        .title {
          height 4.75rem
          display flex
          align-items flex-start
          background #ddd
          opacity 0.8
          border-radius 1.5rem 1.5rem 0 0
          .icon {
            width 3.75rem
            height 3.75rem
            display inline-flex
            align-items center
            justify-content center
            color #E04853
            i {
              font-size 2rem
            }
          }
          .text {
            width calc(100% - 5rem)
            height 3rem
            margin-top 0.375rem
            line-height 1.5rem
            overflow hidden
            display -webkit-box
            -webkit-box-orient vertical
            -webkit-line-clamp 2
            font-size 1rem
            text-align left
            color #333
          }
        }
        .list {
          height 100vh
          position relative
          margin-top -1rem
          z-index 100
          border-radius 1.5rem 1.5rem 0 0
          background #fff
          ul {
            width 100vw
          }
          li {
            padding-right 0.9375rem
            height 4rem
            display flex
            align-items center
            justify-content space-between
            .more {
              justify-content flex-end
            }
            div {
              height inherit
              display inline-flex
              align-items center
            }
            div:nth-of-type(1) {
              width 3rem
              font-size 1rem
              color #999
              justify-content center
            }
            div:nth-of-type(3) {
              width 2.5rem
              font-size 0.875rem
              color #999
            }
            div:nth-of-type(2) {
              width calc(100% - 6rem)
              flex-direction column
              align-items unset
              justify-content center
              span {
                display inline-flex
                align-items center
                overflow hidden
                text-overflow ellipsis
                white-space nowrap
              }
              .name {
                width 100%
                font-size 0.9375rem
                color #000
              }
              .artist {
                width 100%
                font-size 0.75rem
                color #999
                height 1.5rem
                span {
                  display inline-block
                  text-align left
                  line-height 1.5rem
                  height inherit
                  width 100%
                }
                .iconfont {
                  color #E04853
                  font-size 1.25rem
                }

              }
            }
          }
          li[disabled]:hover {
            cursor not-allowed
          }
          .btn-bar {
            padding-right 15px
            height 4rem
            display flex
            align-items center
            justify-content space-between
            span:nth-of-type(1) {
              height inherit
              display inline-flex
              align-items center
              font-size 18px
              font-weight bold
              color #000
              i {
                width 3rem
                font-size 1.5rem
                color #000
              }
            }


          }
          .subscribers {
            background #fff
            height 3.75rem
            display flex
            align-items center
            justify-content space-between
            .img {
              width 3rem
              height 3rem
              display inline-flex
              align-items center
              justify-content center
              overflow hidden
              img {
                width 2rem
                height 2rem
                border-radius 50%
              }
            }
            .count {
              font-size 14px
              color #999
              margin-right 0.9375rem
            }
          }
        }
      }

    }

  }
  .swiper-title {
    border-bottom 1px solid #ccc
    .swiper-slide {
      width 25%
      height 2.5rem
      display inline-flex
      align-items center
      justify-content center
      position relative
    }
    .swiper-slide-active {
      color #fc4444
      border-bottom-color #fc4444 !important
    }
    .swiper-slide-active::after {
      content ''
      width 50%
      height 2px
      background #fc4444
      position absolute
      bottom 0
    }
  }
  .swiper-list {
    width 100%
    height calc(100vh - 2.5rem)
    box-sizing border-box
    position absolute
    top 2.5rem
    overflow-y scroll
    .swiper-slide {
      padding-top 0.5rem
      height calc(100vh - 3rem)
      overflow-y scroll
    }
  }
</style>

```

## 这个B太懒了，自己看代码吧- -
- 因为使用官方的双向控制有Bug,就自己实现了一个类似的。
 - 核心：在标题的tap()事件中，获取translate的值，然后根据clickedIndex的值，做相应的过渡，最后再通过 slideTo(clickedIndex) 让内容同步变化。
