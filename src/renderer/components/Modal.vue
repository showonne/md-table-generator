<template>
  <div>
    <input class="modal-state" :id="id" type="checkbox">
      <div class="modal">
      <label class="modal-bg" :for="id"></label>
      <div class="modal-body">
          <label class="btn-close" :for="id">X</label>
          <h4 class="modal-title" v-if="$slots.title">
            <slot name="title"></slot>
          </h4>
          <h5 class="modal-subtitle" v-if="$slots.subtitle">
            <slot name="subtitle"></slot>
          </h5>
          <div class="row flex-edges" :class="{'flex-end': !isConfirm}" style="margin-bottom: 0;">
            <label :for="id" class="modal-link">{{cancelText}}</label>
            <label v-if="isConfirm" :for="id" class="modal-link text-danger" @click="confirm">Confirm</label>
          </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'modal',
  props: {
    value: {
      type: Boolean
    },
    id: String,
    isConfirm: Boolean,
    cancelText: {
      type: String,
      default: 'Cancel'
    }
  },
  methods: {
    confirm () {
      this.$emit('confirm')
    }
  }
}
</script>

<style lang="less">
  .row.flex-end{
    justify-content: flex-end;
  }
</style>

<style lang="less">
.modal-link.text-danger{
  background-image: linear-gradient(5deg, transparent 65%, #a7342d 80%, transparent 90%),linear-gradient(165deg, transparent 5%, #a7342d 15%, transparent 25%),linear-gradient(165deg, transparent 45%, #a7342d 55%, transparent 65%),linear-gradient(15deg, transparent 25%, #a7342d 35%, transparent 50%)
}
</style>
