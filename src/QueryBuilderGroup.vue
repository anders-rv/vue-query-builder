<template>
  <el-card class="vqb-group" :class="classObject">
    <div class="clearfix" slot="header">
      <div class="match-type-container" :class="{ 'form-inline': styled }">
        
        <el-form>
          <el-form-item :label="labels.matchType">
          <el-select v-model="query.logicalOperator">
            <el-option :label="labels.matchTypeAll" :value="labels.matchTypeAll"></el-option>
            <el-option :label="labels.matchTypeAny" :value="labels.matchTypeAny"></el-option>
          </el-select>
          <el-button size="large" style="float:right;padding:3px;" type="default" v-if="this.depth > 1" @click="remove" v-html="labels.removeGroup"> </el-button>
          </el-form-item>
        </el-form>
        <!--<div :class="{ 'form-group': styled }">
          <label for="vqb-match-type" class="mr-2">{{ labels.matchType }}</label>
          <select id="vqb-match-type" :class="{ 'form-control': styled }" v-model="query.logicalOperator">
            <option>{{ labels.matchTypeAll }}</option>
            <option>{{ labels.matchTypeAny }}</option>
          </select>
        </div>
        <button :class="{ 'close ml-auto': styled }" v-if="this.depth > 1" @click="remove" v-html="labels.removeGroup"></button> -->
        
      </div>
    </div>

    <div class="vqb-group-body" :class="{ 'card-body': styled }">
      <div class="rule-actions" :class="{ '': styled }">
        <div :class="{ 'form-row': styled }">
        <el-form>
          <el-row :gutter="10">
            <el-col :span="5" :sm="10" :xs="10">
              <el-select v-model="selectedRule" value-key="id" style="width:100%;">
                  <el-option 
                    v-for="(rule, index) in rules" 
                    :key="index" 
                    :label="rule.label" 
                    :value="rule">
                  </el-option>
              </el-select>
            </el-col>
            <el-col :span="10" :sm="14" :xs="14">
                <el-button type="default" v-html="labels.addRule" @click="addRule"> </el-button>
                <el-button type="default" v-if="this.depth < this.maxDepth" @click="addGroup" v-html="labels.addGroup"  :style="{marginLeft : '5px'}"> </el-button>

            </el-col>
          </el-row>
        </el-form>
          <!-- <div class="col-12 col-md-4 col-lg-4 col-sm-12 col-xs-12"> -->
            
            
          
          
          <!-- <select v-model="selectedRule" :class="{ 'form-control': styled }">
            <option v-for="(rule, index) in rules" :key="index" :value="rule">{{ rule.label }}</option>
          </select>
          </div>
          <div class="col-12  col-md-8 col-lg-8 col-sm-12 col-xs-12">
          <button @click="addRule" :class="{ 'btn btn-secondary': styled }" v-html="labels.addRule"></button>
          <button :class="{ 'btn btn-secondary': styled }" v-if="this.depth < this.maxDepth" @click="addGroup" v-html="labels.addGroup"></button>
          </div> -->
        </div>
        
      </div>
<el-form>
      <div class="children">
        <component
          v-for="(child, index) in query.children"
          :key="index"
          :is="child.type"
          :type="child.type"
          :query.sync="child.query"
          :ruleTypes="ruleTypes"
          :rules="rules"
          :rule="ruleById(child.query.rule)"
          :index="index"
          :maxDepth="maxDepth"
          :depth="depth + 1"
          :styled="styled"
          :labels="labels"
          :operatorsWithoutValue="operatorsWithoutValue"
          v-on:child-deletion-requested="removeChild">
        </component>
      </div>
</el-form>
    </div>
  </el-card>
</template>

<script>
import QueryBuilderRule from './QueryBuilderRule.vue';
import deepClone from './utilities.js';

export default {
  name: "query-builder-group",

  components: {
    QueryBuilderRule
  },

  props: ['ruleTypes', 'type', 'query', 'rules', 'index', 'maxDepth', 'depth', 'styled', 'labels','operatorsWithoutValue'],

  methods: {
    ruleById (ruleId) {
      var rule = null;

      this.rules.forEach(function(value){
        if ( value.id === ruleId ) {
          rule = value;
          return false;
        }
      });

      return rule;
    },

    addRule () {
      let updated_query = deepClone(this.query);
      let child = {
        type: 'query-builder-rule',
        query: {
          rule: this.selectedRule.id,
          selectedOperator: this.selectedRule.operators[0],
          selectedOperand: typeof this.selectedRule.operands === "undefined" ? this.selectedRule.label : this.selectedRule.operands[0],
          value: null
        }
      };
      // A bit hacky, but `v-model` on `select` requires an array.
      if (this.ruleById(child.query.rule).type === 'multi-select') {
        child.query.value = [];
      }
      updated_query.children.push(child);
      this.$emit('update:query', updated_query);
    },

    addGroup () {
      
      let updated_query = deepClone(this.query);
      if ( this.depth < this.maxDepth ) {
        updated_query.children.push({
          type: 'query-builder-group',
          query: {
            logicalOperator: "All",
            children: []
          }
        });
        this.$emit('update:query', updated_query);
      }
    },

    remove () {
      this.$emit('child-deletion-requested', this.index);
    },

    removeChild (index) {
      let updated_query = deepClone(this.query);
      updated_query.children.splice(index, 1);
      this.$emit('update:query', updated_query);
    }
  },

  data () {
    return {
      selectedRule: this.rules[0]
    }
  },

  computed: {
    classObject () {
      var classObject = {
        'box-card': this.styled,
      }

      classObject['depth-' + this.depth.toString()] = this.styled;

      return classObject;
    }
  },

  mounted:function(){
    this.selectedRule = this.rules[0];
    
  }
}
</script>
