<template>
  <div v-show="currentValue">
    <div class="weui-mask weui-animate-fade-in"></div>
    <div class="weui-picker weui-animate-slide-up">
      <div class="weui-picker__hd">
        <a class="weui-picker__action" @click="cancel" v-text="cancelText"></a>
        <a class="weui-picker__action" @click="confirm" v-text="confirmText"></a>
      </div>
      <div class="weui-picker__bd">
        <wv-picker-slot v-for="(slot, key, index) in slots" :key="key" :values="slot.values || []" :valueKey="valueKey" :divider="slot.divider" :content="slot.content" v-model="values[slot.valueIndex]"></wv-picker-slot>
      </div>
    </div>
  </div>
</template>

<script>
  import WvPickerSlot from './picker-slot.vue'

  export default {
    name: 'wv-picker',

    components: {
      WvPickerSlot
    },

    props: {
      confirmText: {
        type: String,
        default: '确定'
      },
      cancelText: {
        type: String,
        default: '取消'
      },
      slots: {
        type: Array,
        required: true
      },
      valueKey: String,
      value: Boolean
    },

    data () {
      return {
        currentValue: this.value
      }
    },

    computed: {
      values () {
        let slots = this.slots || []
        let values = []
        slots.forEach(function (slot) {
          if (!slot.divider) values.push(slot.value)
        })
        return values
      },

      slotCount () {
        let slots = this.slots || []
        let count = 0
        slots.forEach((slot) => {
          if (!slot.divider) count++
        })
        return count
      }
    },

    created () {
      this.$on('slotValueChange', this.slotValueChange)
      let slots = this.slots || []
      let values = this.values
      let valueIndexCount = 0
      slots.forEach(slot => {
        if (!slot.divider) {
          slot.valueIndex = valueIndexCount++
          values[slot.valueIndex] = (slot.values || [])[slot.defaultIndex || 0]
          this.slotValueChange()
        }
      })
    },

    methods: {
      slotValueChange () {
        this.$emit('change', this, this.values)
      },

      getSlot (slotIndex) {
        let slots = this.slots || []
        let count = 0
        let target

        let children = this.$children
        children = children.filter(child => child.$options.name === 'wv-picker-slot')

        slots.forEach(function (slot, index) {
          if (!slot.divider) {
            if (slotIndex === count) {
              target = children[index]
            }
            count++
          }
        })
        return target
      },

      getSlotValue (index) {
        let slot = this.getSlot(index)
        if (slot) {
          return slot.value
        }
        return null
      },

      setSlotValue (index, value) {
        this.$nextTick(() => {
          let slot = this.getSlot(index)
          if (slot) {
            slot.currentValue = value
          }
        })
      },

      getSlotValues (index) {
        let slot = this.getSlot(index)
        if (slot) {
          return slot.mutatingValues
        }
        return null
      },

      setSlotValues (index, values) {
        this.$nextTick(() => {
          let slot = this.getSlot(index)
          if (slot) {
            slot.mutatingValues = values
          }
        })
      },

      getValues () {
        return this.values
      },

      setValues (values) {
        values = values || []
        if (this.slotCount !== values.length) {
          throw new Error('values length is not equal slot count.')
        }

        values.forEach((value, index) => {
          this.setSlotValue(index, value)
        })
      },

      cancel () {
        this.$emit('cancel', this)
        this.currentValue = false
      },

      confirm () {
        this.$emit('confirm', this)
        this.currentValue = false
      }
    },

    watch: {
      value (val) {
        this.currentValue = val
      },

      currentValue (val) {
        this.$emit('input', val)
      }
    }
  }
</script>

<style scoped lang="scss">
</style>
