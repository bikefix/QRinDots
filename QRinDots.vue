<template>
  <svg xmlns="http://www.w3.org/2000/svg"  xmlns:xlink="http://www.w3.org/1999/xlink"  :width="width" :height="height" :viewBox="'0 0 '+(codeWidth)+' '+(code.length)">
    <g v-for="(r, ir) in code" >
      <ellipse
         v-for="(c, ic) in r"
         :cx="xf(ir)+.475"
         :cy="ic+.475"
         :rx="xw(ir)"
         :ry=".55"
         :fill="c?'#000':'#fff'"
      />
    </g>
  </svg>
</template>
<script>
  import qr from 'qr.js'
  export default {
    props: {
      value: {
        type: String,
        default: "https://bikefix.co.uk/0000000000012345",
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
    computed: {
      code(){
        const qrcode = qr(this.value, { errorCorrectLevel: this.level })
        const l = Array(qrcode.modules.length+2)
        return [[...l],...qrcode.modules.map(r => [false,...r,false]),[...l]]
      },
      codeWidth(){
        let w = this.code.length
        return w + (((w/2) * (w/2)) / w / 5 * this.bend)
      }
    },
    methods:{
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
