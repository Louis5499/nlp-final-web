<template>
  <div class="wrapper">
    <span class="back" @click="backButton">返回</span>
    <h3 class="title">{{searchedData.title}}</h3>
    <span class="excerpt">{{searchedData.excerpt}}</span>
    <div class="divide-line"></div>
    <div class="select-banner">
      <div class="tf-button select-button" @click="currentTab = 0" :class="{ 'is-selected': (currentTab === 0) }">tf-idf</div>
      <div class="tf-button select-button" @click="currentTab = 1" :class="{ 'is-selected': (currentTab === 1) }">tf-idf + pf</div>
    </div>
    <div class="content-banner">
      <span class="second">「{{content[0][0]}}」</span>
      <span class="first">貼文內容關鍵字</span>
    </div>
    <div class="graph-row" v-show="currentTab === 0">
      <canvas id="graph-content" width="1000" height="400"></canvas>
    </div>
    <div class="graph-row" v-show="currentTab === 1">
      <canvas id="graph-content-pf" width="1000" height="400"></canvas>
    </div>
    <div class="keyword-set">
      <div v-for="(result, idx) in content" class="keyword-row" :key="idx">
        <span class="keyword-title">{{`${idx+1}: ${result[0]}`}}</span>
        <div class="keyword-related-set">
          <span v-for="(innerResult, innerIdx) in result[6]" class="keyword-content" :key="innerIdx">{{innerResult}}</span>
        </div>
      </div>
    </div>
    <p class="content">{{searchedData.content}}</p>
    <div class="divide-line"></div>
    <div class="content-banner">
      <span class="second">「{{comment[0][0]}}」</span>
      <span class="first">留言關鍵字</span>
    </div>
    <div class="graph-row" v-show="currentTab === 0">
      <canvas id="graph-comment" width="1000" height="400"></canvas>
    </div>
    <div class="graph-row" v-show="currentTab === 1">
      <canvas id="graph-comment-pf" width="1000" height="400"></canvas>
    </div>
    <div class="keyword-set">
      <div v-for="(result, idx) in comment" class="keyword-row" :key="idx">
        <span class="keyword-title">{{`${idx+1}: ${result[0]}`}}</span>
        <div class="keyword-related-set">
          <span v-for="(innerResult, innerIdx) in result[6]" class="keyword-content" :key="innerIdx">{{innerResult}}</span>
        </div>
      </div>
    </div>
  </div>  
</template>

<script>
import Chart from 'chart.js';
import Palette from 'google-palette';

export default {
  data() {
    return {
      currentTab: 0 // 0 -> tf-idf    1 -> tf-idf + pf
    }
  },
  props: {
    searchedData: {
      type: Object
    }
  },
  methods: {
    backButton() {
      this.$emit('back', 0);
    }
  },
  computed: {
    content() {
      return this.currentTab === 0 ? this.searchedData.keyword_content : this.searchedData.keyword_content_pf;
    },
    comment() {
      return this.currentTab === 0 ? this.searchedData.keyword_comments : this.searchedData.keyword_comments_pf;
    }
  },
  mounted() {
    // tidy data
    console.log(this.searchedData);
    const commentKeywordsData = [];
    const commentKeywordsLabel = [];
    const contentKeywordsData = [];
    const contentKeywordsLabel = [];
    for (const comment of this.searchedData.keyword_comments) {
      commentKeywordsData.push(comment[1]);
      commentKeywordsLabel.push(comment[0]);
    }
    for (const content of this.searchedData.keyword_content) {
      contentKeywordsData.push(content[1]);
      contentKeywordsLabel.push(content[0]);
    }

    const contentColorArr = Palette('tol-rainbow', contentKeywordsData.length).map(function(hex) {
      return '#' + hex;
    })

    const colorArr = Palette('tol-rainbow', commentKeywordsData.length).map(function(hex) {
      return '#' + hex;
    })

    const ctxContent = document.getElementById('graph-content');
    new Chart(ctxContent, {
      type: 'doughnut',
      data: {
        datasets: [{
          data: contentKeywordsData,
          backgroundColor: contentColorArr.reverse(),
        }],
        labels: contentKeywordsLabel
      },
      options: {
        responsive: false
      }
    });

    const ctxComment = document.getElementById('graph-comment');
    new Chart(ctxComment, {
      type: 'doughnut',
      data: {
        datasets: [{
          data: commentKeywordsData,
          backgroundColor: colorArr.reverse(),
        }],
        labels: commentKeywordsLabel
      },
      options: {
        responsive: false
      }
    });


    // pf
    const commentKeywordsDataPf = [];
    const commentKeywordsLabelPf = [];
    const contentKeywordsDataPf = [];
    const contentKeywordsLabelPf = [];
    for (const comment of this.searchedData.keyword_comments_pf) {
      commentKeywordsDataPf.push(comment[1]);
      commentKeywordsLabelPf.push(comment[0]);
    }
    for (const content of this.searchedData.keyword_content_pf) {
      contentKeywordsDataPf.push(content[1]);
      contentKeywordsLabelPf.push(content[0]);
    }

    const contentColorArrPf = Palette('tol-rainbow', contentKeywordsDataPf.length).map(function(hex) {
      return '#' + hex;
    })

    const colorArrPf = Palette('tol-rainbow', commentKeywordsDataPf.length).map(function(hex) {
      return '#' + hex;
    })

    const ctxContentPf = document.getElementById('graph-content-pf');
    new Chart(ctxContentPf, {
      type: 'doughnut',
      data: {
        datasets: [{
          data: contentKeywordsDataPf,
          backgroundColor: contentColorArrPf.reverse(),
        }],
        labels: contentKeywordsLabelPf
      },
      options: {
        responsive: false
      }
    });

    const ctxCommentPf = document.getElementById('graph-comment-pf');
    new Chart(ctxCommentPf, {
      type: 'doughnut',
      data: {
        datasets: [{
          data: commentKeywordsDataPf,
          backgroundColor: colorArrPf.reverse(),
        }],
        labels: commentKeywordsLabelPf
      },
      options: {
        responsive: false
      }
    });
  }
}
</script>

<style lang="scss" scoped>
$c-gold: #FEDC3D;
$c-peach: #FEA680;
$c-azure: #02ACAB;

$c-1: #1E2649;
$c-1-l: #9B8DE5;
$c-2: #F66760;
$c-2-l: #F9A1A0;

$c-bg: $c-1;

.wrapper {
  width: 100%;
  height: 100%;
}
// #graph {
//   width: 200px;
//   height: 200px;
// }
.content-banner {
  font-weight: 500;
  display: flex;
  flex-direction: row;
  width: 100%;
  padding-left: 50px;
  box-sizing: border-box;
  align-items: baseline;
  justify-content: center;
  margin: 0 0 30px 0;
  .first {
    color: rgba(white, .6);
    margin-left: 20px;
  }
  .second {
    color: $c-peach;
    font-size: 3em;
  }
}
.back {
  color: rgba(white, .7);
  display: flex;
  padding-left: 50px;
  cursor: pointer;
}
.title {
  color: white;
  display: flex;
  padding-left: 50px;
  font-size: 2em;
}
.excerpt {
  color: white;
  display: flex;
  padding-left: 50px;
  text-align: left;
}
.divide-line {
  background-color: rgba($c-gold, .5);
  width: 70%;
  height: 2px;
  margin: 20px 15%;
}
.keyword-set {
  display: flex;
  flex-direction: row;
  overflow-x: scroll;
  width: 100%;
  flex-wrap: wrap;
  justify-content: center;
  .keyword-row {
    width: 280px;
    display: block;
    font-size: 1.2em;
    margin: 20px 0;
    .keyword-title {
      color: $c-2;
      font-weight: 500;
      display: flex;
    }
    .keyword-related-set {
      width: 100%;
      text-align: left;
      display: flex;
      flex-direction: column;
      margin: 10px 0;
      font-size: .8em;
    }
    .keyword-content {
      margin: 0 10px 0 0;
      color: rgba(white, .7);
    }
  }
}
.content {
  width: 100%;
  color: white;
  padding: 0 20px;
  box-sizing: border-box;
  text-align: left;
  line-height: 30px;
}
.graph-row {
  display: flex;
  flex-direction: row;
  justify-content: center;
}
.select-banner {
  display: flex;
  flex-direction: row;
  height: 60px;
  align-items: center;
  justify-content: center;
  .select-button {
    color: rgba(white, .5);
    font-weight: 500;
    margin: 0 20px;
    font-size: 1.5em;
    cursor: pointer;
    &:hover {
      color: rgba(white, 1);
    }
    &.is-selected {
      color: rgba($c-gold, .9);
    }
  }
}
</style>