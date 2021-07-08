<template>
  <div class="container">
    <nav class="nav">
      <ul>
        <li><nuxt-link to="/">xml2js</nuxt-link></li>
        <li><nuxt-link to="/index2">fast-xml-parser</nuxt-link></li>
      </ul>
    </nav>
    <h1>ぶろぐ更新情報(@nuxt/axios + fast-xml-parser)</h1>
    <div class="article-list">
      <article
        v-for="content in limitCount(limitNumber)"
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
      <!-- 次の10件表示ボタン　100件表示したら非表示になる -->
      <button
        type="button"
        class="article-list__btn-next10"
        @click="showNext10Articles()"
        v-if="limitNumber < 100 && $mq === 'sm'"
      >次の10件を表示する</button>
    </div>
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
  data(){
    return {
      limitNumber: 10, // 表示記事の件数
    }
  },
  methods: {
    // 不要な改行文字を削除
    deleteNewlineCharacter(text) {
      return text.replace(/\r?\n/g, "");
    },
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
    },
  },
};
</script>

<style>
.container {
  margin: 0 auto;
  max-width: 960px;
}
.nav ul {
  margin: 2rem 0;
  padding: 0;
  list-style-type: none;
  display: flex;
}
.nav li {
  margin-right: 2rem;
}
h1, h2 {
  font-weight: normal;
  font-size: 1rem;
}
.article-list {
  padding: 0 1rem 3rem;
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
