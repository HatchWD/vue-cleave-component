<template>
  <input :type="type">
</template>

<script>
  import Cleave from 'cleave.js'

  export default {
    name: 'cleave',
    props: {
      value: {
        default: null,
        required: true,
        validator(value) {
          return value === null || typeof value === 'string' || value instanceof String || typeof value === 'number'
        }
      },
      // https://github.com/nosir/cleave.js/blob/master/doc/options.md
      options: {
        type: Object,
        default: () => ({})
      },
      // Set this prop to false to emit masked value
      raw: {
        type: Boolean,
        default: true
      },
      type: {
        type: String,
        default: 'text'
      }
    },
    data() {
      return {
        // cleave.js instance
        cleave: null
      }
    },
    mounted() {
      /* istanbul ignore if */
      if (this.cleave) return;

      this.cleave = new Cleave(this.$el, this.getOptions(this.options));
      this.cleave.setRawValue(this.value)
    },
    methods: {
      /**
       * Inject our method in config options
       *
       * @param options Object
       */
      getOptions(options) {
        return Object.assign({}, options, {
          onValueChanged: this.onValueChanged
        });
      },
      /**
       * Watch for value changed by cleave and notify parent component
       *
       * @param event
       */
      onValueChanged(event) {
        let value = this.raw ? event.target.rawValue : event.target.value;
        this.$emit('input', value);
      },
    },
    watch: {
      /**
       * Watch for any changes in options and redraw
       *
       * @param newOptions Object
       */
      options: {
        deep: true,
        handler(newOptions) {
          this.cleave.destroy();
          this.cleave = new Cleave(this.$el, this.getOptions(newOptions));
          this.cleave.setRawValue(this.value)
        }
      },

      /**
       * Watch for changes from parent component and notify cleave instance
       *
       * @param newValue
       */
      value(newValue) {
        /* istanbul ignore if */
        if (!this.cleave) return;

        // when v-model is not masked (raw)
        if (this.raw && newValue === this.cleave.getRawValue()) return;
        //  when v-model is masked (NOT raw)
        if (!this.raw && newValue === this.$el.value) return;
        // Lastly set newValue
        this.cleave.setRawValue(newValue);
      }
    },
    /**
     * Free up memory
     */
    beforeDestroy() {
      /* istanbul ignore if */
      if (!this.cleave) return;

      this.cleave.destroy();
      this.cleave = null
    },
  }
</script>
