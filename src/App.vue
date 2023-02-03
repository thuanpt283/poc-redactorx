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
    <RedactorXComponent name="templateArea" v-model="content" :config="templateOptions"></RedactorXComponent>
    <button @click="addTemplate">Add template</button>
    <p>{{ content }}</p>
  </div>
</template>

<script>
import RedactorXComponent from "@/RedactorX.vue";
import '@/assets/redactorx.min.js';
import '@/assets/plugins/inlineformat.min.js';
import '@/assets/plugins/templates.min.js';
import '@/assets/plugins/variables.min.js';

export default {
  name: "App",
  components: { RedactorXComponent },
  data() {
    return {
      content: "",
      listVariable: [
        {
          title: 'Last Name',
          html: 'last_name'
        },
        {
          title: 'First Name',
          html: 'first_name'
        },
        {
          title: 'Appraiser ID',
          html: 'appraiser_id'
        },
        {
          title: 'Borrower',
          html: 'borrower'
        },
      ],
      configOptions: {},
      templateOptions: {},
      templateList: {
        salutation: "<p>Hello <span class='redspan' contenteditable='false'>{order.client.name}</span></p>",
        detail: `<table>
                  <tbody>
                      <tr>
                          <th>Borrower:</th>
                          <td><span class='redspan' contenteditable='false'>{order.borrower.last_name}</span></td>
                      </tr>
                      <tr>
                          <th>Lender:</th>
                          <td><span class='redspan' contenteditable='false'>{order.lender.name}</span></td>
                      </tr>
                      <tr>
                          <th>Address:</th>
                          <td><span class='redspan' contenteditable='false'>{order.address.full_address}</span></td>
                      </tr>
                      <tr>
                          <th>Loan Number:</th>
                          <td><span class='redspan' contenteditable='false'>{order.lender_loan_number}</span></td>
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
      this.content = this.content.replaceAll('data-contenteditable','contenteditable')
      this.$set(this.templateList,`item${Object.keys(this.templateList).length}`,this.content)

      this.editorArea.opts.templates.items[`item${Object.keys(this.templateList).length -1}`] = {
        title: `item${Object.keys(this.templateList).length -1}`,
        html: this.content,
      };
      this.editorArea.opts.draggable[`item${Object.keys(this.templateList).length -1}`] = this.content
      this.$nextTick(() => {
        this.editorArea.editor._buildDraggable();
        this.templateArea.editor.setEmpty()
      })
    },
    init() {
      const me = this
      Object.assign(this.configOptions, {
        plugins: ["templates","variables"],
        templates: {
          title: 'Templates',
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
        variables: {
          items: this.listVariable,
          custom: true,
        },
        subscribe: {
            'variables.insert': function(event) {
                var html = event.get('html');
                this.app.editor.insertContent({
                    html: `<span style="color: #fff; background: #ff3265; padding: 2px 8px; border-radius: 3px;" contenteditable="false" data-contenteditable="false">{${html}}</span>`,
                });
            }
        },
        draggable:me.templateList,
        reorder: true,
       
      });
      Object.assign(this.templateOptions, {
        plugins: ["inlineformat","variables"],
        inlineformat: {
            items: ['redspan'],
            itemsObj: {
                "redspan": {
                    title: '<span style="color: #fff; background: #ff3265; padding: 2px 8px; border-radius: 3px;">Redmark</span>',
                    params: {
                        tag: 'span',
                        attr: {
                            'class': 'redspan',
                            'data-contenteditable': 'false'
                        }
                    }
                },
            }
        },
        variables: {
          items: this.listVariable,
          custom: true,
        },
        subscribe: {
            'variables.insert': function(event) {
                var html = event.get('html');
                this.app.editor.insertContent({
                    html: `<span style="color: #fff; background: #ff3265; padding: 2px 8px; border-radius: 3px;" contenteditable="false" data-contenteditable="false">{${html}}</span>`,
                });
            }
        }
      })
      
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
.rx-content .redspan {
  display: inline-block;
  color: #fff;
  background: #ff3265;
  padding: 2px 8px;
  border-radius: 3px;
}
</style>
