<template>
  <div id="app">
    <img id="img" src="./assets/se.jpg" width="400px">
    <el-input
            id="search_input"
            type="textarea"
            autosize
            placeholder="请输入内容"
            v-model="search_content"
            @input="onSearch">
    </el-input>
    <div class="tag-group">
      <el-tag type="success">exists concept</el-tag>
      <el-tag type="warning">learned concept</el-tag>
    </div>
    <div class="tag-group" id="tagged_content">

      <el-tag
              v-for="item in context_concepts"
              :key="item.label"
              :type="item.type"
              @click="onTag(item.label)">
        {{item.label}}
      </el-tag>
    </div>
    <el-dialog
      title="概念解释"
      :visible.sync="dialogVisible">
      <!--<el-form-->
              <!--ref="concept_detail"-->
              <!--:model="concept_detail">-->
        <!--<el-form-item label="alias:">-->
          <!--{{concept_detail.alias}}-->
        <!--</el-form-item>-->
        <!--<el-form-item label="query:">-->
          <!--{{concept_detail.query}}-->
        <!--</el-form-item>-->
        <!--<el-form-item label="tag">-->
          <!--{{concept_detail.tag}}-->
        <!--</el-form-item>-->
        <!--<el-form-item label="def">-->
          <!--{{concept_detail.def}}-->
        <!--</el-form-item>-->
      <!--</el-form>-->
      <strong>alias: </strong> {{concept_detail.alias}} <br />
      <strong>query: </strong> {{concept_detail.query}} <br />
      <strong>tag: </strong> {{concept_detail.tag}} <br />
      <strong>def: </strong><br /> {{concept_detail.def}} <br />
    </el-dialog>
  </div>
</template>

<script>
  import axios from 'axios'
  // axios.defaults.crossDomain = true
  const tag_url = "http://10.131.200.135:23333/ner/inference"
  const def_url = "http://liu_huangshu.stefans.live:7000"
  export default {
    name: 'app',
    data() {
      return {
        search_content: '',
        context_concepts: [],
        dialogVisible: false,
        concept_detail: {
          alias: '',
          query: '',
          tag: '',
          def: ''
        }
      }
    },
    methods: {
      words_match(pattern_words, words) {
        let matched_words_list = pattern_words.slice(0)
        let i = -1
        let new_matched_words_list = []
        while (matched_words_list.length !== 0) {
          i++
          for (let matched_words of matched_words_list) {
            let matched_words_split = matched_words.split(' ')
            if (matched_words_split.length > i && matched_words_split[i] === words[i]) {
              new_matched_words_list.push(matched_words)
            }
          }
          matched_words_list = new_matched_words_list
          new_matched_words_list = []
        }
        return i
      },
      onSearch() {
        axios.get(tag_url, {
          params: {
            context: this.search_content
          }
        }).then(response => {
          let matched_concepts = response.data["matched_concepts"]
          let new_concepts = response.data["new_concepts"]
          this.context_concepts = []
          for (let i = 0; i < matched_concepts.length; i++) {
            this.context_concepts.push({
              label: matched_concepts[i],
              type: 'success'
            })
          }
          for (let i = 0; i < new_concepts.length; i++) {
            this.context_concepts.push({
              label: new_concepts[i],
              type: 'warning'
            })
          }
        })
      },
      onTag(concept) {
        axios.get(def_url, {
          params: {
            keyword: concept
          }
        }).then(response => {
          this.concept_detail.alias = response.data["alias"]
          this.concept_detail.query = response.data["query"]
          this.concept_detail.tag = response.data["tag"]
          this.concept_detail.def = response.data["def"]
          this.dialogVisible = true
        })
      }
    }

  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    /*text-align: center;*/
    color: #2c3e50;
    margin: 60px auto;
    width: 400px;
  }



</style>
