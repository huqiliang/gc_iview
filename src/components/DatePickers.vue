<script>
import moment from "moment";
import _ from "lodash";
export default {
  name: "DatePickers",
  props: {
    value: {
      type: Object,
      default() {
        return {
          startDate: moment().format("YYYY-MM-DD"),
          endDate: moment()
            .add(1, "d")
            .format("YYYY-MM-DD")
        };
      }
    },
    options: {
      type: Object,
      default() {
        return {
          startOptions: {
            disabledDate(date) {
              return date && date.valueOf() < Date.now() - 86400000;
            }
          }
        };
      }
    },
    required: {
      type: Boolean,
      default: true
    },
    startDatePlaceHolder: {
      type: String,
      default: "开始日期"
    },
    endDatePlaceHolder: {
      type: String,
      default: "结束日期"
    }
  },
  methods: {
    changeValue(type, value) {
      switch (type) {
        case "startChange":
          if (moment(value).isAfter(this.value.endDate)) {
            this.$set(
              this.value,
              "endDate",
              moment(value)
                .add(1, "day")
                .format("YYYY-MM-DD")
            );
          }
          break;
        case "endChange":
          if (moment(value).isBefore(this.value.startDate)) {
            let beforeToday = moment(value).subtract(1, "day");
            if (!beforeToday.isBefore(moment())) {
              this.$set(
                this.value,
                "startDate",
                beforeToday.format("YYYY-MM-DD")
              );
            } else {
              this.$set(this.value, "startDate", moment().format("YYYY-MM-DD"));
            }
          }
          break;
      }
      this.$emit("input", this.value);
    },
    validator(key, callback) {
      if (_.isEmpty(_.get(this.value, key))) {
        callback(new Error(`请输入${key ? "开始" : "结束"}时间`));
      } else {
        callback();
      }
    },
    validate(fn) {
      this.$refs["datePickersForm"].validate(fn);
    },
    resetFields() {
      this.$refs["datePickersForm"].resetFields();
    },
    validateMessage(rule, value, callback, isStart) {
      let val = this.value.endDate;
      let msg = "结束时间不能为空";
      if (isStart === true) {
        val = this.value.startDate;
        msg = "开始时间不能为空";
      }
      if (!val && this.required) {
        callback(new Error(msg));
      } else {
        callback();
      }
    }
  },
  render() {
    const { options } = this.$props;
    const validateStartEmpty = (rule, value, callback) =>
      this.validateMessage(rule, value, callback, true);
    const validateEndEmpty = (rule, value, callback) =>
      this.validateMessage(rule, value, callback, false);
    return (
      <div class="container">
        {this.value ? (
          <Form
            props={{
              model: this.value
            }}
            class="datePickers"
            on={{
              Input: val => {
                console.log(val);
              }
            }}
            ref="datePickersForm"
          >
            {options && options.startLabel ? (
              <span class="ml10">{options.startLabel}</span>
            ) : null}
            <FormItem
              prop="startDate"
              rules={{
                validator: validateStartEmpty,
                trigger: "blur,change"
              }}
            >
              <DatePicker
                {...{ type: "date", attrs: this.$attrs }}
                transfer={true}
                show-week-numbers
                value={this.value.startDate}
                placeholder={this.startDatePlaceHolder}
                onInput={value => {
                  this.$set(
                    this.value,
                    "startDate",
                    value ? moment(value).format("YYYY-MM-DD") : ""
                  );
                  this.changeValue("startChange", value);
                }}
                options={this.options.startOptions}
              />
            </FormItem>
            {options && options.endLabel ? (
              <span class="ml10">{options.endLabel}</span>
            ) : null}
            <FormItem
              class="ml10"
              prop="endDate"
              rules={{
                validator: validateEndEmpty,
                trigger: "blur,change"
              }}
            >
              <DatePicker
                {...{ type: "date", attrs: this.$attrs }}
                show-week-numbers
                value={this.value.endDate}
                placeholder={this.endDatePlaceHolder}
                onInput={value => {
                  this.$set(
                    this.value,
                    "endDate",
                    value ? moment(value).format("YYYY-MM-DD") : ""
                  );
                  this.changeValue("endChange", value);
                }}
                transfer={true}
                options={this.options.startOptions}
              />
            </FormItem>
          </Form>
        ) : null}
      </div>
    );
  }
};
</script>
<style lang="less" scoped>
.container {
  width: 100%;
  .ivu-form-item-content,
  .ivu-date-picker {
    width: 100%;
  }
  .ivu-form-item {
    flex: 1;
    margin-right: 0;
  }
  .ml10 {
    margin-left: 10px;
  }
  .datePickers {
    display: flex;
    flex-direction: row;
    width: 100%;
  }
}
</style>
