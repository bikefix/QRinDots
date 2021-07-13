<template>
  <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"  :width="width" :height="height" :viewBox="'0 0 '+code.length+' '+code.length">
    <g v-for="(r, ir) in code" >
      <circle v-for="(c, ic) in r" :cx="ir+.475" :cy="ic+.475" r=".55" :fill="c?'#000':'#fff'" />
    </g>
  </svg>
</template>
<script>
  import qr from 'qr.js'
  export default {
    props: {
      value: {
        type: String,
        default: "https://bikefix.co.uk/123456789",
      },
      level: {
        type: Number,
        default: 0,
      },
      width: {
        type: Number,
        default: 300,
      },
      heigth: {
        type: Number,
        default: 300,
      },
    },
    computed: {
      code(){
        const qrcode = qr(this.value, { errorCorrectLevel: this.level })
        const l = Array(qrcode.modules.length+2)
        return [[...l],...qrcode.modules.map(r => [false,...r,false]),[...l]]
      }
    },
  }
</script>
