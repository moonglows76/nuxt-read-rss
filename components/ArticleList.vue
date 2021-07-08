<template>
  <div class="article-list">
    <article
      v-for="content in limitCount(limitNumber)"
      :key="content.id"
      class="article-list__item"
    >
      <header class="article-list__header">
        <h2 class="article-list__title">
          <a :href="content.link" target="_blank">
            {{ content.title }}
          </a>
        </h2>
        <div class="article-list__date">- {{ $dayjs(content["dc:date"]).utc().tz('Asia/Tokyo').format('YYYY年MM月DD日 HH:mm') }}</div>
      </header>
      <div class="article-list__publisher">
        <p>
          from<a :href="content['dc:identifier']" target="_blank"
            >{{ content["dc:publisher"] }}</a
          >
        </p>
      </div>
    </article>
    <!-- 次の10件表示ボタン　100件表示したら非表示になる -->
    <button
      type="button"
      class="article-list__btn-next10"
      @click="showNext10Articles()"
      v-if="limitNumber < 100 && $mq === 'sm'"
    >次の10件を表示する</button>
  </div>
</template>

<script>
export default {
  props: {
    contents: { type: Array },
  },
  data(){
    return {
      limitNumber: 10, // 表示記事の件数
    }
  },
  methods: {
    // SPとPCで表示を切り替え
    // リストで表示する件数を調整
    limitCount(number) {
      // PC($mq==='md')は強制的に100件表示、SP($mq==='sm')は引数numberに指定された件数表示
      if( this.$mq === 'md') {
        number = 100
      }
      return this.contents.slice(0, number);
    },
    // 表示記事を10件追加
    showNext10Articles() {
      this.limitNumber += 10
    }
  },
}
</script>

<style>
.article-list {
  padding: 0 1rem 3rem;
}
.article-list__title {
  font-weight: normal;
  font-size: 1rem;
}
.article-list__item {
  margin-top: 2rem;
}
.article-list__btn-next10 {
  display: block;
  width: 75%;
  margin: 2rem auto 0;
  padding: .5rem;
  font-size: 1rem;
  background-color: #ffffff;
  border-radius: 5px;
  color: #17c985;
  border: 2px solid currentColor;
  font-weight: bold;
  cursor: pointer;
}
</style>
