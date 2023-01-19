<template>
    <textarea
        ref="redactorx"
        v-bind="$attrs"
        :name="name"
        :placeholder="placeholder"
        v-model="value"
    />
</template>
<script>
import "@/assets/redactorx.min.css";
import Vue from "vue";
export default {
  name: "Redactor",
  props: {
    value: {
      default: "",
      type: String,
    },
    placeholder: {
      type: String,
      default: null,
    },
    name: {
      type: String,
      default: null,
    },
    config: {
      default: () => {},
      type: Object,
    },
  },
  data() {
    return {
      redactorx: null
    }
  },
  watch: {
    value(newValue, oldValue) {
      if (!this.redactorx.editor.isFocus()) {
        this.redactorx.app.$element.val(newValue);
      }
    },
  },
  mounted() {
    this.init();
  },
  beforeDestroy() {
    this.destroy();
  },
  methods: {
    init() {
      var me = this;
      var subscribe = {
        "editor.change": function (event) {
          var html = event.get("html");
          me.handleInput(html);
          return html;
        },
      };

      // extend config
      if (typeof this.config.subscribe === "undefined") {
        Vue.set(this.config, "subscribe", subscribe);
      } else {
        this.config.subscribe["editor.change"] = subscribe["editor.change"];
      }
      
     
      // call Redactor X
      let app = RedactorX(this.$refs.redactorx, this.config);
      // set instance
      this.redactorx = app;
      this.$parent[this.name] = app;
     
    },
    destroy() {
      // Call destroy on redactor to cleanup event handlers
      RedactorX(this.$refs.redactorx, "destroy");

      // unset instance for garbage collection
      this.redactorx = null;
      this.$parent[this.name] = null;
    },
    handleInput(val) {
      this.$emit("input", val);
    },
  },
};
</script>