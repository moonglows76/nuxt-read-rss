<template>
  <div class="container">
    <h1>ぶろぐ更新情報</h1>
    <div class="article-list">
      <article
        v-for="content in limitCount(10)"
        :key="content.id"
        class="article-list__item"
      >
        <header class="article-list__header">
          <h2 class="article-list__title">
            <a :href="deleteNewlineCharacter(content.link)" target="_blank">
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
    </div>
  </div>
</template>

<script>
const xml2js = require("xml2js");

export default {
  async asyncData({ $axios }) {
    const url = "/blog.xml";
    // axios用の設定
    const config = {
      // responseType: 'document', 'document'はブラウザ環境以外ではtextと同じ
      // transformResponse: axiosでデータを取得したら下記の内容を実行。
      // transformResponseでは非同期処理は扱えないので、変換処理が同期的に実行できるモジュール（xml2jsなど）を選ぶ必要がある
      transformResponse: [
        (data) => {
          let jsonData;
          // XMLをJSONに変換するオブジェクトのインスタンスを作成
          // async: 非同期かどうか
          // explicitArray: trueの場合、常に子ノードを配列に配置。それ以外の場合、配列は複数ある場合にのみ作成
          const parser = new xml2js.Parser({
            async: false,
            explicitArray: false,
          });
          // 変換を実行
          // data: 取得したxml
          // json: 変換結果(jsonデータ)
          parser.parseString(data, (error, json) => {
            jsonData = json;
          });
          // transformResponseの実行結果を戻す
          return jsonData;
        },
      ],
    };
    return await $axios
      .get(url, config)
      .then((response) => {
        // console.log(response.data["rdf:RDF"].item)
        // console.log(JSON.stringify(response.data["rdf:RDF"].item, null, 2));
        // console.log(response.data)
        // console.log( JSON.stringify(response.data, null, 2) )
        return { contents: response.data["rdf:RDF"].item };
      })
      .catch((e) => console.error(e));
  },
  methods: {
    // 不要な改行文字を削除
    deleteNewlineCharacter(text) {
      return text.replace(/\r?\n/g, "");
    },
    // リストで表示する件数を調整
    limitCount(number) {
      return this.contents.slice(0, number);
    },
  },
};
</script>

<style>
.container {
  margin: 0 auto;
  max-width: 960px;
}
h1, h2 {
  font-weight: normal;
  font-size: 1rem;
}
.article-list {
}
.article-list__item {
  margin-top: 2rem;
}
</style>
