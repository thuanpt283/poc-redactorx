<template>
    <div>
    <textarea
        ref="redactorx"
        v-bind="$attrs"
        :name="name"
        :placeholder="placeholder"
        :value="value"
    />


    <ul style="display: flex;">
        <li v-for="(value, key, index) in templateList" :key="key" @click="insertHTML"><img :src="'https://dummyimage.com/300x200/000/fff&text='+key" :data-rx-drop-id="'card-0'+index"></li>
    </ul>
    <div 
    v-for="(value, key, index) in templateList" :key="key"
    :data-rx-drop-item="'card-0'+index" 
    style="display: none" 
    v-html="value"></div>
    <!-- <div data-rx-drop-item="card-01" style="display: none" v-html="templateList.salutation"></div>
    <div data-rx-drop-item="card-02" style="display: none" v-html="templateList.detail"></div>
    <div data-rx-drop-item="card-03" style="display: none" v-html="templateList.signature"></div> -->
    </div>
</template>
<script>
import RedactorX from "@/assets/redactorx.js";
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
      default: {},
      type: Object,
    },
  },
  data() {
    return {
        templateList: {
            salutation : '<p>Hello <mark>{order.client.name}</mark></p><br/>',
            detail:`<table>
                    <tbody>
                        <tr>
                            <th>Borrower:</th>
                            <td><mark>{order.borrower.last_name}</mark></td>
                        </tr>
                        <tr>
                            <th>Lender:</th>
                            <td><mark>{order.lender.name}</mark></td>
                        </tr>
                        <tr>
                            <th>Address:</th>
                            <td><mark>{order.address.full_address}</mark></td>
                        </tr>
                        <tr>
                            <th>Loan Number:</th>
                            <td><mark>{order.lender_loan_number}</mark></td>
                        </tr>
                        <tr>
                            <th>Product:</th>
                            <td>Conventional USDA to FHA Conversion</td>
                        </tr>
                    </tbody>
                </table><br/>`,
            signature:'<p>Regards,</p><br/>'
        }
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
    RedactorX.add("plugin", "myplugin", {
      start: function () {
        this.app.topbar.add("mybutton", {
          title: "My Button",
          icon: '<i class="fa fa-superpowers"></i>',
          command: "myplugin.toggle",
        });
      },
      toggle: function (params, button) {
        alert("Button Toggle");
      },
    });
  },
  beforeDestroy() {
    this.destroy();
  },
  methods: {
    insertHTML(block) {
        this.redactorx.editor.insertContent({ html: block});
    },
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
      RedactorX.add("plugin", "myplugin", {
        start: function () {
          this.app.toolbar.add("mybutton", {
            title: "My Button",
            icon: '<i class="fa fa-file-alt"></i>',
            command: 'myplugin.popup',
            params: {
              arg1: 'value1',
              arg2: 'value2',
            },
          });
        },
        popup: function(params, button, name, e) {
            // create
            this.app.popup.create('myplugin', {
                items: {
                    salutation: {
                        title: 'Salutation',
                        content: me.templateList.salutation,
                        command: 'myplugin.toggle'
                    },
                    detail: {
                        title: 'Details of New Order to Client',
                        content: me.templateList.detail,
                        command: 'myplugin.toggle'
                    },
                    signature: {
                        title: 'Signature of New Order to Client',
                        content: me.templateList.signature,
                        command: 'myplugin.toggle'
                    }
                },
            });

            // open
            this.app.popup.open({ button: button });
        },
        toggle: function (params, button, name, e) {
          // get params

          me.insertHTML(button.params.content)
          this.app.popup.close();
          console.log(button.params.content);
        },
      });
      Object.assign(this.config, {
        plugins: ["myplugin"],
        context: true,
        control: true,
        reorder: true
    })
    
      // call Redactor X
      var app = RedactorX(this.$refs.redactorx, this.config);

      // set instance
      this.redactorx = app;
      this.$parent.redactorx = app;
     
    },
    destroy() {
      // Call destroy on redactor to cleanup event handlers
      RedactorX(this.$refs.redactorx, "destroy");

      // unset instance for garbage collection
      this.redactorx = null;
      this.$parent.redactorx = null;
    },
    handleInput(val) {
      this.$emit("input", val);
    },
  },
};
</script>