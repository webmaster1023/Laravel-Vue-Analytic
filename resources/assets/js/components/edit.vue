<template>
    <div class="row">
        <div class="col-md-12">
            <div class="panel panel-default">
                <div class="panel-heading"><h2>Edit Experiment</h2></div>
                <div class="panel-body">
                    <div class="row">
                        <div class="col-md-8">
                            <input type="text" id="experiment_id" placeholder="Experiment name" v-model="exp_name" name="exp_name" value="">
                        </div>
                    </div>
                    <div class="row rules">
                        <div class="col-md-12">
                            <label>Rules</label>
                        </div>
                        <div class="col-md-12">
                            <button v-if="has_rule == 0" v-on:click="has_rule = 1" type="button" class="btn btn-sm btn-primary">Add Rule</button>
                        </div>
                        <div v-if="has_rule == 1">
                            <div class="col-md-3">
                                <v-select v-model="custom_rules.variable" label="value" :searchable="false" :options="rule_variable_type" ></v-select>
                            </div>
                            <div class="col-md-3">
                                <v-select v-model="custom_rules.operator" label="value"  :searchable="false"  :options="rule_operator_type"></v-select>
                            </div>
                            <div class="col-md-3">
                                <input type="text" v-model="custom_rules.value" required placeholder="Enter value">
                            </div>
                            <div class="col-md-3">
                                <button v-on:click="has_rule = 0" type="button">-</button>
                            </div>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-12">
                            <label>Options</label>
                        </div>
                    </div>
                    <ul>
                        <li v-for="(option, index) in custom_options" class="row option_item">
                            <div class="col-md-3">
                                <input type="text" v-model="option.css_selector" placeholder="Enter CSS Selector Value"  value="" required>
                            </div>
                            <div class="col-md-3">
                                <v-select v-model="option.type" label="value"  :searchable="false" :options="option_type"></v-select>
                            </div>
                            <div class="col-md-3">
                                <div v-if="option.type.label === 'image'">
                                    <vue-core-image-upload
                                      :class="['btn', 'btn-primary', 'btn-sm']"
                                      @imageuploaded="imageuploaded"
                                      :max-file-size="5242880"
                                      url="/rule/fileupload"
                                      :data="{id:index}"
                                      >
                                    </vue-core-image-upload>
                                    <br />
                                    <div v-if="option.value" id="img-filename"><img v-bind:src="'../../images/'+option.value" height="50"></div>
                                </div>
                                <input v-if="option.type.label === 'text'" type="text" v-model="option.value" placeholder="Enter Value" required value="">
                            </div>
                            <div class="col-md-3">
                                <button v-on:click="removeOption(index)" type="button">-</button>
                            </div>
                        </li>
                    </ul>

                    <div class="row">
                        <div class="col-md-9">
                            <button v-on:click="addOption" class="pull-right" type="button">New Options</button>
                        </div>
                    </div>
                    <div class="" style="margin-top:20px">
                        <!-- Indicates a successful or positive action -->
                        <input type="hidden" v-model="param" name="custom_options" value="">
                        <button v-on:click="storeRules" type="submit" class="btn btn-primary">Save</button>
                        <a href="/rule"><button type="button" class="btn btn-default">Back</button></a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import VueCoreImageUpload from 'vue-core-image-upload'
    import vSelect from "vue-select"

    export default {
        props: ['experimentData'],
        components:{
            'vue-core-image-upload': VueCoreImageUpload,
            'v-select': vSelect
        },
        data() {
            return {
                exp_id: null,
                has_rule: 0,
                exp_name : '',
                rule_variable_type :[{
                    label:'pagepath',
                    value :'Page Path'
                }, {
                    label:'hostname',
                    value :'Hostname'
                }],
                rule_operator_type :[{
                    label:'contain',
                    value :'Contains'
                }, {
                    label:'equalto',
                    value :'Equals To'
                }, {
                    label:'not_contain',
                    value :'Does Not Contain'
                }, {
                    label:'not_equalto',
                    value :'Not Equals To'
                }],
                option_type :[{
                    label:'text',
                    value :'Text'
                }, {
                    label:'image',
                    value :'Image'
                }],
                custom_rules : {},
                custom_options: [],
                param: '',
                src: ''
            }
        },
        mounted() {
            this.getVariables()
        },
        methods: {
            addOption: function(event) {
                this.custom_options.push({
                    css_selector: '',
                    type: this.option_type[0],
                    value: '',
                    org_filename: ''
                })
            },
            removeOption: function(id) {
                if (this.custom_options.length == 1) {
                    alert("At least one option required.")
                    return;
                }
                if (confirm("Do you really want to remove this option?")) {
                    this.custom_options.splice(id, 1)
                }
            },
            getVariables: function() {
                let exp_arr = this.experimentData;
                let that  = this;
                this.exp_id = exp_arr.id;
                this.exp_name = exp_arr.name;
                this.custom_rule_indexes = {}
                if (typeof exp_arr.rules.value == 'undefined' || exp_arr.rules.value == '') {
                    this.has_rule = 0;
                    this.custom_rules = {
                        variable: this.rule_variable_type[0],
                        operator: this.rule_operator_type[0],
                        value: ''
                    }
                } else {
                    this.has_rule = 1;
                    this.custom_rule_indexes['variable'] =  this.rule_variable_type.findIndex(x => x.label== exp_arr.rules.variable);
                    this.custom_rule_indexes['operator'] =  this.rule_operator_type.findIndex(x => x.label== exp_arr.rules.operator);
                    this.custom_rules = {
                        variable: this.rule_variable_type[this.custom_rule_indexes['variable']],
                        operator: this.rule_operator_type[this.custom_rule_indexes['operator']],
                        value: exp_arr.rules.value
                    }
                }

                exp_arr.options.map(function(item) {
                    that.custom_options.push({
                        css_selector    : item.css_selector,
                        type            : that.option_type[that.option_type.findIndex(x => x.label== item.type)],
                        value           : item.value,
                        org_filename    : item.org_filename
                    })
                })
            },
            imageuploaded: function(res) {
              if (res.errors == false) {
                  let data = res.data;
                  this.custom_options[data.id].value = data.filename
                  this.custom_options[data.id].org_filename = data.org_filename
              }
            },
            storeRules: function(e){
                let rule_param = {}
                if (this.has_rule) {
                    rule_param = {
                        variable    : this.custom_rules.variable['label'],
                        operator    : this.custom_rules.operator['label'],
                        value       : this.custom_rules.value
                    }
                }

                let option_param = this.custom_options.map((item) => {
                    let tmp_result = {
                        css_selector    : item.css_selector,
                        type            : item.type['label'],
                        value           : item.value
                    }

                    if (item.type['label'] == 'image')
                        tmp_result['org_filename'] = item.org_filename
                    return tmp_result
                })

                for (var i = 0; i < this.custom_options.length; i++) {
                    if (this.custom_options[i].type['label'] == 'image' &&
                        (this.custom_options[i].org_filename == '' || typeof this.custom_options[i].org_filename == 'undefined')) {
                        alert('Please upload image.');
                        e.preventDefault();
                    }
                }

                this.param = JSON.stringify({
                    exp_id: this.exp_id,
                    rules: JSON.stringify(rule_param),
                    options: JSON.stringify(option_param)
                })
            }
        }
    }
</script>

<style lang="less" scoped>
ul {
    li {
        list-style: none;
        margin-bottom: 10px
    }
    .option_item{
        input[type='file']{
            display: none;
        }
    }
    margin:0px;
    padding:0px;
}
#img-filename {
    margin-top: 10px;
}
</style>
