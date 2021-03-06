<template>
  <Select
    v-model="cascaderSelect.formData[propKey]"
    clearable
    filterable
    @on-change="changeHandler"
    @on-open-change="openChangeHandler"
    v-on="$listeners"
    v-bind="$attrs"
    :disabled="userDisabled || disabled"
  >
    <template v-for="item in options">
      <Option :value="item.value" :key="item.value"
        >{{ item["label"] }}({{ item["value"] }})</Option
      >
    </template>
  </Select>
</template>
<script>
import Emitter from "../../libs/emitter.js";

// 1.不提供校验功能 ，校验功能应该在auto-form上
export default {
  name: "CascaderSelectItem",
  mixins: [Emitter],
  inject: ["cascaderSelect"],
  props: {
    value: {
      type: Object,
      default: () => {
        return {};
      }
    },
    pId: {
      type: [String, Number],
      default: ""
    },
    disabled: {
      type: Boolean,
      default: false
    },

    propKey: {
      type: String,
      default: ""
    },
    loadData: {
      type: Function
    },
    message: {
      type: String,
      default: ""
    },
    // 针对集团酒店联动组件权限的disabled是否使用
    userLoginDisabled: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      options: [],
      userDisabled: false,
      parentHasValue: false,
      isOpenChange: false,
      isRequest: true //是否可以重新请求加载
    };
  },
  watch: {
    formDataValue: {
      handler(val, oldVal) {
        if (this.isOpenChange) {
          return;
        }
        if (val && val !== oldVal) {
          const notRequest = this.options.some(item => item.value === val);
          if (!notRequest) {
            this.getData();
          }
        } else {
          this.isOpenChange = false;
        }
      },
      immediate: true
    }
  },
  computed: {
    formDataValue() {
      return this.cascaderSelect.formData[this.propKey];
    }
  },
  methods: {
    reset() {
      this.changeRequest(true);
      this.clearOptions(true);
    },

    changeRequest(boolean) {
      this.isRequest = boolean;
    },
    clearOptions() {
      this.options.length = 0;
    },
    async getData() {
      const map = {};
      try {
        const loadDataPromise = this.loadData(this.cascaderSelect.formData);
        return loadDataPromise.then(data => {
          data = data.filter(item => {
            if (!map[item.value]) {
              map[item.value] = true;
              return item;
            }
          });

          this.$set(this, "options", data);
          this.changeRequest(false);
        });
      } catch (error) {
        console.error("loadData需要返回一个Promise", error);
      }
    },
    openChangeHandler(boolean) {
      if (boolean) {
        this.isOpenChange = true;
        if (this.pId && !this.parentHasValue) {
          this.pId &&
            this.pId.split(",").forEach(item => {
              this.dispatch("CascaderSelect", "on-cascader-item-open-change", {
                id: this.propKey,
                pId: item
              });
            });
        } else {
          if (this.isRequest) {
            this.getData();
          }
        }
      } else {
        this.isOpenChange = false;
      }
    },
    changeHandler(value) {
      const pId = this.pId ? this.pId.split(",") : [];
      this.dispatch("CascaderSelect", "on-cascader-item-clear-value", {
        pId,
        value,
        id: this.propKey
      });
    }
  },
  created() {},
  mounted() {
    this.dispatch("CascaderSelect", "on-cascader-item-add", this);
    this.$on("set-union", obj => {
      this.cascaderSelect.formData[obj.key] = obj.value;
    });
    this.$on("set-group", obj => {
      this.cascaderSelect.formData[obj.key] = obj.value;
    });
    this.$on("set-hotel", obj => {
      this.cascaderSelect.formData[obj.key] = obj.value;
    });
    this.$on("disabled-union", obj => {
      this.userDisabled = obj.disabled;
    });
    this.$on("disabled-group", obj => {
      this.userDisabled = obj.disabled;
    });
    this.$on("disabled-hotel", obj => {
      this.userDisabled = obj.disabled;
    });
  },
  beforeDestroy() {
    this.dispatch("CascaderSelect", "on-cascader-item-remove", this);
  },
  destroyed() {}
};
</script>

<style lang="less" scoped></style>
