<template>
  <el-row type="flex" class="poster-container" justify="center">
    <el-col :span="12">
      <div id="poster-preview"
        :style="{'cursor': 'default'}"
      >
        <div class="bg"></div>
        <img class="poster-template" :src="'poster-template.png'">
        <div v-for="podcastLogoImage in podcastLogoImageList" v-if="podcastLogoImageList.length > 0" class="podcast-logo-img" :style="{'left': podcastLogoImage.left + 'vh', 'top': podcastLogoImage.top + 'vh'}">
          <img  :src="podcastLogoImage.logoPos" ref="avatar"
            :style="{'width': podcastLogoImage.width + 'vh', 'height': podcastLogoImage.height + 'vh', 'left': podcastLogoImage.left + 'vh', 'top': podcastLogoImage.top + 'vh'}"
          >
          <p :style="{'text-align': 'center','width': podcastLogoImage.width + 'vh','color':'white', 'margin-top':'0.1em', 'font-size':'1.7vh'}">{{podcastLogoImage.name}}</p>
        </div>
        <div class="poster-content">
          <div class="topic" :style="{'font-size': 3.5 * topicFontSize + 'vh'}">{{ topic }}</div>
          <div class="time">{{ time }}</div>
        </div>
      </div>
    </el-col>
    <el-col :span="8" class="poster-control" v-loading="isDownloading" element-loading-text="生成海报中">
      <el-row>
        <h1>技术播客月海报生成器</h1>
        <p>串台播客数量</p><el-radio-group size="small" v-model="podcastNum">
          <el-radio-button label="1"></el-radio-button>
          <el-radio-button label="2"></el-radio-button>
          <el-radio-button label="3"></el-radio-button>
        </el-radio-group>
      </el-row>
      <el-form>
        <el-form-item label="播客" id="podcast">
          <el-select v-model="podcastList" multiple placeholder="请选择" :multiple-limit="podcastNums">
            <el-option
              v-for="item in podcastSelectorList"
              :key="item.id"
              :label="item.name"
              :value="item.id">
            </el-option>
          </el-select>
          <el-collapse v-model="activePodcastLogos" v-if="podcastLogoImageList.length > 0">
            <el-collapse-item v-for="podcastLogoImage in podcastLogoImageList" :key="podcastLogoImage.id" :title="podcastLogoImage.name" :name="podcastLogoImage.id">
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
              <el-button icon="el-icon-refresh-left" @click="zoomReset(podcastLogoImage.id)" circle></el-button>
            </el-collapse-item>
          </el-collapse>
        </el-form-item>
      </el-form>
      <el-form>
        <el-form-item label="串台主题">
          <el-input v-model="topic" />
        </el-form-item>
        <el-form-item label="直播时间">
          <el-date-picker
            v-model="pickerTime"
            type="datetimerange"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :default-time="['20:00:00', '22:00:00']">
          </el-date-picker>
          <el-input v-model="time" />
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

        <el-form-item>
          <el-button type="primary" @click="download()"
          >
            生成海报
          </el-button>
        </el-form-item>

        <el-form-item class="info">
          <i class="el-icon-service"></i> 本工具由 <a href="https://github.com/le0zh0u">@leozhou</a> 开发，<a href="mailto:hi@leozhou.me">问题反馈</a>
        </el-form-item>
      </el-form>
    </el-col>
  </el-row>
</template>

<script lang="ts">
import Vue from 'vue';
import moment from 'moment';

// @ts-ignore
import domtoimage from 'retina-dom-to-image';
import podcastDataRaw from './data/podcast-data'


// 播客信息 - 用于数据导出时适配试用
type PodcastInfo = {
  id: number,
  name: string,
  logoName: string
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

// 从数据源中导入播客数据
function getPodcastData(raw:String): PodcastInfo[] {
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

// 构建播客数据
const podcastInfoList = getPodcastData(podcastDataRaw)

export default Vue.extend({
  data() {
    return {
      // 串台主题
      topic: '', 
      // 直播时间
      time: '',
      pickerTime:[],
      // 主题文字缩放
      topicFontSize: 1,

      // 是否导出中
      isDownloading: false,
      posterBase64: '',

      // new attribute
      // 播客数量
      podcastNum: "2",
      // 选中的播客
      podcastList: [],
      // 选中的播客图片信息
      podcastLogoImageList: [] as PodcastLogoInfo[],
      // 展开的播客logo编辑
      activePodcastLogos: []
    };
  },

  watch: {
    // 如果选中的播客发生变化，需要重新构建播客logo的配置
    podcastList(newList, oldList) {
      if(newList == oldList) return ;
      console.log(newList)
      const list:PodcastLogoInfo[] = newList.sort((a:number, b:number) => {
        return a - b
      }).map((item:number, index:number) => {
        const podcast = podcastInfoList.filter(podcastInfo => {
          return podcastInfo.id === item
        })
        return this.genLogoPos(podcast[0], index)
      })

      this.podcastLogoImageList = list
    },
    // 如果时间选择发生变化，格式化时间
    pickerTime(newTimeArr, oldTimeArr){
      if(newTimeArr == oldTimeArr) return 

      const startTime = moment(newTimeArr[0]).format('YYYY/MM/DD HH:mm')
      var endTime = ''
      if(moment(newTimeArr[0]).dayOfYear === moment(newTimeArr[1]).dayOfYear){
        // 同一天
         endTime = moment(newTimeArr[1]).format('HH:mm')
      } else {
        endTime = moment(newTimeArr[1]).format('MM/DD HH:mm')
      }
      
      this.time = startTime + " - " + endTime
      
    }
  },
  mounted() {
  },

  computed: {
    // 用于下拉框
    podcastSelectorList(){
      return podcastInfoList
    },
    // 用于多选的数量限制
    podcastNums(){
      let s:string = this.podcastNum
      return Number(s)
    }
  },

  methods: {

    //生成logo的位置
    genLogoPos(podcastInfo: PodcastInfo, index:number){
      const logDefaultPos: PodcastLogoDefaultPos = {
        width: 16,//417 * 100 / 2208,
        height: 16,//417 * 100 / 2208,
        top: 170 / 718 * 100,//373 / 2208 * 100,
        left: 486/864/2 * 100,//100 / 2208 * 1242,
      }

      if(this.podcastNums === 1){
        // none
      } else if(this.podcastNums === 2){
        logDefaultPos.left = 70 / 508 * 100 + (16 + 11) * index
      } else if(this.podcastNums === 3) {
        if(index == 0){
          logDefaultPos.left = 8
        } else if(index == 1){
          // logDefaultPos.left =42
        } else {
          logDefaultPos.left = 486 / 864 * 100 - 8
        }
      }

      return {
          id: podcastInfo.id,
          name: podcastInfo.name,
          logoPos: 'logos/'+podcastInfo.logoName,
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
    zoomReset(id:number) {
      console.log(this.podcastLogoImageList)
      const filterResult = this.podcastLogoImageList.filter(item => {
        return item.id === id
      })
      if(filterResult.length == 0){
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
  }
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
      padding: 46vh 3vh 0 3vh;
      text-align: center;
      color: #fff;
      font-size: 2vh;
    }
      .podcast-logo-img{
        position: absolute;
        z-index: 10;
      }

      .topic {
        margin-bottom: 0.5vh;
        font-size: 2.3vh;
        font-weight: bold;
        white-space: pre-line;
        word-break: break-all;
        word-wrap: break-word;

      }

      .time {
        font-size: 2vh;
        color: #fff;
        /* color: #ccc; */
      }

.el-form-item {
  margin-bottom: 5px;
}

.el-form-item__label{
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

.el-upload:hover .avatar-icon, a:hover .avatar-icon {
  color: #409EFF;
}

.avatar {
  width: 40px;
  height: 40px;
  display: block;
}

.info, .info a {
  color: #aaa;
}

.logo-attribute{
  margin-bottom: 10px;
}
</style>
