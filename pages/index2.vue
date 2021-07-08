<template>
  <div class="container">
    <nav class="nav">
      <ul>
        <li><nuxt-link to="/">xml2js</nuxt-link></li>
        <li><nuxt-link to="/index2">fast-xml-parser</nuxt-link></li>
      </ul>
    </nav>
    <h1>ぶろぐ更新情報(@nuxt/axios + fast-xml-parser)</h1>
    <ArticleList :contents="contents"></ArticleList>
  </div>
</template>

<script>
const parser = require("fast-xml-parser");
const he = require('he');
const options = {
    attributeNamePrefix : "@_",
    attrNodeName: "attr", //default is 'false'
    textNodeName : "#text",
    ignoreAttributes : false,
    ignoreNameSpace : false,
    allowBooleanAttributes : false,
    parseNodeValue : true,
    parseAttributeValue : false,
    trimValues: true,
    cdataTagName: "__cdata", //default is 'false'
    cdataPositionChar: "\\c",
    localeRange: "", //To support non english character in tag/attribute values.
    parseTrueNumberOnly: false,
    attrValueProcessor: a => he.decode(a, {isAttributeValue: true}),//default is a=>a
    tagValueProcessor : a => he.decode(a) //default is a=>a
};

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

          // 方法1 XMLのバリデーション後に変換
          if( parser.validate(data) === true) { //optional (it'll return an object in case it's not valid)
            var jsonData = parser.parse(data, options);
          }

          // 方法2 中間オブジェクトを挟んでconvertToJsonメソッドで変換
          // var tObj = parser.getTraversalObj(data, options);
          // var jsonData = parser.convertToJson(tObj, options);

          // transformResponseの実行結果を戻す
          return jsonData;
        },
      ],
    };

    // XMLから記事データを読み込む＆Vue内で使えるようにデータをreturnする
    // （asyncData関数の外にデータを出して、templateタグ内やscriptタグ内で扱えるようにする）
    return await $axios
      .get(url, config)
      .then((response) => {
        // console.log(response)
        console.log(response.data["rdf:RDF"].item)
        // console.log(JSON.stringify(response.data["rdf:RDF"].item, null, 2));
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
