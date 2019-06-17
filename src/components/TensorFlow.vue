<template>
  <div class="tensor-flow">
    <div style="text-align:center">
      <video hidden ref="vid" id="vid" width="400" height="300"></video>
      <canvas ref="canvas" id="canvas"></canvas>
    </div>
  </div>
</template>

<script>
import * as cocoSSD from '@tensorflow-models/coco-ssd'

export default {
  name: 'tensor-flow',
  data () {
    return {
    }
  },
  mounted () {
    this.webcamInit()
    this.predictWithCocoModel()
  },
  methods: {
    webcamInit: function () {
      this.video = this.$refs.vid
      navigator.mediaDevices
        .getUserMedia({
          audio: false,
          video: { facingMode: 'user' }
        })
        .then(stream => {
          this.video.srcObject = stream
          this.video.onloadedmetadata = () => {
            this.video.play()
          }
        })
    },
    predictWithCocoModel: async function () {
      const model = await cocoSSD.load('lite_mobilenet_v2')
      this.detectFrame(this.video, model)
      console.log('Model loaded...')
    },
    detectFrame: function (video, model) {
      model.detect(video).then(predictions => {
        this.renderPredictions(predictions)
        requestAnimationFrame(() => {
          this.detectFrame(video, model)
        })
      })
    },
    renderPredictions: function (predictions) {
      const canvas = this.$refs.canvas
      const ctx = canvas.getContext('2d')
      canvas.width = 400
      canvas.height = 300
      ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height)

      const font = '16px sans-serif'
      ctx.font = font
      ctx.textBaseline = 'top'
      ctx.drawImage(this.video, 0, 0, 400, 300)

      predictions.forEach(prediction => {
        const x = prediction.bbox[0]
        const y = prediction.bbox[1]
        const width = prediction.bbox[2]
        const height = prediction.bbox[3]
        // Bounding box
        ctx.strokeStyle = '#00FFFF'
        ctx.lineWidth = 2
        ctx.strokeRect(x, y, width, height)
        // Label background
        ctx.fillStyle = '#00FFFF'
        const textWidth = ctx.measureText(prediction.class).width
        const textHeight = parseInt(font, 10)
        ctx.fillRect(x, y, textWidth + 4, textHeight + 4)
      })

      predictions.forEach(prediction => {
        const x = prediction.bbox[0]
        const y = prediction.bbox[1]
        ctx.fillStyle = '#000000'
        ctx.fillText(prediction.class, x, y)
      })
    }
  }
}
</script>
