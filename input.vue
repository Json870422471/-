<template>
  <input
    :style="{ width: width + 'px' }"
    type="text"
    v-if="type == 'text'"
    :value="value"
    @input="textInput"
    @blur="blur"
    @focus="focus"
  />
  <input
    type="text"
    :style="{ width: width + 'px' }"
    v-else-if="type == 'number'"
    :value="value"
    @input="numberInput"
    @blur="blur"
    @focus="focus"
  />
  <input
    type="text"
    v-else-if="type == 'float'"
    :style="{ width: width + 'px' }"
    :value="value"
    @input="floatInput"
    @blur="blur"
    @focus="focus"
  />
</template>

<script>
export default {
  name: "commonInput",
  props: {
    value: {
      type: [String, Number],
      default: "",
    },
    type: {
      type: [String],
      default: "text",
    },
    width: {
      type: [String, Number],
      default: 100,
    },
    min: {
      type: [String, Number],
      default: 0,
    },
    max: {
      type: [String, Number],
      default: Infinity,
    },
    fixed: {
      type: [String, Number],
      default: "",
    },
  },
  methods: {
    blur() {
      this.$emit("blur", this.value);
    },
    focus() {
      this.$emit("focus", this.value);
    },
    textInput(e) {
      this.$emit("input", e.target.value);
    },
    numberInput(e) {
      let val = e.target.value;
      if (isNaN(e.data) || e.data == " ") {
        e.target.value = this.value;
        return;
      }
      if (isNaN(val)) {
        e.target.value = this.min;
        return;
      }
      if (val < this.min) {
        e.target.value = this.min;
        return;
      }
      if (val > this.max) {
        e.target.value = this.max;
        return;
      }
      this.$emit("input", Number(e.target.value));
    },
    floatInput(e) {
      let val = e.target.value;
      if (isNaN(val) || e.data == " ") {
        e.target.value = this.value;
        return;
      }
      if (val < this.min) {
        e.target.value = this.min;
        return;
      }
      if (val > this.max) {
        e.target.value = this.max;
        return;
      }
      if (this.fixed) {
        this.$emit("input", Number(e.target.value).toFixed(this.fixed));
      } else {
        this.$emit("input", Number(e.target.value));
      }
    },
  },
};
</script>

<style lang="scss" scoped>
input {
  height: 32px;
  line-height: 32px;
  text-align: center;
  outline: none;
  -webkit-appearance: none;
  background-color: #fff;
  background-image: none;
  border-radius: 4px;
  border: 1px solid #dcdfe6;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  color: #606266;
  display: inline-block;
  font-size: inherit;
  outline: 0;
  padding: 0 15px;
  -webkit-transition: border-color 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
  transition: border-color 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
  &:focus {
    border-color: #409eff;
  }
}
</style>
