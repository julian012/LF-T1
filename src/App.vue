<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      Taller Número 1
    </v-app-bar>
    <v-content>
      <v-row>
        <v-col cols="10" offset="1">
          <v-card>
            <v-toolbar dark color="orange" dense>
              <v-toolbar-title>
                Configurar Gramatica
              </v-toolbar-title>
            </v-toolbar>
            <v-card-text>
              <v-row>
                <v-col cols="6">
                  <v-row>
                    <v-col offset="1" cols = 10>
                      <v-text-field
                              label="Variables (separado por comas)"
                              v-model="variables"
                      />
                    </v-col>
                    <v-col offset="1"  cols = 10>
                      <v-text-field
                              label="Alfabeto (separado por comas)"
                              v-model="alphabet"
                      />
                    </v-col>
                    <v-col offset="1"  cols = 10>
                      <v-text-field
                              label="Simbolo inicial"
                              v-model="initial"
                      />
                    </v-col>
                  </v-row>
                </v-col>
                <v-col cols="6">
                  <v-row>
                    <v-col offset="1" cols = 10>
                      <h2>Producciones</h2>
                    </v-col>
                    <v-col offset="1"  cols = 3>
                      <v-text-field
                              label="Variable"
                              v-model="production.variable"
                      />
                    </v-col>
                    <v-col cols="1" align-self="center">
                      <h1>&#8594;</h1>
                    </v-col>
                    <v-col offset="1"  cols = 3>
                      <v-text-field
                              label="Alfabeto"
                              v-model="production.alphabet"
                      />
                    </v-col>
                    <v-col cols = 3 align-self="center">
                      <v-btn color="success" outlined @click="addProduction" >Agregar</v-btn>
                    </v-col>
                    <v-col cols="12">
                      <v-simple-table dense>
                        <template v-slot:default>
                          <thead>
                          <tr>
                            <th class="text-center">No</th>
                            <th class="text-center">Produccion</th>
                            <th class="text-center">Acciones</th>
                          </tr>
                          </thead>
                          <tbody>
                          <tr v-for="(item, index) in productions" :key="item.name">
                            <td class="text-center" >{{ index + 1 }}</td>
                            <td class="text-center pro">{{item.variable}} <h1>&#8594;</h1> {{item.alphabet}} </td>
                            <td class="text-center">
                              <v-btn
                                      color="error"
                                      icon
                                      x-small
                                      @click="deleteProduction(index)"
                              >
                                <v-icon>mdi-close</v-icon>
                              </v-btn>
                            </td>
                          </tr>
                          </tbody>
                        </template>
                      </v-simple-table>
                    </v-col>
                  </v-row>
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
        <v-col cols="10" offset="1">
          <v-card>
            <v-toolbar dark color="orange" dense>
              <v-toolbar-title>
                Validar una palabra
              </v-toolbar-title>
            </v-toolbar>
            <v-card-text>
              <v-row>
                <v-col offset="1"  cols = 3>
                  <v-text-field
                          label="Palabra"
                          v-model="search"
                  />
                </v-col>
                <v-col cols = 3 align-self="center">
                  <v-btn color="success" outlined @click="makeTree">Validar</v-btn>
                </v-col>
                <v-col cols="3" offset="1" v-if="word_list.length > 0">
                  <v-alert :type="(isExist) ? 'success': 'error'" dense outlined>
                    {{getText}}
                  </v-alert>
                </v-col>
              </v-row>
              <v-row class="overflow-auto">
                <TreeChart :json="treeData" />
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-content>
  </v-app>
</template>

<script>

  import TreeChart from "vue-tree-chart";

export default {
  name: 'App',
  components: {
    TreeChart
  },
  data() {
    return {
      max_level: 0,
      treeData: {},
      production: {
        variable: '',
        alphabet: ''
      },
      word_list: [],
      variables: 'S,A,B',
      alphabet: 'a,b,c',
      initial: 'S',
      search: 'aa',
      productions: [
        {
          variable: 'S',
          alphabet: 'A'
        },
        {
          variable: 'S',
          alphabet: 'B'
        },
        {
          variable: 'A',
          alphabet: 'aA'
        },
        {
          variable: 'A',
          alphabet: 'b'
        },
        {
          variable: 'B',
          alphabet: 'aA'
        }
      ]
    }
  },
  methods: {
    addProduction() {
      if(this.production.alphabet.length === 0 || this.production.variable.length === 0) return
      this.productions.push(JSON.parse(JSON.stringify(this.production)))
      this.cleanProductionInput()
    },
    cleanProductionInput() {
      this.production.variable = ''
      this.production.alphabet = ''
    },
    deleteProduction(index) {
      this.productions.splice(index, 1)
    },
    async makeTree(){
      this.max_level = this.search.length + 2
      this.word_list = []
      let children = await this.getChildren(this.initial, 1)
      this.treeData = {
        name: this.initial,
        children
      }
    },
    async getChildren(word, level) {
      let children = []
      const next_level = level + 1
      const { variable, sub_word } = await this.extractVariable(word)
      if(variable){ //S
        for (const p of this.productions) {
          if(p.variable === variable) {
            const name = sub_word + p.alphabet //aaaA
            if(this.isWord(name)) this.word_list = [...this.word_list, name]
            const sub_children = (level !== this.max_level) ? await this.getChildren(name, next_level) : null
            children = [...children, {
              name,
              children: sub_children
            }]
          }
        }
      }
      return children;
    },
    async extractVariable(word) {
      const variable = await this.variables.includes(word.charAt(word.length - 1)) ? word.charAt(word.length - 1) : null
      const sub_word = (variable) ? word.replace(variable,'') : word
      return {
        variable,
        sub_word 
      }
    },
    isWord(sub_word) {
      for (let i = 0; i < sub_word.length; i++){
        if(this.variables.includes(sub_word.charAt(i))) return false
      }
      return true
    }
  },
  computed: {
    isExist(){
      return this.word_list.includes(this.search)
    },
    getText(){
      return (this.word_list.includes(this.search)) ? 'La palabra si pertenece' : 'La palabra no pertenece'
    }
  }
};
</script>
<style scoped>
  .pro {
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>



