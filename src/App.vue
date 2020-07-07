<template>
  <div id="app">
    <div class="help" v-if="music == 0">
      <p>使用方法：将musicxml文件拖入本页面</p>
      <p>最低八度：<input v-model="level" type="text"></p>
    </div>
    <ul class="music">
      <li v-for="(bar, index) in music" v-bind:key="index" class="bar">
        <ul class="notes">
          <li v-for="(note, index) in bar" v-bind:key="index" :class="['note', 'duration' + note[0], note[2], note[3]]">
            <p>{{note[1]}}</p>
          </li>
        </ul>
        <ul class="line">
          <li class="line_dashed"></li>
          <li class="line_dashed"></li>
          <li class="line_dashed"></li>
          <li class="line_solid"></li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
import * as X2JS from 'x2js'

export default {
  name: 'App',
  data() {
    return{
      //时值 音高 八度 类型
      music: [],
      level: 5
    }
  },
  methods: {
    getMusic(json) {
      let bar = json['score-partwise']['part']['measure']
      let level = this.level - 1
      console.log(bar)
      bar.forEach((e, index) => {
        let id = index
        // this.music[id] = new Array()
        this.music.push([])
        if(Array.isArray(e.note)){
          e.note.forEach(e => {
            let push = []
            push[0] = e.duration
            //判断空音符
            if(e['rest'] == ''){
              push[1] = ''
              push[2] = ''
              push[3] = 'null'
            }else{
              //判断音高
              let note = e['pitch']
              push[1] = 'CDEFGAB'.indexOf(note['step']) + 1
              if(note['alter'] == 1){
                push[1] += '#'
              }else if(note['alter'] == -1){
                push[1] += 'b'
              }
              //判断八度
              if(note['octave'] == level){
                push[2] = 'down'
              }else if(note['octave'] == level + 1){
                push[2] = ''
              }else if(note['octave'] == level + 2){
                push[2] = 'high'
              }
              //判断类型
              if(e['tie']){
                if(e['tie'].length == undefined){
                  if(e['tie']['_type'] == 'start'){
                    push[3] = 'start'
                  }else{
                    push[3] = 'stop'
                  }
                }else{
                  push[3] = 'middle'
                }
              }
            }
            this.music[id].push(push)
          })
        }else{
          this.music[id] = null
        }
      })
    }
  },
  mounted() {
    let body = document.body
    body.addEventListener("dragenter", function (e) {
      e.preventDefault()
      e.stopPropagation()
    }, false)
    body.addEventListener("dragover", function (e) {
      e.preventDefault()
      e.stopPropagation()
    }, false)
    body.addEventListener("dragleave", function (e) {
      e.preventDefault()
      e.stopPropagation()
    }, false)
    body.addEventListener("drop", (e) => {
      e.preventDefault()
      e.stopPropagation()
      let df = e.dataTransfer
      let file = df.items[0].getAsFile()
      if(file.type != 'text/xml'){
        console.log('文件格式有误')
      }else{
        let reader = new FileReader();
        reader.readAsText(file)
        reader.onload = (e) => {
          let x2js = new X2JS();
          let json = x2js.xml2js(e.target.result)
          this.getMusic(json)
        }
      }
    }, false)
  }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
html, body, #app, .help{
  height: 100vh;
}
.help{
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.music{
  position: relative;
  top: 2em;
  display: grid;
  grid-template-columns: 14em 14em 14em 14em;
  width: 56em;
  margin: 0 auto;
  color: #fff;
}
.music li{
  list-style: none;
}
.bar{
  height: 2em;
  display: flex;
  align-items: center;
}
.line{
  display: grid;
  grid-template-columns: 3.5em 3.5em 3.5em 3.5em;
  height: 100%;
}
.line_dashed{
  border-right: 1px dashed #ccc;
}
.line_solid{
  border-right: 1px solid #bbb;
}
.bar:nth-child(4n+1) .line_dashed:first-child{
  border-left: 1px solid #bbb;
}
.notes{
  position: absolute;
  display: flex;
}
.note{
  height: 1.3em;
  padding-left: 0.2em;
  border-radius: 0.2em;
  background: #4ebf55;
  border-right: 1px solid #fff;
}
.note p{
  position: absolute;
}
.down{
  background: #337add;
}
.high{
  background: #f4a233;
}
.start{
  border-right: none;
  border-radius: 0.2em 0 0 0.2em;
}
.stop{
  border-radius: 0 0.2em 0.2em 0;
  color: transparent;
}
.middle{
  border-radius: 0;
  border: none;
  color: transparent;
}
.null{
  opacity: 0;
}
.duration6{
  width: 0.875em;
}
.duration12{
  width: 1.75em;
}
.duration18{
  width: 2.625em;
}
.duration24{
  width: 3.5em;
}
.duration36{
  width: 5.25em;
}
.duration48{
  width: 7em;
}
.duration72{
  width: 10.5em;
}
</style>