<template>
  <svg xmlns="http://www.w3.org/2000/svg"  xmlns:xlink="http://www.w3.org/1999/xlink"  :width="width" :height="height" :viewBox="'0 0 '+(codeWidth)+' '+(code.length)">
    <g v-if="paths[0] && paths[0][0]">
      <g v-for="(path) in paths">
        <g v-for="(link) in path" :transform="'translate('+ (link.x||0) +' '+ (link.y||0) + ')'" >
          <path v-if="link.d === 'w'" transform="translate(0 -1)" d="M .05 .5 C 
            .25 1, .25 1, .05 1.5,
            .25 1.5, .25 1.5, .95 1.5,
            .75 .95, .75 .95, .95 .50"
            :fill="link.c"
          />
          <path v-if="link.d === 's'" d="M .05 .5 C 
            .25 1, .25 1, .05 1.5,
            .25 1.5, .25 1.5, .95 1.5,
            .75 .95, .75 .95, .95 .50"
            :fill="link.c"
          />
          <g v-if="link.d === 'a'" transform="translate(-1 0)">
            <path  d="M .50 .05 C 
              1 .25, 1 .25, 1.5 .05,
              1.5 .25, 1.5 .25, 1.5 .95,
              .95 .75, .95 .75, .5 .95"
              :fill="link.c"
            />
          </g>
          <path v-if="link.d === 'd'" d="M .50 .05 C 
            1 .25, 1 .25, 1.5 .05,
            1.5 .25, 1.5 .25, 1.5 .95,
            .95 .75, .95 .75, .5 .95"
            :fill="link.c"
          />
        </g>
      </g>
    </g>
    <g v-for="(y, iy) in code" >
      <circle
        v-for="(dot, ix) in y" 
        :cx="iy+.5" 
        :cy="ix+.5" 
        :fill="dot?'#000':'#fff'"
        r=".45"
      />
      <circle 
        v-for="(dot, ix) in y" 
        :cx="iy+.5" :cy="ix+.5" 
        :fill="dot?'#444':'#ddd'" 
        r=".165"
      />
    </g>
  </svg>
</template>
<script>
  // import { QRCode, ErrorCorrectLevel, QRNumber, QRAlphaNum, QR8BitByte, QRKanji } from 'qrcode-generator-ts'
  import { qrcode2 } from 'qrcode-generator'
  function randomElement(array) {
    const index = Math.floor(Math.random() * array.length)
    return array[index]
  }
  export default {
    props: {
      value: {
        type: String,
        default: "https://bikefix.co.uk/j/UiuQG4dDkjeICYIDSJo2",
      },
      level: {
        type: Number,
        default: 0,
      },
      width: {
        type: Number,
        default: 200,
      },
      heigth: {
        type: Number,
        default: 200,
      },
      bend: {
        type: Number,
        default: 2,
      },
    },
    mounted(){
      this.makeCode()
      this.newPickFrom(this.code)
      this.pathsInterval = setInterval(() => {
        this.makePath(this.code, 50)
      }, 0)
    },
    data(){
      return {
        pathsInterval: 0,
        pickFrom: [],
        paths: [],
        dots: [],
      }
    },
    computed: {
      codeWidth(){
        let w = this.code.length
        return w + (((w / 2) * (w / 2)) / w / 5 * this.bend)
      },
    },
    methods:{
      makeCode(){
        const qr = qrcode2(0, 'L')
        qr.addData(this.value)
        qr.make()
        const length = qr.getModuleCount()
        const l = Array(length+2)
        for (let i = 0; i < l.length; ++i) {
          l[i] = false
        }
        let theDots = []
        theDots.push([...l])
        for (let y = 0; y < length; y++) {
          let theLine = []
          for (let x = 0; x < length; x++) {
            theLine.push(qr.isDark(x, y))
          }
          theDots.push([false, ...theLine, false])
        }
        theDots.push(l)
        this.code = theDots
      },
      newPickFrom(code){
        code.forEach((row, x) => {
          row.forEach((dotDone, y) => {
            this.pickFrom.push({x, y})
          })
        })
      },
      pickable(x, y){
        const index = this.pickFrom.findIndex(dot => {
          return dot.x === x && dot.y === y
        })
        if (index !== -1) {
          return true
        }
        return false
      },
      picked(x, y){
        const index = this.pickFrom.findIndex(dot => {
          return dot.x === x && dot.y === y
        })
        if (index !== -1) {
          this.pickFrom.splice(index, 1)
          return true
        }
        return false
      },
      randomPick(){
        const index = Math.floor(Math.random() * this.pickFrom.length)
        const picked = this.pickFrom.splice(index, 1)[0]
        return picked
      },
      makePath(code, minLen) {
        // console.log(this.pickFrom.length)
        
        if (this.pickFrom.length === 0) {
          clearInterval(this.pathsInterval)
          this.newPickFrom(this.code)
          this.paths.forEach(path => {
            // console.log(path)
            path.forEach(link => {
              this.picked(link.x, link.y)
            })
          })
          if (minLen > 0) {
            this.pathsInterval = setInterval(() => {
              this.makePath(this.code,minLen * 0.8)
            }, 0)
          }
          
          return 
        }
        const picked = this.randomPick()
        if (!picked) {
          console.log(picked, "not picked")
          return
          // clearInterval(this.pathsInterval)
        }
        
        const x = picked.x
        const y = picked.y
        const move = this.move(x, y, "", [], code, false)
        
        if (move && move.length > minLen) {
          
          console.log(move)
          console.log(move[0].x)
          console.log(move[move.length-1].x)
          console.log(move[0].y)
          console.log(move[move.length-1].y)
          
          this.paths.push(move)
        }
        
        // console.log(this.pickFrom.length)
      },
      move(x, y, dir = "", links = [], dots, reversed = false){
        // console.log({x, y, dir, links, dots, reversed})
        if (dir) {
          if (dir === "a") x--
          if (dir === "d") x++
          if (dir === "w") y--
          if (dir === "s") y++
          const previouslyPicked = !this.picked(x, y)
          if (previouslyPicked) {
            console.log("previonsly picked")
            return links
          }
        }

        let Directions = []
        const code = dots[x][y]
        if (dir !== "d" && dots[x-1] !== undefined && this.code[x-1][y] === code) {
          if (this.pickable(x-1,y)) Directions.push("a")
        }
        if (dir !== "a" && dots[x+1] !== undefined && this.code[x+1][y] === code){
          if (this.pickable(x+1,y)) Directions.push("d")
        }
        if (dir !== "s" && dots[x][y-1] === code){
          if (this.pickable(x,y-1)) Directions.push("w")
        }
        if (dir !== "w" && dots[x][y+1] === code){
          if (this.pickable(x,y+1)) Directions.push("s")
        }
        
        if (!Directions.length) {
          if (links.length > 1) {
            if (!reversed) {
              const head = links[0]
              links = links.reverse()
              const newHead = links[0]
              
              // console.log({newHead, head})
              
              const hx = newHead.x
              const hy = newHead.y
              
              this.picked(hx, hy)
              
              // console.log("reversed", {hx, hy, links, dots})
              return this.move(x, y, "", links, dots, true)
            }
            
            return links
          }
          return false
        }
        const d = randomElement(Directions)
        
        const c = (links.length % 2) ? (code ? '#000':'#fff') : (code ? '#444':'#ddd')
          
        return this.move(x,y,d,[{
          x, y, c, d,
        }, ...links], dots)
      },
      xf(i){
        let w = this.code.length
        let v = 0
        if (i < w / 2) {
          let fromMid = (w / 2) - i
          v = i - (fromMid * fromMid / w / 10 * this.bend)
        } else {
          let fromMid = i - (w / 2)
          v = i + (fromMid * fromMid / w / 10 * this.bend)
        }
        return v + this.bend
      },
      xw(i){
        let w = this.code.length
        let v = 0
        if (i < w / 2) {
          let fromMid = (w / 2) - i
          v = ((fromMid * fromMid / w / 5) * (this.bend / 10)) + .55
        } else {
          let fromMid = i - (w / 2)
          v = ((fromMid * fromMid / w / 5) * (this.bend / 10)) + .55
        }
        return v
      }
    }
  }
</script>
