<template>
  <div id="gui" class="fixed-top w-100 py-3">
    <div class="row no-gutters text-center text-white">
      <div class="col-4">
        <div class="score">
          <div class="strokes">strokes: {{strokes}}</div>
          <div class="par">par: {{stage.par}}</div>
        </div>  
      </div>
      <div class="col-4">
        <div class="hole">hole {{stage.id}}</div>
      </div>
      <div class="col-4"><div class="options"><i class="mdi mdi-settings"></i></div></div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Gui',
  props: {
    strokes: Number,
    stage: Object
  }
}
</script>