<template>
  <!-- app -->
  <div id="app" :class="{ 'results-mode': resultsMode }">
    <div class="search">
      <div class="search-inner">
        <input ref="searchField" v-model="state.keyword" :placeholder="currentInputState === 0 ? 'keyword' : 'Article ID'" @keyup.enter="search" @keyup.esc="reset"/>
        <span v-if="!resultsMode" class="switch-button" @click="switchInput">Switch</span>
        <transition
          v-on:enter="clearBtnEnter"
          v-on:leave="clearBtnLeave"
          v-bind:css="false"
          appear
        >
          <button class="btn btn-reset" v-if="state.resultsMode" @click="reset">&times;</button>
        </transition>
      </div>
    </div>

    <pulse-loader :loading="isLoading" :color="'#fff'" ></pulse-loader>

    <transition-group
      v-on:enter="enter"
      v-on:leave="leave"
      v-bind:css="false"
      appear
      tag="ul"
      mode="out-in"
      class="results">
      <li v-for="(result, index) in state.results" :key="index" :data-index="index" @click="changeWeb(result)">
        <a :href="result.link" target="_blank">
          <h3>{{ result.title }}</h3>
          <p>{{ result.excerpt }}</p>
        </a>
      </li>
    </transition-group>
  </div>
</template>

<script>
import $ from 'jquery';
import { TimelineMax, Back, Expo, Power4 } from "gsap";
import PulseLoader from 'vue-spinner/src/PulseLoader.vue'

export default {
  name: 'HelloWorld',
  data() {
    return {
      state: {
        apiResults: null,
        results: [],
        keyword: '',
        resultsMode: false
      },
      currentInputState: 0, // 0 -> keyword   1 -> article id
      isLoading: false,
      ENDPOINT: '//en.wikipedia.org/w/api.php'
    }
  },
  components: { PulseLoader },
  computed: {
    resultsMode() {
      return this.state.resultsMode;
    }
  },
  methods: {
    apiQuery() {
      // $.ajax({
      //   url: this.ENDPOINT,
      //   data: {
      //     action: 'opensearch',
      //     search: this.state.keyword,
      //     limit:10,
      //     namespace:0,
      //     format: 'json'
      //   },
      //   dataType: 'jsonp',
      //   success: res => {
      //     this.state.apiResults = res;
      //     this.results();
      //   }
      // });
      let apiRoute;
      if (this.currentInputState === 0) apiRoute = `http://localhost:5000/search/${this.state.keyword}`;
      else apiRoute = `http://localhost:5000/article/${this.state.keyword}`;
      this.isLoading = true;
      $.ajax({
        url: apiRoute,
        dataType: 'json',
        success: res => {
          this.isLoading = false;
          this.state.results = res;
          // this.results();
        }
      })
    },
    switchInput() {
      if (this.currentInputState === 0) this.currentInputState = 1;
      else this.currentInputState = 0;
    },
    results() {
      if (this.state.apiResults) {
        let obj = [];
        for (let i = 0; i < this.state.apiResults[1].length; i++) {
          obj.push({
            'title': this.state.apiResults[1][i],
            'summary': this.state.apiResults[2][i],
            'link': this.state.apiResults[3][i]
          });
        }
        this.state.results = obj;
      }
    },
    changeWeb(resultData) {
      this.$emit('changeWeb', resultData);
    },
    search() {
      this.state.results = [];
      this.state.resultsMode = true;
      this.state.apiResults = null;
      this.apiQuery();
    },
    reset() {
      this.state.resultsMode = false;
      this.state.results = null;
      this.state.apiResults = null;
      this.state.keyword = '';
      this.$refs.searchField.focus();
    },
    clearBtnEnter(el, done) {
      const tl = new TimelineMax({
				onComplete: done
			});
			tl.set(el, {
        opacity:0,
        rotation:-180,
        scale:0.2
			});
      setTimeout(() => {
        tl.to(el, 0.6, {
          opacity:1,
          rotation:0,
          scale:1,
          ease: Back.easeOut
        });
      }, 800);
    },
    clearBtnLeave(el, done) {
      const tl = new TimelineMax({
				onComplete: done
			});
      tl.to(el, 0.4, {
        opacity:0,
        rotation:180
      });
    },
    enter(el, done) {
      const a = $(el).find('a');
      const delay = el.dataset.index * 100;
			const tl = new TimelineMax({
				onComplete: done
			});
			tl.set(el, {
        perspective:100
			});
      tl.set(a, {
				transformStyle:'preserve-3d',
        rotationZ:45,
        rotationY:-45,
        rotationX:180,
        opacity:0,
        x:400
			});
      setTimeout(() => {
        tl.to(a, .5, {
          opacity:1,
          rotationZ:0,
          rotationY:0,
          rotationX:0,
          x: 0,
          ease: Expo.easeOut
        })
      }, delay);
    },
    leave(el, done) {
      const a = $(el).find('a');
			const tl = new TimelineMax({
				onComplete: done
			});
      tl.to(a, 0.4, {
        scale: 0.5,
        opacity:0,
        ease: Power4.easeOut
      });
    }
  },
  mounted() {
    this.$refs.searchField.focus();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
$c-gold: #FEDC3D;
$c-peach: #FEA680;
$c-azure: #02ACAB;

$c-1: #1E2649;
$c-1-l: #9B8DE5;
$c-2: #F66760;
$c-2-l: #F9A1A0;

$c-bg: $c-1;


@mixin placeholder($color) {
  &::-moz-placeholder {
    color:$color;
    opacity:1;
    transition:color .2s;
  }
  &:-ms-input-placeholder {
    color:$color;
    transition:color .2s;
  }
  &::-webkit-input-placeholder {
    color:$color;
    transition:color .2s;
  }
}

.switch-button {
  font-size: .5em;
  color: rgba(white, .5);
  cursor: pointer;
}

.search {
  position:fixed;
  top:0;
  left:0;
  width:100%;
  height:100%;
  transition:all .2s ease-in-out;
  font-size:40px;
  z-index:1;
  .results-mode & {
    height:100px;
    font-size:20px;
    transition-delay:0s;
    background:linear-gradient(to bottom, rgba($c-1,1), rgba($c-1,0));
  }
}
.search-inner {
  display:flex;
  height:100%;
  align-items:center;
  justify-content:center;
}
input {
  @include placeholder(rgba($c-1-l,.3));
  color:$c-2;
  background:none;
  padding:0;
  border:none;
  border-bottom:2px solid rgba($c-1-l,.2);
  padding:.5em 0;
  transition:border-color .2s;
  width: 35%;
  height: 50px;
  font-size: 30px;
  &:focus {
    outline:none;
    @include placeholder(rgba($c-1-l, .2));
  }
  .results-mode & {
    border-bottom-width:1px;
  }
}

.results {
  margin:0 auto;
  max-width:800px;
  padding:0;
  li {
    margin:0;
    list-style:none;
    cursor: pointer;
  }
  a {
    overflow:hidden;
    display:block;
    padding:2rem;
    text-decoration:none;
    position:relative;
    &:before {
      content:'';
      display:block;
      position:absolute;
      top:0;
      left:0;
      width:200%;
      height:100%;
      background:linear-gradient(to right, $c-1-l, $c-1-l 50%, rgba($c-1-l,0));
      transform:scaleX(.25);
      transition:all .5s;
      transform-origin:0 0;
      opacity:0;
    }
    h3, p {
      position:relative;
    }
    h3 {
      color:white;
    }
    p {
      color:rgba($c-1-l,.7);
    }
    &:hover {
      &:before {
        opacity:.4;
        transform:scaleX(1);
      }
    }
  }
}
.btn {
  background:none;
  border:none;
  line-height:1;
  display:flex;
  align-items:center;
  justify-content:center;
  text-align:center;
  color:rgba($c-1-l,.3);
  transition:color .2s;
  cursor:pointer;
  text-decoration:none;
  padding:5px;
  margin-left:1em;
  &:hover {
    color:rgba($c-1-l,.5);
  }
}
.btn-reset {
  font-size:1.5em;
  padding-top:0;
  margin-left:-1em;
  width:1em;
}
</style>
