<template>
  <div class="ebook-slide-contents">
    <div class="slide-contents-search-wrapper">
      <div class="slide-contents-search-input-wrapper">
        <div class="slide-contents-search-icon">
          <span class="icon-search"></span>
        </div>
        <input
          type="text"
          class="slide-contents-search-input"
          v-model="searchText"
          :placeholder="$t('book.searchHint')"
          @keyup.enter.exact="search"
          @click="showSearchPage">
      </div>
      <div class="slide-contents-search-cancel"
           v-if="searchVisible"
           @click="hideSearchPage">
        {{ $t('book.cancel')}}
      </div>
    </div>
    <div class="slide-contents-book-wrapper" v-show="!searchVisible">
      <div class="slide-contents-book-img-wrapper">
        <img :src="cover" alt="" class="slide-contents-book-img">
      </div>
      <div class="slide-contents-book-info-wrapper">
        <div class="slide-contents-book-title">
          <span class="slide-contents-book-title-text">{{ metadata.title }}</span>
        </div>
        <div class="slide-contents-book-author">
          <span class="slide-contents-book-author text">{{ metadata.creator }}</span>
        </div>
      </div>
      <div class="slide-contents-book-progress-wrapper">
        <div class="slide-contents-book-progress">
          <span class="progress">{{ progress  + '%'}}</span>
          <span class="progress-text">{{ $t('book.haveRead2') }}</span>
        </div>
        <div class="slide-contents-book-time">{{ getReadTimeText() }}</div>
      </div>
    </div>
    <scroll class="slide-contents-list"
      :top="156"
      :bottom="48"
      v-show="!searchVisible"
      ref="scroll">
      <div class="slide-contents-item" v-for="(item, index) in navigation" :key="index">
        <span class="slide-contents-item-label"
              :class="{'selected': section === index }"
              :style="contentItemStyle(item)" @click="displayContent(item.href)">{{ item.label }}</span>
        <span class="slide-contents-item-page"></span>
      </div>
    </scroll>
    <scroll class="slide-search-list"
            :top="66"
            v-show="searchVisible"
            :bottom="48">
      <div class="slide-search-item"
           v-for="(item, index) in searchList"
           :key="index"
           v-html="item.excerpt"
           @click="displayContent(item.cfi,true)"
      >
      </div>
    </scroll>
  </div>
</template>
<script>
  import { ebookMinx } from '../../utils/mixin'
  import Scroll from '../common/Scroll'
  import { px2rem } from '../../utils/utils'

  export default {
    name: 'EbookSlideContents',
    components: { Scroll },
    mixins: [ebookMinx],
    data () {
      return {
        searchVisible: false,
        searchList: null,
        searchText: ''
      }
    },
    methods: {
      search () {
        if (this.searchText && this.searchText.length > 0) {
          this.doSearch(this.searchText).then(list => {
            this.searchList = list
            this.searchList.map(item => {
              item.excerpt = item.excerpt.replace(this.searchText, `<span class="content-search-text">${this.searchText}</span>`)
              return item
            })
          })
        }
      },
      doSearch (q) {
        return Promise.all(
          this.currentBook.spine.spineItems.map(
            section => section.load(this.currentBook.load.bind(this.currentBook))
              .then(section.find.bind(section, q))
              .finally(section.unload.bind(section)))
        ).then(results => Promise.resolve([].concat.apply([], results)))
      },
      displayContent (target, highLight = false) {
        this.display(target, () => {
          this.hideTitleAndMenu()
          if (highLight) {
            this.currentBook.rendition.annotations.highlight(target)
          }
        })
      },
      contentItemStyle (item) {
        return {
          marginLeft: `${px2rem(item.level * 15)}rem`
        }
      },
      showSearchPage () {
        this.searchVisible = true
      },
      hideSearchPage () {
        this.searchVisible = false
        this.searchText = ''
        this.searchList = null
      }
    },
    mounted () {
    }
  }
</script>
<style lang="scss" rel="styleshee/scss" scoped>
  @import "../../assets/styles/global";
  .ebook-slide-contents {
    width: 100%;
    font-size: 0;
    .slide-contents-search-wrapper {
      display: flex;
      width: 100%;
      height: px2rem(36);
      margin: px2rem(20) 0 px2rem(10) 0;
      padding: 0 px2rem(15);
      box-sizing: border-box;
      .slide-contents-search-input-wrapper {
        flex: 1;
        border-radius: px2rem(3);
        @include center;
        .slide-contents-search-icon {
          flex: 0 0 px2rem(28);
          font-size: px2rem(12);
          @include center;
          .icon-search {
            font-size: px2rem(12);
          }
        }
        .slide-contents-search-input {
          flex: 1;
          width: 100%;
          height: px2rem(32);
          font-size: px2rem(14);
          background: transparent;
          border: none;
          &:focus {
          outline: none;
        }
        }
      }
      .slide-contents-search-cancel {
        flex: 0 0 px2rem(50);
        font-size: px2rem(14);
        @include right;
      }
    }
    .slide-contents-book-wrapper {
      display: flex;
      width: 100%;
      height: px2rem(90);
      padding: px2rem(10) px2rem(15) px2rem(20) px2rem(15);
      box-sizing: border-box;
      .slide-contents-book-img-wrapper {
        flex: 0 0 px2rem(45);
        box-sizing: border-box;
        .slide-contents-book-img {
          width: px2rem(45);
          height: px2rem(60);
        }
      }
      .slide-contents-book-info-wrapper {
        flex: 1;
        @include columnLeft
        .slide-contents-book-title {
          //  375*0.85 = 318.75 -30 =288.75-20=268.75-45=223.75-70=153.75
          // width: px2rem(153.75);
          font-size: px2rem(14);
          line-height: px2rem(16);
          padding: 0 px2rem(10);
          box-sizing: border-box;
          @include left
          .slide-contents-book-title-text {
            @include ellipse2(1)
          }
        }
        .slide-contents-book-author {
          //width: px2rem(153.75);
          font-size: px2rem(12);
          line-height: px2rem(14);
          padding: 0 px2rem(10);
          box-sizing: border-box;
          margin-top: px2rem(5);
          @include left
          .slide-contents-book-author-text {
            @include ellipse2(1)
          }
        }
      }
      .slide-contents-book-progress-wrapper {
        flex: 0 0 px2rem(70);
        @include columnLeft
        .slide-contents-book-progress {
          .progress {
            font-size: px2rem(14);
            line-height: px2rem(16);
          }
          .progress-text {
            font-size: px2rem(12);
            line-height: px2rem(14);
            margin-left: px2rem(2);
          }
        }
        .slide-contents-book-time {
          font-size: px2rem(12);
          line-height: px2rem(14);
          margin-top: px2rem(5);
        }
      }
    }
    .slide-contents-list {
      padding: 0 px2rem(15);
      box-sizing: border-box;
      .slide-contents-item {
        display: flex;
        padding: px2rem(20) 0;
        box-sizing: border-box;
        .slide-contents-item-label {
          flex: 1;
          font-size: px2rem(14);
          line-height: px2rem(16);
          @include ellipsis;
        }
        .slide-contents-item-page {
          flex: 0 0 px2rem(30);
          font-size: px2rem(10);
          @include right;
        }
      }
    }
    .slide-search-list {
      padding: 0 px2rem(15);
      box-sizing: border-box;
      .slide-search-item {
        font-size: px2rem(14);
        line-height: px2rem(16);
        padding: px2rem(20) 0;
        box-sizing: border-box;
      }
    }
  }
</style>
