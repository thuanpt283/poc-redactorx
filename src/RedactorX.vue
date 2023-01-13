<template>
    <div>
        <ul style="display: flex;">
        <li style="margin: 0 20px;" v-for="(value, key, index) in templateList" :key="key" @click="insertHTML"><img :src="'https://dummyimage.com/150x100/000/fff&text='+key" :data-rx-drop-id="'card-0'+index"></li>
    </ul>
    <textarea
        ref="redactorx"
        v-bind="$attrs"
        :name="name"
        :placeholder="placeholder"
        :value="value"
    />
    <div 
    v-for="(value, key, index) in templateList" :key="key"
    :data-rx-drop-item="'card-0'+index" 
    style="display: none" 
    v-html="value"></div>
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
            salutation : '<p>Hello <mark>{order.client.name}</mark></p>',
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
                </table>`,
            signature:'<p>Regards,</p>'
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
        RedactorX.add("plugin", "clips", {
        translations: {
            en: {
                clips: {
                    clips: "Clips"
                }
            }
        },
        defaults: {
            icon: '<svg height="16" viewBox="0 0 16 16" width="16" xmlns="http://www.w3.org/2000/svg"><path d="m12.1666667 1c1.0193924 0 1.8333333.83777495 1.8333333 1.85714286v10.28571424c0 1.0193679-.8139409 1.8571429-1.8333333 1.8571429h-8.33333337c-1.01868744 0-1.83333333-.8379215-1.83333333-1.8571429v-10.28571424c0-1.01922137.81464589-1.85714286 1.83333333-1.85714286zm-.1666667 2h-8v10h8zm-2 7c.5522847 0 1 .4477153 1 1 0 .5128358-.3860402.9355072-.8833789.9932723l-.1166211.0067277h-4c-.55228475 0-1-.4477153-1-1 0-.5128358.38604019-.9355072.88337887-.9932723l.11662113-.0067277zm0-3c.5522847 0 1 .44771525 1 1 0 .51283584-.3860402.93550716-.8833789.99327227l-.1166211.00672773h-4c-.55228475 0-1-.44771525-1-1 0-.51283584.38604019-.93550716.88337887-.99327227l.11662113-.00672773zm0-3c.5522847 0 1 .44771525 1 1 0 .51283584-.3860402.93550716-.8833789.99327227l-.1166211.00672773h-4c-.55228475 0-1-.44771525-1-1 0-.51283584.38604019-.93550716.88337887-.99327227l.11662113-.00672773z"/></svg>',
            items: !1
        },
        start: function() {
            this.opts.clips.items && this.app.toolbar.add("clips", {
                title: "## clips.clips ##",
                icon: this.opts.clips.icon,
                command: "clips.popup",
                blocks: {
                    all: "editable"
                }
            })
        },
        popup: function(t, i) {
            var p = {};
            for (var s in this.opts.clips.items)
                p[s] = {
                    title: this.opts.clips.items[s].title,
                    command: "clips.insert"
                };
            this.app.popup.create("clips", {
                items: p
            }),
            this.app.popup.open({
                button: i
            })
        },
        insert: function(t, i, p) {
            this.app.popup.close();
            var s = this.opts.clips.items[p].html;
            this.app.editor.insertContent({
                html: s
            })
        }
    });

    Object.assign(this.config, {
        plugins: ["clips"],
        clips: {
            items: {
                salutation: {
                    title: 'Salutation',
                    html: me.templateList.salutation,
                },
                detail: {
                    title: 'Details of New Order to Client',
                    html: me.templateList.detail,
                },
                signature: {
                    title: 'Signature of New Order to Client',
                    html: me.templateList.signature,
                }
            }
        },
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