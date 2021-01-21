<template>
  <div class="hello">
    <div v-if="!loginSuccess">
      <span> 房间号:<a-input type="text" v-model:value="roomId" /> </span>
      <br />
      <span> 昵称:<a-input type="text" v-model:value="username" /> </span>
      <br />
      <a-button @click="addNum">进入</a-button>
    </div>
    <div v-else>
      <div v-if="!dealed">
        <span v-for="(item, key) in members" :key="key">{{ key }},</span>
      </div>

      <a-button v-if="readyTodeal && members[username].owner" @click="Groups"
        >分组</a-button
      >
      <div
        v-if="grouping"
        style="width: 400px;
    position: absolute;
    left: calc(50% - 200px);"
      >
        <a-layout>
          <a-layout-header>
            <a-button @click="change">
              {{ orders[0] }}
            </a-button>
          </a-layout-header>
          <a-layout>
            <a-layout-sider>
              <a-button @click="change">
                {{ orders[3] }}
              </a-button>
            </a-layout-sider>
            <a-layout-content></a-layout-content>
            <a-layout-sider>
              <a-button @click="change">
                {{ orders[1] }}
              </a-button>
            </a-layout-sider>
          </a-layout>
          <a-layout-footer>
            <a-button @click="change">
              {{ orders[2] }}
            </a-button>
          </a-layout-footer>
        </a-layout>
      </div>
      <a-button v-if="readyTodeal && members[username].owner" @click="startdeal"
        >start</a-button
      >
      <div style="position:absolute;width:100%;height:100%" v-if="dealed">
        <a-layout>
          <a-layout-header>
            <span>{{ orders[(selfIndex + 4 - 2) % 4] }}</span>
            <span v-if="orders[(selfIndex + 4 - 2) % 4] == turning"
              >asdasd</span
            >
          </a-layout-header>
          <a-layout>
            <a-layout-sider>
              {{ orders[(selfIndex + 4 - 1) % 4] }}
              <span v-if="orders[(selfIndex + 4 - 1) % 4] == turning"
                >asdasd</span
              >
            </a-layout-sider>
            <a-layout-content></a-layout-content>
            <a-layout-sider>
              {{ orders[(selfIndex + 4 - 3) % 4] }}
              <span v-if="orders[(selfIndex + 4 - 3) % 4] == turning"
                >asdasd</span
              >
            </a-layout-sider>
          </a-layout>
          <a-layout-footer>
            <div>
              <a-button
                v-if="turning == username && lastTurning != username"
                @click="passcards"
                >pass</a-button
              >
              <a-button v-if="turning == username" @click="playcards"
                >play</a-button
              >
            </div>
            <div
              id="cards"
              style="display:flex;justify-content:center"
              @mousedown="startChoose"
              @mouseup="endChoose"
              @touchstart="startChoose"
              @touchmove="chooseCards"
              @touchend="endChoose"
            >
              <img
                v-for="(item, key) in cards"
                :key="key"
                src="'cards/'+item.type+'-'+item.value+'.jpg'"
                alt=""
                :style="changeStyles(key)"
                @mousedown="chooseSingle1"
                @mouseup="chooseSingle2"
                :data-index="key"
              />
            </div>
          </a-layout-footer>
        </a-layout>
      </div>
      <div v-if="readyToNext">
        <a-button @click="next">next</a-button>
      </div>
    </div>
    <div></div>
  </div>
</template>

<script>
//'cards/'+item.type+'-'+item.value+'.jpg'

// import {  ref } from "vue";
import io from 'socket.io-client';
export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  data() {
    return {
      socketReady: false,
      roomId: '1111',
      username: '',
      loginSuccess: false,
      members: [],
      cards: [],
      grouping: false,
      readyTodeal: false,
      orders: [],
      changing: false,
      dealed: false,
      choosedCards: {},
      cardsMap: {},
      playedCards: {},
      lastCards: { username: undefined, cards: [] },
      turning: '',
      lastTurning: '',
      finished: false,
      current: 3,
      readyToNext: false,
      needTogong: false,
      gongs: [],
    };
  },
  //    setup(props,context) {
  //     const number = ref(0);
  //  //   const username= ref('')
  //     setInterval(()=>{
  //       number.value++;
  //     },1000);
  //     console.log('setup::::',props,context);
  //     return {
  //       number,
  //    //   username
  //     };
  //   },

  created() {
    var sUserAgent = navigator.userAgent.toLowerCase();
    if (
      /ipad|iphone|midp|rv:1.2.3.4|ucweb|android|windows ce|windows mobile/.test(
        sUserAgent
      )
    ) {
      console.log('mobile');
    } else {
      console.log('PC');
    }

    this.socket = io('http://localhost:3000/test');
    this.socket.on('queryed', (val) => {
      if (val) this.socketReady = true;
    });
    this.socket.on('newMember', (val) => {
      console.log('message:::::', val);
      this.members = val;
    });
    this.socket.on('dealCards', (val) => {
      this.readyToNext = false;
      this.playedCards = {};
      this.choosedCards = {};
      this.cards = val.cards.sort((a, b) => a.value - b.value);
      this.cardsMap = {};
      if (val.orders) {
        this.orders = val.orders;
      }
      this.orders.forEach((ele, index) => {
        if (ele == this.username) {
          this.selfIndex = index;
        }
      });
      this.cards.forEach((ele, index) => {
        this.cardsMap[index] = ele;
        ele._index = index;
      });
      this.dealed = true;
      this.turning = val.turning;
      this.lastTurning = val.lastTurning;
      if (val.gong) {
        if (val.gong.includes(this.username)) {
          this.needTogong = true;
          let card = this.cards[this.card.length - 1];
          this.gongs = this.cards.filter((ele) => {
            return ele.value == card.value;
          });
        }
      }
    });
    this.socket.on('playedcards', (val) => {
      this.turning = val.turning;
      this.lastTurning = val.lastTurning;
      console.log(val.turning, val.lastTurning, this.username);
      if (!this.finished) {
        console.log(val);
      } else if (val.turning == this.username) {
        if (val.turning == val.lastTurning) {
          this.socket.emit('jump', {
            roomId: this.roomId,
            order: this.selfIndex,
          });
        } else {
          this.socket.emit('play', {
            cards: null,
            username: this.username,
            roomId: this.roomId,
          });
        }
      }
    });
    this.socket.on('end', (val) => {
      this.current = val.current;
      this.readyToNext = true;
    });
    //      console.log('::::::',this.socket)
    this.username =
      '' +
      Math.floor(Math.random() * 10) +
      Math.floor(Math.random() * 10) +
      Math.floor(Math.random() * 10) +
      Math.floor(Math.random() * 10);
  },
  beforeCreate() {},
  watch: {
    socketReady: function() {},
    members: function(value) {
      if (Object.keys(value).length == 4) {
        this.readyTodeal = true;
      }
      this.orders = Object.keys(value);
    },
  },
  methods: {
    changeStyles(index) {
      let ifplay = this.playedCards[index];
      if (ifplay) {
        return { display: 'none' };
      }
      let ifchoose = this.choosedCards[index];
      if (ifchoose) {
        return index == 0
          ? {
              width: '50px',
              marginTop: '-10px',
            }
          : {
              width: '50px',
              marginLeft: '-38px',
              marginTop: '-10px',
            };
      }
      return index == 0
        ? {
            width: '50px',
          }
        : {
            width: '50px',
            marginLeft: '-38px',
          };
    },
    Groups: function() {
      this.grouping = true;
    },
    addNum: function() {
      if (!this.username || this.username.length < 1) {
        return;
      }
      if (!this.roomId || this.roomId.length < 1) {
        return;
      }
      //    console.log( this.username,this.roomId,this.socketReady)
      this.socket.emit(
        'login',
        { roomId: this.roomId, username: this.username },
        (val) => {
          if (val) {
            this.loginSuccess = true;
          }
        }
      );
      //  this.socket.emit('start')
    },
    startdeal: function() {
      this.grouping = false;
      this.readyTodeal = false;
      this.dealed = true;
      this.socket.emit('start', {
        username: this.username,
        roomId: this.roomId,
        orders: this.orders,
      });
    },
    change(val) {
      if (!this.changing) {
        this.changing = true;
        this.changingName = val.target.innerText;
        return;
      } else {
        if (val.target.innerText == this.changingName) {
          return;
        }
        let a, b;
        this.orders.forEach((ele, index) => {
          if (ele == this.changingName) a = index;
        });
        this.orders.forEach((ele, index) => {
          if (ele == val.target.innerText) b = index;
        });
        this.orders[a] = val.target.innerText;
        this.orders[b] = this.changingName;
        this.changing = false;
      }
    },
    startChoose() {
      this.choosing = true;
      //  console.log('::start', e);
    },
    chooseCards(e) {
      if (this.choosing) {
        console.log('::ing', e);
      }
    },
    endChoose() {
      //   console.log('endChoose');
      this.choosing = false;
    },
    chooseSingle1(e) {
      e.stopPropagation();
    },
    chooseSingle2(e) {
      e.stopPropagation();
      let index = e.target.dataset.index;
      //  console.log(this.choosedCards);
      let ifchoosed = this.choosedCards[index];
      if (ifchoosed) {
        delete this.choosedCards[index];
      } else {
        this.choosedCards[index] = this.cardsMap[index];
      }
    },
    playcards() {
      if (Object.keys(this.choosedCards).length < 1) {
        console.log('need one at least');
        return;
      }
      this.playedCards = Object.assign(this.playedCards, this.choosedCards);
      if (Object.keys(this.playedCards).length > 26) {
        this.finished = true;
        this.socket.emit('play', {
          cards: this.choosedCards,
          username: this.username,
          roomId: this.roomId,
          finished: true,
        });
      } else {
        this.socket.emit('play', {
          cards: this.choosedCards,
          username: this.username,
          roomId: this.roomId,
        });
      }
      this.choosedCards = {};
    },
    passcards() {
      this.socket.emit('play', {
        cards: null,
        username: this.username,
        roomId: this.roomId,
      });
    },
    next() {
      this.readyToNext = false;
      this.playedCards = {};
      this.choosedCards = {};
      this.socket.emit('next', { roomId: this.roomId });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.ant-layout {
  text-align: center;
}
.ant-layout-header,
.ant-layout-footer {
  background: #7dbcea;
  color: #fff;
}
.ant-layout-footer {
  line-height: 1.5;
}
.ant-layout-sider {
  background: #3ba0e9;
  color: #fff;
  line-height: 120px;
}
.ant-layout-content {
  background: rgba(16, 142, 233, 1);
  color: #fff;
  min-height: 120px;
  line-height: 120px;
}

.ant-layout:last-child {
  margin: 0;
}
</style>
