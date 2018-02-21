<template>
  <el-card class="vqb-rule" :body-style="{ padding:'0px'}" :class="{ 'box-card ': styled }">
    
      <el-form-item  :label="rule.label"   style="margin-bottom:'0px'">
        <el-row :gutter="10">
          <el-col :span="4">
            <el-select v-if="typeof rule.operands !== 'undefined'" v-model="query.selectedOperand">
              <el-option v-for="operand in rule.operands" :key="operand" :label="operand" :value="operand"></el-option>
            </el-select>
            <el-select v-on:change="operatorChange" v-if="! isMultipleChoice" v-model="query.selectedOperator">
              <el-option v-for="operator in rule.operators" :key="operator" :label="operator" :value="operator"></el-option>
            </el-select>
          </el-col>
          <el-col :span="8">
            <el-input  v-if="rule.inputType === 'text'" type="text" v-model="query.value" :placeholder="labels.textInputPlaceholder"></el-input>
            <el-input-number v-if="rule.inputType === 'number'" type="text" v-model="query.value" :placeholder="labels.textInputPlaceholder"></el-input-number>
            <template v-if="isCustomComponent">
              <component :value="query.value" @input="updateQuery" :is="rule.component"></component>
            </template>
            <el-checkbox-group v-if="rule.inputType === 'checkbox'" v-model="query.value" >
                <el-checkbox style="display:inline" v-for="choice in rule.choices" :key="choice.value" :label="choice.value"></el-checkbox>
            </el-checkbox-group>  
            <el-radio-group v-if="rule.inputType === 'radio'" v-model="query.value" >
                <el-radio style="display:inline" v-for="choice in rule.choices" :key="choice.value" :label="choice.value"></el-radio>
            </el-radio-group>  
            <el-select v-if="rule.inputType === 'select'" value-key="value" 
              :multiple="rule.type == 'multi-select'" 
              v-model="query.value"
              style="width:100%;">
              <el-option
              v-for="(choice,index) in rule.choices"
              :value="choice.value"
              :key="index"
              :label="choice.label">
              </el-option>
            </el-select>
            <el-date-picker  v-if="rule.inputType === 'date'"
              v-model="query.value"
              value-format="dd/MM/yyyy"
              @input="dateUpdate"
              :format="rule.format !== 'undefined' ? rule.format : 'dd/MM/yyyy'"
              :type="query.selectedOperator !=='range' ? 'date' : 'daterange'"
              :placeholder="query.selectedOperator !=='range' ? 'Pick a date' : ''"
              range-separator="To"
              start-placeholde="Start date"
              end-placeholde="End date"
              >
              
            </el-date-picker>
          </el-col>
          <el-button size="large" style="float:right;padding:3px;" type="info" @click="remove" v-html="labels.removeRule"></el-button>
        </el-row>
        
          
          
          
      </el-form-item>

     
      <!-- <label>{{ rule.label }}</label>
      <div>
      <select v-if="typeof rule.operands !== 'undefined'" v-model="query.selectedOperand" :class="{ 'form-control': styled }">
        <option v-for="operand in rule.operands">{{ operand }}</option>
      </select>
      </div>
      <div>
        <select v-if="! isMultipleChoice" v-model="query.selectedOperator" :class="{ 'form-control': styled }">
          <option v-for="operator in rule.operators" v-bind:value="operator">
            {{ operator }}
          </option>
        </select>
      </div>
      <div class="ml-1">
        <input :class="{ 'form-control': styled }" v-if="rule.inputType === 'text'" type="text" v-model="query.value" :placeholder="labels.textInputPlaceholder"></input>
        <input :class="{ 'form-control': styled }" v-if="rule.inputType === 'number'" type="number" v-model="query.value"></input>

        <template v-if="isCustomComponent">
          <component :value="query.value" @input="updateQuery" :is="rule.component"></component>
        </template>

        <div class="checkbox" v-if="rule.inputType === 'checkbox'">
          <label v-for="choice in rule.choices" :class="{ 'float-left': styled }">
            <input type="checkbox" :value="choice.value" v-model="query.value"> {{ choice.label }}
          </label>
        </div>

        <div class="radio" v-if="rule.inputType === 'radio'">
          <label v-for="choice in rule.choices" :class="{ 'float-left': styled }">
            <input type="radio" :value="choice.value" v-model="query.value"> {{ choice.label }}
          </label>
        </div>

        <div v-if="rule.inputType === 'select2'" class="bg-white">
          <v-select multiple @input="updateVSelectValue" v-model="rule.selected"  :options="rule.choices"></v-select>        
        </div>
        <div v-if="rule.inputType === 'date'" class="bg-white">
          <input
              type="text"
              ref="node"
              class="form-control date-picker"
              v-model="query.value" 
              >
        </div>

        <select
          v-if="rule.inputType === 'select'"
          :class="{ 'form-control': styled }"
          :multiple="rule.type === 'multi-select'"
          v-model="query.value">
          <option v-for="choice in rule.choices" :value="choice.value">{{ choice.label }}</option>
        </select>
      </div>
      <span class="ml-auto"><button :class="{ 'close float-right': styled }" @click="remove" v-html="labels.removeRule"></button></span> -->
    
  </el-card>
</template>

<script>
import deepClone from './utilities.js';

export default {
  name: "query-builder-rule",

  props: ['query', 'index', 'rule', 'styled', 'labels'],

  beforeMount () {
    if (this.rule.type === 'custom-component' || this.rule.type =='select2') {
      this.$options.components[this.id] = this.rule.component;
    }
  },

  methods: {
    remove: function() {
      this.$emit('child-deletion-requested', this.index);
    },
    updateQuery(value) {
      let updated_query = deepClone(this.query);
      updated_query.value = value;
      this.$emit('update:query', updated_query);
    },
    updateVSelectValue: function(val){
         this.$emit('input', val);
         this.query.value = val.map((item) => {return item }).join(",");
         
    },
    dateUpdate:function(val){
      //console.log(val);
      this.query.value = val;
    },
    operatorChange:function(){
      if(this.rule.type && this.query.selectedOperator !== 'range')
      {
          this.query.value = '';
      }
    }
  },

  computed: {
    isMultipleChoice () {
      return ['radio', 'checkbox', 'select'].indexOf(this.rule.inputType) >= 0;
    },

    isCustomComponent () {
      return this.rule.type === 'custom-component';
    }
  },

  mounted () {
    let updated_query = deepClone(this.query);
    if (this.rule.inputType === 'checkbox') {
      updated_query.value = [];
      this.$emit('update:query', updated_query);
    }
    if (this.rule.type === 'select') {
      updated_query.value = this.rule.choices[0].value;
      this.$emit('update:query', updated_query);
    }
    if (this.rule.type === 'custom-component') {
      updated_query.value = this.rule.default || null;
      this.$emit('update:query', updated_query);
    }
/*
    if (this.rule.type === 'select2') {
      updated_query.value = this.rule.default || null;
      this.$emit('update:query', updated_query);
    }
     if (this.rule.type === 'date') {
      updated_query.value = this.rule.default || null;
      this.$emit('update:query', updated_query);
      this.htmlNode = this.$refs.node
      let vm = this
      this.picker = new Pikaday({
      field: this.htmlNode,
      format: vm.rule.format!=null && vm.rule.format!= 'undefined' ? vm.rule.format: "DD/MM/YYYY",
      showMonthAfterYear: this.showMonthAfterYear,
      onSelect () {
        
        vm.currentDate = this.getMoment().toDate()
        vm.$emit('onSelect', vm.currentDate);
        vm.query.value = vm.picker.toString();
      }
    });
    }*/
  }
}
</script>
