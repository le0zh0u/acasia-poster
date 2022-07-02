<template>
  <el-row type="flex" class="poster-container" justify="center">
    <el-col :span="12">
      <div id="poster-preview" :style="{ 'cursor': 'default' }">
        <div class="bg"></div>
        <img class="poster-template" :src="bgUrl">
        <div v-for="podcastLogoImage in podcastLogoImageList" v-if="podcastLogoImageList.length > 0"
          class="podcast-logo-img"
          :style="{ 'left': podcastLogoImage.left + 'vh', 'top': podcastLogoImage.top + 'vh' }">
          <img :src="podcastLogoImage.logoPos" ref="avatar"
            :style="{ 'width': podcastLogoImage.width + 'vh', 'height': podcastLogoImage.height + 'vh', 'left': podcastLogoImage.left + 'vh', 'top': podcastLogoImage.top + 'vh' }">
          <p
            :style="{ 'text-align': 'center', 'width': podcastLogoImage.width + 'vh', 'color': 'white', 'margin-top': '0.1em', 'font-size': '1.7vh' }">
            {{ podcastLogoImage.name }}</p>
        </div>
        <div class="poster-content">
          <div class="topic" :style="{ 'font-size': 3.5 * topicFontSize + 'vh' ,'padding-left':'3vh','padding-right':'3vh' }">{{ topic }}</div>
          <div v-if="topicDescList.length > 0 && topicDescList[0].value != ''"
            v-for="(topicDesc, index) in topicDescList" :key="index" class="sub-topic"
            :style="{ 'font-size': 2.5 * topicDescFontSize + 'vh' }">{{ topicDesc.value }}</div>
          <div class="time">{{ time }}</div>
          <!-- :auto-color="true" -->
<!-- :logo-src="" :style="{ 'padding-left': 0 + 'vh', 'padding-right': 0 + 'vh', 'background-color': 'rgba(255, 255, 255, 0.6)', 'height': liveRoomQRPannelHeight + 'vh' }" -->
          <div class="entire-live-room-content"
            >
            <div v-for="liveRoom in entireLiveRoomQRList" v-if="entireLiveRoomQRList.length > 0"
              class="podcast-logo-img" :style="{ 'left': liveRoom.left + 'vh' , 'bottom': liveRoom.bottom + 'vh', 'background-color': 'rgba(255, 255, 255, 0.7)' }">
              <vue-qr 
                  :margin="10"
                  :dot-scale="1"
                  :text="liveRoom.text"
                  :logo-src = "liveRoom.logoSrc"
                   :style="{ 'width': liveRoom.width + 'vh', 'height': liveRoom.height + 'vh', 'left': liveRoom.left + 'vh', 'bottom': liveRoom.bottom + 'vh' }"
              />
              <!-- <img :src="liveRoom.qrPos" ref="avatar"
                :style="{ 'width': liveRoom.width + 'vh', 'height': liveRoom.height + 'vh', 'left': liveRoom.left + 'vh', 'bottom': liveRoom.bottom + 'vh' }"> -->
              <p
                :style="{ 'text-align': 'center', 'width': liveRoom.width + 'vh', 'color': 'black', 'margin-top': '0', 'font-size': '0.8vh','word-wrap':'break-word' }">
                {{ liveRoom.name }}</p>
            </div>
          </div>
        </div>
      </div>
    </el-col>
    <el-col :span="8" class="poster-control" v-loading="isDownloading" element-loading-text="生成海报中">
      <el-row>
        <h1>技术播客月海报生成器</h1>
      </el-row>
      <el-form label-position="top">
        <el-form-item label="海报背景地址" id="bgUrl">
          <el-input placeholder="请输入背景地址" v-model="bgUrl">
            <template slot="append">可填入URL地址-HTTPS</template>
          </el-input>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="播客" id="podcast">
          <el-select v-model="podcastList" multiple placeholder="请选择" :multiple-limit="3" class="podcast-select">
            <el-option v-for="item in podcastSelectorList" :key="item.id" :label="item.name" :value="item.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="播客Logo调整" v-if="podcastLogoImageList.length > 0">
          <el-collapse v-model="activePodcastLogos" class="podcast-logo-edit-panel">
            <el-collapse-item v-for="podcastLogoImage in podcastLogoImageList" :key="podcastLogoImage.id"
              :title="podcastLogoImage.name" :name="podcastLogoImage.id">
              <el-input placeholder="请输入宽度" v-model="podcastLogoImage.width" class="logo-attribute">
                <template slot="prepend">Width:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="podcastLogoImage.height" class="logo-attribute">
                <template slot="prepend">Height:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="podcastLogoImage.top" class="logo-attribute">
                <template slot="prepend">Top:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="podcastLogoImage.left" class="logo-attribute">
                <template slot="prepend">Left:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="podcastLogoImage.logoPos" class="logo-attribute">
                <template slot="prepend">Image:</template>
                <template slot="append">可填入URL地址-HTTPS</template>
              </el-input>
              <el-button icon="el-icon-refresh-left" @click="zoomReset(podcastLogoImage.id)" circle></el-button>
            </el-collapse-item>
          </el-collapse>
        </el-form-item>
      </el-form>
      <el-divider></el-divider>
      <el-form>
        <el-form-item label="串台主题">
          <el-input v-model="topic" />
        </el-form-item>
        <el-form-item label="字号调整（串台主题）">
          <div :span="12">
            <a href="javascript:;" @click="topicFontAdd()">
              <i class="el-icon-zoom-in avatar-icon"></i>
            </a>
            <a href="javascript:;" @click="topicFontMinus()">
              <i class="el-icon-zoom-out avatar-icon"></i>
            </a>
            <a href="javascript:;" @click="topicFontReset()">
              <i class="el-icon-refresh-left avatar-icon"></i>
            </a>
          </div>
        </el-form-item>
        <el-form-item label="主题描述">
          <el-button icon="el-icon-plus" @click="addTopicDesc()" circle></el-button>
        </el-form-item>
        <el-form-item v-for="(topicDesc, index) in topicDescList" :key="index" :label="'第' + (index + 1) + '行'">
          <div class="topic-desc-item">
            <el-input v-model="topicDesc.value" />
            <el-button icon="el-icon-minus" @click="minusTopicDesc(index)" type="danger" circle></el-button>
          </div>
        </el-form-item>
        <el-form-item label="字号调整（主题描述）">
          <div :span="12">
            <a href="javascript:;" @click="topicDescFontAdd()">
              <i class="el-icon-zoom-in avatar-icon"></i>
            </a>
            <a href="javascript:;" @click="topicDescFontMinus()">
              <i class="el-icon-zoom-out avatar-icon"></i>
            </a>
            <a href="javascript:;" @click="topicDescFontReset()">
              <i class="el-icon-refresh-left avatar-icon"></i>
            </a>
          </div>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="直播时间">
          <el-date-picker v-model="pickerTime" type="datetimerange" start-placeholder="开始日期" end-placeholder="结束日期"
            :default-time="['20:00:00', '22:00:00']">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="直播时间展示">
          <el-input v-model="time" />
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item label="直播预约">
          <el-button icon="el-icon-plus" @click="addLiveRoomQR()" circle></el-button>
          <el-input placeholder="直播日期" v-model="liveRoomkey" class="logo-attribute">
            <template slot="prepend">7月:</template>
            <template slot="append">日</template>
          </el-input>
        </el-form-item>
        <el-form-item label="">
          <el-collapse v-model="activeLiveRoomQRList" class="podcast-logo-edit-panel">
            <el-collapse-item v-for="(liveRoom, index) in entireLiveRoomQRList" :title="liveRoom.name" :name="index">
            <el-input placeholder="请输入名称" v-model="liveRoom.name" class="logo-attribute">
                <template slot="prepend">Name:</template>
              </el-input>
              <el-input placeholder="请输入宽度" v-model="liveRoom.width" class="logo-attribute">
                <template slot="prepend">Width:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="liveRoom.height" class="logo-attribute">
                <template slot="prepend">Height:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="liveRoom.bottom" class="logo-attribute">
                <template slot="prepend">Bottom:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="liveRoom.left" class="logo-attribute">
                <template slot="prepend">Left:</template>
                <template slot="append">vh</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="liveRoom.text" class="logo-attribute">
                <template slot="prepend">内容:</template>
              </el-input>
              <el-input placeholder="请输入内容" v-model="liveRoom.logoSrc" class="logo-attribute">
                <template slot="prepend">Logo:</template>
                <template slot="append">可填入URL地址-HTTPS</template>
              </el-input>
              <el-button icon="el-icon-minus" @click="delLiveRoomQR(index)" type="danger" circle></el-button>
            </el-collapse-item>
          </el-collapse>
        </el-form-item>
        <el-divider></el-divider>
        <el-form-item>
          <el-button type="primary" @click="download()">
            生成海报
          </el-button>
        </el-form-item>

        <el-form-item class="info">
          <i class="el-icon-service"></i> 本工具由 <a href="https://github.com/le0zh0u">@leozhou</a> 开发，<a
            href="mailto:hi@leozhou.me">问题反馈</a>
        </el-form-item>
      </el-form>
    </el-col>
  </el-row>
</template>

<script lang="ts">
import Vue from 'vue';
import moment from 'moment';
import VueQr from 'vue-qr';

// @ts-ignore
import domtoimage from 'retina-dom-to-image';
import podcastDataRaw from './data/podcast-data';
import liveRoomDataRaw from './data/liveroom-data';


// 播客信息 - 用于数据导出时适配试用
type PodcastInfo = {
  id: number,
  name: string,
  logoName: string
};

// 直播间数据
type PlatformLiveRoomInfo = {
    platform: string,
    a: string,
    b: string,
    c: string,
    d: string,
    e: string,
    f: string,
    g: string,
    h: string,
    i: string,
    j: string
};

// 播客logo默认配置，用于调整后恢复使用
type PodcastLogoDefaultPos = {
  width: number,
  height: number,
  top: number,
  left: number
};

// 播客logo信息，用于展示
type PodcastLogoInfo = {
  id: number,
  name: string,
  logoPos: string,
  width: number,
  height: number,
  top: number,
  left: number,
  defaultPos: PodcastLogoDefaultPos
};

// 直播间信息
type LiveRoomInfo = {
  account: string,
  image: ImageData
}

type LiveRoomQRDefaultPos = {
  width: number,
  height: number,
  bottom: number,
  left: number,
}

// 直播间二维码信息，用于展示
type LiveRoomQRInfo = {
  name: string,
  text: string,
  logoSrc: string,
  qrPos: string,
  width: number,
  height: number,
  bottom: number,
  left: number,
  defaultPos: LiveRoomQRDefaultPos
};

// 从数据源中导入播客数据
function getPodcastData(raw: String): PodcastInfo[] {
  return raw.split('\n').slice(2)
    .filter(line => !!line)
    .map((line) => {
      const arr = line.split(',')
      return {
        id: Number(arr[0]),
        name: arr[1],
        logoName: arr[2],
      }
    })
}

function getLiveRoomData(raw:String): PlatformLiveRoomInfo[] {
  return raw.split('\n').slice(2)
    .filter(line => !!line)
    .map((line) => {
      const arr = line.split(',')
      return {
        platform: arr[0],
        'a': arr[1],
        'b': arr[2],
        'c': arr[3],
        'd': arr[4],
        'e': arr[5],
        'f': arr[6],
        'g': arr[7],
        'h': arr[8],
        'i': arr[9],
        'j': arr[10]
      }
    })
}

// 构建播客数据
const podcastInfoList = getPodcastData(podcastDataRaw)
const liveRoomInfoList = getLiveRoomData(liveRoomDataRaw)

export default Vue.extend({
  data() {
    return {
      // bgURL
      bgUrl: 'poster-template.jpg', //'http://img.leozhou.me/picGo/技术播客月直播预告.png',
      // 串台主题
      topic: '',
      // 主题描述
      topicDescList: [{ value: '' }],
      // 直播时间
      time: '',
      pickerTime: [],
      // 主题文字缩放
      topicFontSize: 1,
      topicDescFontSize: 1,

      // 是否导出中
      isDownloading: false,
      posterBase64: '',

      // new attribute
      // 选中的播客
      podcastList: [],
      // 选中的播客图片信息
      podcastLogoImageList: [] as PodcastLogoInfo[],
      // 展开的播客logo编辑
      activePodcastLogos: [],

      // 全程直播间信息
      activeLiveRoomQRList: [],
      entireLiveRoomQRList: [] as LiveRoomQRInfo[],
      liveRoomCount: 0,

      // 单场
      singleLiveRoomQRList: [] as LiveRoomQRInfo[],
      liveRoomQRPannelHeight: 10,

      liveRoomkey: "4"

    };
  },
  created() {
    this.generateEntireLiveRoomData()
  },

  watch: {
    // 如果选中的播客发生变化，需要重新构建播客logo的配置
    podcastList(newList, oldList) {
      if (newList == oldList) return;
      const length = newList.length
      const list: PodcastLogoInfo[] = newList.sort((a: number, b: number) => {
        return a - b
      }).map((item: number, index: number) => {
        const podcast = podcastInfoList.filter(podcastInfo => {
          return podcastInfo.id === item
        })
        return this.genLogoPos(podcast[0], index, length)
      })

      this.podcastLogoImageList = list
    },
    // entireLiveRoomQRList(newList, oldList) {
    //   if (newList == oldList) return;
    //   if (newList.length == oldList.length) return;
    //   this.liveRoomCount = newList.length;
    //   console.log('entireLiveRoomQRList changed', this.liveRoomCount)
    // },
    // liveRoomCount(newVal, oldVal) {

    //   this.recalLiveRoomPos(newVal)
    // },
    // 如果时间选择发生变化，格式化时间
    pickerTime(newTimeArr, oldTimeArr) {
      if (newTimeArr == oldTimeArr) return

      const startTime = moment(newTimeArr[0]).format('YYYY/MM/DD HH:mm')
      var endTime = ''
      if (moment(newTimeArr[0]).diff(moment(newTimeArr[1]), 'days') == 0) {
        // 同一天
        endTime = moment(newTimeArr[1]).format('HH:mm')
      } else {
        endTime = moment(newTimeArr[1]).format('MM/DD HH:mm')
      }

      this.time = startTime + " - " + endTime

    },
    liveRoomkey(newKey, oldKey) {
      this.generateEntireLiveRoomData()
    }
  },
  mounted() {
  },

  computed: {
    // 用于下拉框
    podcastSelectorList() {
      return podcastInfoList
    },
  },
  components: {
      VueQr 
  },
  methods: {

    recalLiveRoomPos() {

      const listLen = this.entireLiveRoomQRList.length

      const bottom = 21
      var sideLen = 8
      var left = 4
      var splitSpace = 1
      const maxWidth = 71

      var basicLen = (left - splitSpace) + (sideLen + splitSpace) * listLen + left
      // console.log('basicLen - ', basicLen)
      if (basicLen > maxWidth) {
        // 比预计长，缩小边长
        const contentWidth = maxWidth - left - left + splitSpace
        sideLen = contentWidth / listLen - splitSpace
      } else {
        // const contentWidth = (sideLen + splitSpace) * listLen - splitSpace
        let space = (maxWidth - (2 * left) - (listLen * sideLen)) / (listLen - 1)
        if (space > 4) {
          splitSpace = (maxWidth - (listLen * sideLen)) / (listLen + 1)
          left = splitSpace
        } else {
          splitSpace = space
        }
        // console.log('splitSpace - ', splitSpace)
      }

      var list = this.entireLiveRoomQRList.map((item: LiveRoomQRInfo, index: number) => {
        item.left = left + (sideLen + splitSpace) * index
        item.width = sideLen
        item.height = sideLen
        return item
      })

      this.entireLiveRoomQRList = list
      this.liveRoomQRPannelHeight = sideLen + 3

    },

    generateEntireLiveRoomData() {
      this.entireLiveRoomQRList = [] as LiveRoomQRInfo[]
      const bottom = 21
      const width = 8
      const height = 8
      const letf = 4

      const splitSpace = 1

      // 思否
      
      const segmentfault = {
        name: "思否",
        logoSrc: "logos/segementfault.png",
        text: this.getLiveRoomMap('sifou'),
        qrPos: "empty-image.png",
        width: width,
        height: height,
        bottom: bottom,
        left: letf + (splitSpace + width) * 3,
      }
      this.entireLiveRoomQRList.push(segmentfault as LiveRoomQRInfo)

      // CSDN
      const csdn = {
        name: "CSDN",
        logoSrc: "logos/csdn.png",
        text: this.getLiveRoomMap('csdn'),
        qrPos: "empty-image.png",
        width: width,
        height: height,
        bottom: bottom,
        left: letf,
      }
      this.entireLiveRoomQRList.push(csdn as LiveRoomQRInfo)
      
      // huodongxing
      const huodongxing = {
        name: "活动行",
        logoSrc: "logos/huodongxing.png",
        text: this.getLiveRoomMap('huodongxing'),
        qrPos: "empty-image.png",
        width: width,
        height: height,
        bottom: bottom,
        left: letf + (splitSpace + width),
      }
      this.entireLiveRoomQRList.push(huodongxing as LiveRoomQRInfo)

      this.recalLiveRoomPos()
    },

    //生成logo的位置
    genLogoPos(podcastInfo: PodcastInfo, index: number, podcastNum: number) {
      const logDefaultPos: PodcastLogoDefaultPos = {
        width: 16,//417 * 100 / 2208,
        height: 16,//417 * 100 / 2208,
        top: 170 / 718 * 100,//373 / 2208 * 100,
        left: 486 / 864 / 2 * 100,//100 / 2208 * 1242,
      }

      if (podcastNum === 1) {
        // none
      } else if (podcastNum === 2) {
        logDefaultPos.left = 70 / 508 * 100 + (16 + 11) * index
      } else if (podcastNum === 3) {
        if (index == 0) {
          logDefaultPos.left = 8
        } else if (index == 1) {
          // logDefaultPos.left =42
        } else {
          logDefaultPos.left = 486 / 864 * 100 - 8
        }
      }

      return {
        id: podcastInfo.id,
        name: podcastInfo.name,
        logoPos: 'logos/' + podcastInfo.logoName,
        width: logDefaultPos.width,//417 * 100 / 2208,
        height: logDefaultPos.height,//417 * 100 / 2208,
        top: logDefaultPos.top,//373 / 2208 * 100,
        left: logDefaultPos.left,//100 / 2208 * 1242,
        logoZoom: 1,
        defaultPos: logDefaultPos
      }
    },

    // 生成图片并下载
    download() {
      this.isDownloading = true;
      domtoimage.toJpeg(document.getElementById('poster-preview'))
        .then((url: string) => {
          this.posterBase64 = url;
          this.isDownloading = false;
          const link = document.createElement('a');
          link.href = url;
          link.download = this.topic + '.jpeg';
          link.click();
        })
    },

    // 重置图片大小
    zoomReset(id: number) {
      const filterResult = this.podcastLogoImageList.filter(item => {
        return item.id === id
      })
      if (filterResult.length == 0) {
        return
      }

      const logo = filterResult[0]
      logo.width = logo.defaultPos.width;
      logo.height = logo.defaultPos.height;
      logo.left = logo.defaultPos.left;
      logo.top = logo.defaultPos.top;
    },

    // 放大主题文字
    topicFontAdd() {
      this.topicFontSize += 0.1;
    },

    // 缩小主题文字
    topicFontMinus() {
      this.topicFontSize -= 0.1;
    },
    // 重置主题文字大小
    topicFontReset() {
      this.topicFontSize = 1;
    },

    // 放大主题描述文字
    topicDescFontAdd() {
      this.topicDescFontSize += 0.1;
    },

    // 缩小主题描述文字
    topicDescFontMinus() {
      this.topicDescFontSize -= 0.1;
    },
    // 重置主题描述文字大小
    topicDescFontReset() {
      this.topicDescFontSize = 1;
    },

    // 添加主题描述
    addTopicDesc() {
      this.topicDescList.push({ value: '' })
    },
    minusTopicDesc(index: number) {
      this.topicDescList.splice(index, 1)
    },

    // 添加直播二维码
    addLiveRoomQR() {
      const liveRoom = {
        name: "直播间",
        qrPos: "empty-image.png",
        logoSrc: "",
        text:"https://github.com/opensource-f2f/episode/issues/96",
        width: 8,
        height: 9,
        bottom: 21,
        left: 0,
      }

      this.entireLiveRoomQRList.push(liveRoom as LiveRoomQRInfo)

      this.recalLiveRoomPos()
    },

    delLiveRoomQR(index: number) {
      this.entireLiveRoomQRList.splice(index, 1)
      this.recalLiveRoomPos()
    },

    getLiveRoomMap(platform: string) : string {
      //Map<String,PlatformLiveRoomInfo>
      let map = new Map(liveRoomInfoList.map(i => [i.platform, i]))
      let liveRoom = map.get(platform) as PlatformLiveRoomInfo
      if(liveRoom) {
        if(this.liveRoomkey == '4'){
          return liveRoom.a
        }
        else if(this.liveRoomkey == '5'){
          return liveRoom.b
        }else if(this.liveRoomkey == '6'){
          return liveRoom.c
        }else if(this.liveRoomkey == '7'){
          return liveRoom.d
        }else if(this.liveRoomkey == '8'){
          return liveRoom.e
        }else if(this.liveRoomkey == '11'){
          return liveRoom.f
        }else if(this.liveRoomkey == '12'){
          return liveRoom.g
        }else if(this.liveRoomkey == '13'){
          return liveRoom.h
        }else if(this.liveRoomkey == '14'){
          return liveRoom.i
        }else if(this.liveRoomkey == '15'){
          return liveRoom.j
        }
        
      }

      return ""
    }
  },
})
</script>

<style>
@font-face {
  font-family: 'Open Sans';
  font-style: normal;
  font-weight: bold;
  src: url(~assets/OpenSans-Bold.ttf) format('truetype');
}

@font-face {
  font-family: 'Open Sans';
  font-style: normal;
  font-weight: normal;
  src: url(~assets/OpenSans-Light.ttf) format('truetype');
}

@font-face {
  font-family: 'SourceHanSerifSC';
  font-style: normal;
  font-weight: 300;
  src: url(~assets/SourceHanSerifSC-Heavy.otf) format('opentype');
}

h1 {
  margin-bottom: 15px;
  font-size: 20px;
  font-family: 'SourceHanSerifSC', 'Open Sans';
}

.poster-container {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  font-family: 'Open Sans';
}

#poster-preview {
  position: relative;
  width: max-content;
  height: 100vh;
  overflow: hidden;
  -webkit-user-select: none;
  user-select: none;
}

.bg {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  background: white;
  z-index: -20;
}

.poster-template {
  height: 100vh;
}

.poster-content {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  padding: 46vh 0vh 0 0vh;
  text-align: center;
  color: #fff;
  font-size: 2vh;
}

.podcast-logo-img {
  position: absolute;
  z-index: 10;
}

.topic {
  margin-bottom: 0.5vh;
  font-size: 3.5vh;
  font-weight: bold;
  white-space: pre-line;
  word-break: break-all;
  word-wrap: break-word;
}

.sub-topic {
  margin-bottom: 0.5vh;
  font-size: 2.5vh;
  /* font-weight: bold; */
  white-space: pre-line;
  word-break: break-all;
  word-wrap: break-word;
}

.time {
  font-size: 2vh;
  /* color: #fff; */
  color: #ccc;
}

.el-form-item {
  margin-bottom: 5px;
}

#podcast {
  margin-top: 10px;
}

.avatar-icon {
  font-size: 22px;
  color: #ccc;
  width: 40px;
  height: 40px;
  line-height: 40px;
  text-align: center;
}

.icon-panel {
  margin-top: 12px;
}

.el-upload:hover .avatar-icon,
a:hover .avatar-icon {
  color: #409EFF;
}

.avatar {
  width: 40px;
  height: 40px;
  display: block;
}

.info,
.info a {
  color: #aaa;
}

.logo-attribute {
  margin-bottom: 10px;
}

.topic-desc-item .el-input {
  width: 50%;
}

.el-button.is-circle {
  padding: 6px;
}

.podcast-logo-edit-panel {
  margin-top: 10px
}

.podcast-select {
  width: 100%
}
</style>
