<template>
  <div id="appContainer">
    <demo />
  </div>
</template>

<script>
//import HelloWorld from "./components/HelloWorld.vue";
import demo from './components/demo.vue';
export default {
  name: 'App',
  components: {
    //  HelloWorld,
    demo,
  },
  created() {
    const evt =
      'onorientationchange' in window ? 'orientationchange' : 'resize';
    window.addEventListener(evt, this.resize, false);
  },
  mounted() {
    this.resize();
  },
  methods: {
    resize: function() {
      const width = document.documentElement.clientWidth;
      const height = document.documentElement.clientHeight;
      const contentDOM = document.getElementById('appContainer');
      let style = ''
      console.log(width, height);
  

      if (width >= height) {
        // 横屏
        style += 'width:' + width + 'px;'; // 注意旋转后的宽高切换
        style += 'height:' + height + 'px;';
        style += '-webkit-transform: rotate(0); transform: rotate(0);';
        style += '-webkit-transform-origin: 0 0;';
        style += 'transform-origin: 0 0;';
      } else {
        // 竖屏
        style += 'width:' + height + 'px;';
        style += 'height:' + width + 'px;';
        style += '-webkit-transform: rotate(90deg); transform: rotate(90deg);';
        // 注意旋转中点的处理
        style +=
          '-webkit-transform-origin: ' + width / 2 + 'px ' + width / 2 + 'px;';
        style += 'transform-origin: ' + width / 2 + 'px ' + width / 2 + 'px;';
      }
      contentDOM.style.cssText = style;
    },
  },
};
</script>

<style>
* {
  margin: 0px 0px;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  position: absolute;
  height: 100%;
  width: 100%;
}
#appContainer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
/* @media screen and (orientation: portrait) {

  #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  transform : rotate(-90deg);
  transform-origin : 50% 50%;
}
} 
@media screen and (orientation: landscape) {
  #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: aliceblue;
    position: absolute;
  height: 100%;
  width: 100%;
}
} */
</style>
