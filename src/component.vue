<template>

  <input type="text"
         @input="onInput"
         data-input>

</template>

<script type="text/javascript">
  import Flatpickr from 'flatpickr';
  // You have to import css yourself

  export default {
    name: 'flat-pickr',
    props: {
      value: {
        default: null,
        required: true,
        validator(value) {
          return value === null || value instanceof Date || typeof value === 'string' || value instanceof String || value instanceof Array || typeof value === 'number'
        }
      },
      // https://chmln.github.io/flatpickr/options/
      config: {
        type: Object,
        default: () => ({
          wrap: false
        })
      },
    },
    data() {
      return {
        /**
         * The flatpickr instance
         */
        fp: null,
        /**
         * config.onChange method backup
         */
        oldOnChange: null,
        /**
         * config.onClose method backup
         */
        oldOnClose: null,
      };
    },
    mounted() {
      // Return early if flatPickr is already loaded
      /* istanbul ignore if */
      if (this.fp) return;

      // Backup original handler
      this.oldOnChange = this.config.onChange;
      this.oldOnClose = this.config.onClose;
      // Hook our handler
      this.config.onChange = this.onChange;
      this.config.onClose = this.onClose;

      // Init flatpickr
      this.fp = new Flatpickr(this.getElem(), this.config);
      // Set initial date
      this.fp.setDate(this.value, true);

    },
    methods: {
      /**
       * Get the HTML node where to bind the flatpickr
       * Bind on parent element if wrap is true
       */
      getElem() {
        return this.config.wrap ? this.$el.parentNode : this.$el
      },

      /**
       * Emit on-change event
       */
      onChange(...args) {
        // Call original handler if registered
        /* istanbul ignore else */
        if (typeof this.oldOnChange === 'function') {
          this.oldOnChange(...args);
        }
        this.$emit('on-change', ...args);
      },

      /**
       * Emit on-close event
       */
      onClose(...args) {
        if (this.config.allowInput && !this.config.altInput) {
            /* Update the FlatPicker instance and redraw */
            this.fp.setDate(this.fp.input.value, true, this.config.dateFormat);
        }
        if (this.config.allowInput && this.config.altInput) {
            /* Update the FlatPicker instance with the value from altInput and redraw */
            this.fp.setDate(args[2].altInput.value, true, this.config.altFormat);
        }
      
        // Call original handler if registered
        /* istanbul ignore else */
        if (typeof this.oldOnClose === 'function') {
          this.oldOnClose(...args);
        }
        this.$emit('on-close', ...args);
      },

      /**
       * Watch for value changed by date-picker itself and notify parent component
       *
       * @param event
       */
      onInput(event) {
        this.$emit('input', this.fp.selectedDates[0]);
      },
    },
    watch: {
      /**
       * Watch for any config changes and redraw date-picker
       *
       * @param newConfig Object
       */
      config(newConfig) {
        this.fp.config = Object.assign({}, this.fp.config, newConfig);
        // Note: jumpToDate method also calls redraw()
        this.fp.jumpToDate();
      },

      /**
       * Watch for changes from parent component and update DOM
       *
       * @param newValue
       */
      value(newValue) {
        // Prevent updates if v-model value is same as input's current value
        //if (newValue === this.$el.value) return;
        if (this.fp.selectedDates.length>0 && newValue === this.fp.selectedDates[0]) return;
        // Make sure we have a flatpickr instance
        this.fp &&
        // Notify flatpickr instance that there is a change in value
        this.fp.setDate(newValue, true);
      }
    },
    beforeDestroy() {
      // Free up memory
      /* istanbul ignore else */
      if (this.fp) {
        this.fp.destroy();
        this.fp = null;
        this.oldOnChange = null;
        this.config.onChange = null
      }
    },
  };
</script>
