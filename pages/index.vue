<template>
  <div class="container">
    <nav class="nav">
      <ul>
        <li><nuxt-link to="/">xml2js</nuxt-link></li>
        <li><nuxt-link to="/index2">fast-xml-parser</nuxt-link></li>
      </ul>
    </nav>
    <h1>ぶろぐ更新情報(@nuxt/axios + xml2js)</h1>
    <ArticleList :contents="contents"></ArticleList>
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
            trim: true,
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
};
</script>

<style>
@import '~assets/css/style.css';
</style>
