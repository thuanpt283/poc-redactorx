<template>
  <div id="app">
    <ul style="display: flex;">
        <li style="margin: 0 20px;" v-for="(value, key) in templateList" :key="key">
          <img :src="'https://dummyimage.com/150x100/000/fff&text='+key" :data-rx-drop-id="key" draggable>
        </li>
    </ul>
    <RedactorXComponent name="editorArea" :config="configOptions"></RedactorXComponent>
    <hr style="margin: 50px 0"/>
    <h1>Add Template</h1>
    <RedactorXComponent name="templateArea" v-model="content" :config="{}"></RedactorXComponent>
    <button @click="addTemplate">Add template</button>
    <p>{{ content }}</p>
  </div>
</template>

<script>
import RedactorXComponent from "@/RedactorX.vue";

import RedactorX from "@/assets/redactorx.min.js";
export default {
  name: "App",
  components: { RedactorXComponent },
  data() {
    return {
      content: "",
      configOptions: {},
      templateList: {
        salutation: "<p>Hello <mark>{order.client.name}</mark></p>",
        detail: `<table>
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
        signature: "<p>Regards,</p>",
      },
    };
  },
  methods: {
    addTemplate() {
      if(!this.content) return
      this.$set(this.templateList,`item${Object.keys(this.templateList).length}`,this.content)
      this.editorArea.clips.addItem(
        `item${Object.keys(this.templateList).length -1}`,
        `item${Object.keys(this.templateList).length -1}`,
        this.content
      );
    },
    init() {
      const me = this
      Object.assign(this.configOptions, {
        plugins: ["clips"],
        clips: {
          items: {
            salutation: {
              title: "Salutation",
              html: this.templateList.salutation,
            },
            detail: {
              title: "Details of New Order to Client",
              html: this.templateList.detail,
            },
            signature: {
              title: "Signature of New Order to Client",
              html: this.templateList.signature,
            },
          },
        },
        draggable:me.templateList,
        reorder: true,
      });
      RedactorX.add("plugin", "clips", {
        translations: {
          en: {
            clips: {
              clips: "Clips",
            },
          },
        },
        defaults: {
          icon: '<svg height="16" viewBox="0 0 16 16" width="16" xmlns="http://www.w3.org/2000/svg"><path d="m12.1666667 1c1.0193924 0 1.8333333.83777495 1.8333333 1.85714286v10.28571424c0 1.0193679-.8139409 1.8571429-1.8333333 1.8571429h-8.33333337c-1.01868744 0-1.83333333-.8379215-1.83333333-1.8571429v-10.28571424c0-1.01922137.81464589-1.85714286 1.83333333-1.85714286zm-.1666667 2h-8v10h8zm-2 7c.5522847 0 1 .4477153 1 1 0 .5128358-.3860402.9355072-.8833789.9932723l-.1166211.0067277h-4c-.55228475 0-1-.4477153-1-1 0-.5128358.38604019-.9355072.88337887-.9932723l.11662113-.0067277zm0-3c.5522847 0 1 .44771525 1 1 0 .51283584-.3860402.93550716-.8833789.99327227l-.1166211.00672773h-4c-.55228475 0-1-.44771525-1-1 0-.51283584.38604019-.93550716.88337887-.99327227l.11662113-.00672773zm0-3c.5522847 0 1 .44771525 1 1 0 .51283584-.3860402.93550716-.8833789.99327227l-.1166211.00672773h-4c-.55228475 0-1-.44771525-1-1 0-.51283584.38604019-.93550716.88337887-.99327227l.11662113-.00672773z"/></svg>',
          items: !1,
        },
        start: function () {
          this.opts.clips.items &&
            this.app.toolbar.add("clips", {
              title: "## clips.clips ##",
              icon: this.opts.clips.icon,
              command: "clips.popup",
              blocks: {
                all: "editable",
              },
            });
        },
        popup: function (t, i) {
          var p = {};
          for (var s in this.opts.clips.items)
            p[s] = {
              title: this.opts.clips.items[s].title,
              command: "clips.insert",
            };
          this.app.popup.create("clips", {
            items: p,
          }),
            this.app.popup.open({
              button: i,
            });
        },
        insert: function (t, i, p) {
          this.app.popup.close();
          var s = this.opts.clips.items[p].html;
          this.app.editor.insertContent({
            html: s,
          });
        },
        addItem: function (key, title, html) {
          this.opts.clips.items[key] = {
            title,
            html,
          };
          this.opts.draggable[key] = html
          me.$nextTick(() => {
            this.app.editor._buildDraggable();
          })
        },
      });
      
    },
  },
  created() {
    this.init();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
