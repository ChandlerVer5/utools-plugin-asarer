<template>
  <div class="file-browser-root">
    <!-- 倒三角指示器 -->
    <div v-if="false" style="width: 0; height: 0; margin: 0 auto">
      <span
        style="
          display: block;
          background: linear-gradient(
            135deg,
            rgb(255, 255, 255) 0%,
            rgb(238, 238, 238) 60%,
            rgba(238, 238, 238, 0) 60%,
            rgba(238, 238, 238, 0) 100%
          );
          margin: 0px 0 -6px;
          top: -6px;
          width: 12px;
          height: 12px;
          border-radius: 4px 0px 0px;
          border-width: 1px 0px 0px 1px;
          border-style: solid;
          border-color: rgb(204, 204, 204);
          border-image: initial;
          box-shadow: rgba(50, 50, 93, 0.05) -3px -3px 5px;
          transform: rotate(45deg);
          position: relative;
        "
      />
    </div>

    <div
      style="
        background: #fff;
        border-radius: 4px;
        box-shadow: rgba(50, 50, 93, 0.1) 0px 50px 100px, rgba(50, 50, 93, 0.15) 0px 15px 35px,
          rgba(0, 0, 0, 0.1) 0px 5px 15px;
      "
    >
      <div class="file-browser">
        <SearchList
          v-if="search"
          :files-list="allFullPaths"
          :need-search="search"
          @item-click="whenClickIt($event)"
          @show-result="hasChar($event)"
        ></SearchList>
        <!-- main part 这里的 v-if 真的恐怖！-->
        <ul v-show="showList" class="fb-wrap-viewer" @scroll="onScroll">
          <Fileslist
            :dir="pkgName"
            :list="filesData"
            @item-click="whenClickIt($event)"
            @dom-mounted="allFullPaths = $event"
          />
        </ul>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
// test :https://runkit.com/api/npm/info/mocha?version=2.6.0
import { defineComponent } from 'vue';
import Fileslist from './FilesList.vue';
import SearchList from './SearchList.vue';

interface Dir {
  name: string;
  type: 'file' | 'directory';
  files?: Dir[];
  [key: string]: any;
}

export default defineComponent({
  name: 'FileBrowser',
  components: {
    Fileslist,
    SearchList,
  },
  props: {
    pkgName: {
      type: String,
      default: '',
    },
    search: {
      type: Boolean,
      default: false,
    },
    directoryListing: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      filesData: {} as Dir, // 转换过之后的数据
      showList: true, // 显示文件列表
      allFullPaths: [], // 获得所有路径数组
      scrollEL: this.$el,
      wrapperEl: this.$el,
    };
  },
  created() {
    // 构造数据！
    this.directoryListing
      ? (this.filesData = {
          name: this.pkgName, // give a name
          type: 'directory',
          files: this.deepTrans(this.directoryListing),
        })
      : console.error('FileBrowser must have a directory-listing prop for DATA!!!');
  },
  mounted() {
    this.wrapperEl = this.$el.querySelector('.browser-top-container') as HTMLElement;
  },
  methods: {
    whenClickIt(path: string) {
      this.$emit('get-path', path);
    },
    filesAdd(arr: Array<any>) {
      const tempArr: any[] = [];
      for (let i = 0; i < arr.length; i++) {
        // const item = arr[i];
        tempArr.push({
          name: arr[i],
          type: 'file',
          files: null,
        });
      }
      return tempArr;
    },
    // 非字符即数组
    deepTrans(arr: Array<any>) {
      // 如果数据已经满足格式要求，直接返回
      if (arr[0].type && arr[0].name) return arr;

      let tempArr: any[] = [];
      // const Flist = FileBrowser.list;

      if (typeof arr[0] === 'string' && typeof arr[1] !== 'object') {
        tempArr = this.filesAdd(arr);
        return tempArr;
      }

      // 数组包含数组情况
      if (typeof arr[0] !== 'string') {
        for (let i = 0; i < arr.length; i++) {
          if (typeof arr[i] !== 'string') {
            const data = this.deepTrans(arr[i]);
            tempArr.push({
              name: arr[i][0],
              type: 'directory',
              files: data[0]['files'],
            });
          } else {
            // 前面去那是文件夹形式，后面是文件形式
            tempArr.push({
              name: arr[i],
              type: 'file',
              files: null,
            });
            // }
          }
        }
      }
      // 递归
      if (typeof arr[0] === 'string' && typeof arr[1] !== 'string') {
        const data = this.deepTrans(arr[1]);
        tempArr.push({
          name: arr[0],
          type: 'directory',
          files: data,
        });
      }

      return tempArr;
    },

    hasChar(hasChar: boolean) {
      this.showList = !hasChar; // 搜索框有子符，那么不显示文件列表

      this.$nextTick(() => {
        // 切换之后，DOM渲染完成之后操作
        this.scrollEL && this.isScrolled(this.scrollEL);
      });
    },
    // 监听 scroll 添加样式
    isScrolled({ scrollTop }: HTMLElement) {
      if (scrollTop > 0) {
        this.wrapperEl.classList.add('scroll');
      } else {
        this.wrapperEl.classList.remove('scroll');
      }
    },

    // 监听 scroll
    onScroll({ target }: Event) {
      this.scrollEL = target as HTMLElement;
      this.isScrolled(this.scrollEL);
    },
  },
});
</script>

<style scoped lang="scss">
@import './font/fontawesome-webfont.css';
.file-browser-root {
  ul {
    list-style-type: none;
    margin: 0;
    padding: 0;
    li {
      padding: 0;
      margin: 0;
    }
  }
  top: 10px;
  left: 14px;
  width: 100%;
  margin: 0px;
  font-size: 14px;
  z-index: 100;
}
.file-browser {
  height: auto;
  z-index: 100;
}
.fb-wrap-viewer {
  position: relative;
  width: 100%;
}
</style>
