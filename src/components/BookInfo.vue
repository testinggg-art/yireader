<template>
  <div class="bookInfo">
    <div class="bookCover">
      <router-link :to="detailRoute">
        <img :alt="info.name+'封面'" :src="info.coverUrl">
      </router-link>
    </div>
    <dl>
      <dt>
        <router-link class="name" :to="detailRoute">{{info.name}}<span class="redpoint" v-if="contentChanged" /></router-link>
        <span class="author">{{info.authorName}}</span>
      </dt>
      <dd>{{info.summary}}</dd>
      <b-button-group class="operate my-3" size="sm">
        <b-button class="mx-1" variant="danger" v-if="inBookshelf" @click="removeFromBookshelf">移出</b-button>
        <b-button class="mx-1" variant="secondary" v-if="inBookshelf" :disabled="!moveUpEnable" @click="moveUp">上移</b-button>
        <b-button class="mx-1" variant="secondary" v-if="inBookshelf" :disabled="!moveDownEnable" @click="moveDown">下移</b-button>
      </b-button-group>
      <div>
        <span v-if="info.readingChapter">云进度：<router-link :to="readingRoute">{{info.readingChapter.name}}</router-link></span>
      </div>
    </dl>
  </div>
</template>

<script>
export default {
  name: 'BookInfo',
  props: ['info', 'inBookshelf', 'index'],
  computed: {
    bookFullName() {
      return this.info.name + '-' + this.info.authorName;
    },
    moveUpEnable() {
      return this.inBookshelf && this.index > 0;
    },
    moveDownEnable() {
      return this.inBookshelf && this.index < this.$store.state.books.all.length - 1;
    },
    bookUserData() {
      return this.$store.getters.getBookUserData(this.info.id);
    },
    contentChanged() {
      return this.inBookshelf && this.info.contentChanged;
    },
    detailRoute() {
      if (!this.inBookshelf) {
        return {
          name:'BookDetail',
          params: {
            name: this.info.name,
            authorName: this.info.authorName
          },
          query: this.info,
        };
      }
      const bookUserData = this.bookUserData;
      let chapterIndex = 0;
      if (bookUserData) {
        chapterIndex = bookUserData.chapterIndex;
      }
      if (chapterIndex === -1) {
        chapterIndex = 0;
      }
      return {
        name: 'BookChapter',
        params: {
          bookId: this.info.id,
          chapterIndex: chapterIndex,
        },
        query: {
          name: this.info.name,
          authorName: this.info.authorName,
        }
      };
    },
    readingRoute() {
      if (!this.inBookshelf) {
        return {
          name:'BookDetail',
          params:{
            name: this.info.name,
            authorName: this.info.authorName
          },
          query: this.info
        };
      }
      return {
        name: 'BookChapter',
        params: {
          bookId: this.info.id,          
          chapterIndex: this.info.readingChapter.index,
        },
        query: {
          name: this.info.name,
          authorName: this.info.authorName,
        }
      };
    },
  },
  created() {
    console.log('BookInfo info', this.info);
    console.log('BookInfo inBookshelf', this.inBookshelf);
    console.log('BookInfo index', this.index);
  },
  methods: {
    removeFromBookshelf() {
      this.$store.dispatch('deleteBook', this.info.id);
    },
    moveUp() {
      this.$store.dispatch('moveUpBook', this.info.id);
    },
    moveDown() {
      this.$store.dispatch('moveDownBook', this.info.id);
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.bookInfo {
  margin: 10px 0 0;
  border: 1px solid black;
  display: flex;
  padding: 5px;
}
.bookInfo dl {
  flex: 9.5;
  margin: 10px 0 ;
}
.bookInfo dl dt {
  border-bottom: 1px dotted #A6D3E8;
}
.bookInfo dl dt .author {
  float: right;
  color: #999;
}
.redpoint{
  vertical-align: top;
  display:inline-block;
  height:10px;
  width:10px;
  border-radius:10px;
  background:red;
}
.bookInfo dl dd {
  padding: 7px 0 0 0;
  color: #999;
  margin: 0;
  line-height: 1.4;
  max-height: 4.4em;
  overflow-y: hidden;
}
.bookCover {
  flex: 2.5;
  margin-right: 10px;
}
.bookCover img {
  max-width: 100%;
}
</style>
