<template>
    <meta-table
        v-if="!hasInjectComponent || componentsInjected"
        :field_list="field_list"
        :mode="mode"
        :fields="fields"
    >
        <template slot="label" slot-scope="scope">
            <labels
                :label="field_list[scope.field].label"
                :labelComponent="needInjectLabelComponents.map[scope.field]"
            >
                <component
                    v-if="needInjectLabelComponents.map[scope.field]"
                    :is="needInjectLabelComponents.map[scope.field].name"
                    :label="field_list[scope.field].label"
                    v-bind="needInjectLabelComponents.map[scope.field].config || {}"
                ></component>
            </labels>
        </template>
        <template slot-scope="scope">
            <component
                :ref="scope.field"
                :is="field_list[scope.field].editorComponent.name" 
                v-model="record[scope.field]" 
                v-bind="mergeAttrsConfig(field_list[scope.field].editorComponent.config,field_list[scope.field].editorComponent[mode + 'Config'])"
            ></component>
            <p 
                v-if="field_list[scope.field].tip" 
                class="form-helper"
            >
                {{field_list[scope.field].tip}}
            </p>
            <p 
                v-if="validators[scope.field] && validators[scope.field]['hasErr']"
                class="text-danger form-helper"
            >
                {{validators[scope.field]['errMsg']}}
            </p>
        </template>
    </meta-table>
</template>

<script>
import labels from "@/components/common/labels/labels"

import {observe_relates} from "./field_relates_helper.js"

import mergeAttrsConfig from "@/mixins/common/mergeAttrsConfig.js"

import injectComponents from "@/widget/injectComponents"

import AsyncValidator from 'async-validator';

import filterLabelComponents from "@/injectHelper/labelComponentHelper"


export default{
    mixins:[
        mergeAttrsConfig,
    ],
    data(){
        return {
            labelComponentsInjected:false,
            editorComponentsInjected:false,
            validators:{},
            recordUnwatchs:[],
            hasValidateListener:false,
        }
    },
    components:{
        labels,
        field_array_model:()=>import("./field_array_model"),
        field_array_model_json:()=>import("./field_array_model_json"),
        field_async_array_model:()=>import("./field_async_array_model"),
        field_async_array_model_json:()=>import("./field_async_array_model_json"),
        field_async_enum_radio:()=>import("./field_async_enum_radio"),
        field_async_enum_select:()=>import("./field_async_enum_select"),
        field_async_model:()=>import("./field_async_model"),
        field_async_tag:()=>import("./field_async_tag"),
        field_async_tag_json:()=>import("./field_async_tag_json"),
        field_bool:()=>import("./field_bool"),
        field_day:()=>import("./field_day"),
        field_enum_radio:()=>import("./field_enum_radio"),
        field_enum_select:()=>import("./field_enum_select"),
        field_file_mono:()=>import("./field_file_mono"),
        field_file_mono_json:()=>import("./field_file_mono_json"),
        field_file_multi:()=>import("./field_file_multi"),
        field_file_multi_json:()=>import("./field_file_multi_json"),
        field_image_mono:()=>import("./field_image_mono"),
        field_image_mono_json:()=>import("./field_image_mono_json"),
        field_image_multi:()=>import("./field_image_multi"),
        field_image_multi_json:()=>import("./field_image_multi_json"),
        field_int:()=>import("./field_int"),
        field_model:()=>import("./field_model"),
        field_month:()=>import("./field_month"),
        field_number:()=>import("./field_number"),
        field_pwd:()=>import("./field_pwd"),
        field_relates_array_model:()=>import("./field_relates_array_model"),
        field_relates_array_model_json:()=>import("./field_relates_array_model_json"),
        field_relates_enum_radio:()=>import("./field_relates_enum_radio"),
        field_relates_enum_select:()=>import("./field_relates_enum_select"),
        field_relates_model:()=>import("./field_relates_model"),
        field_relates_tag:()=>import("./field_relates_tag"),
        field_relates_tag_json:()=>import("./field_relates_tag_json"),
        field_sex:()=>import("./field_sex"),
        field_string:()=>import("./field_string"),
        field_tag:()=>import("./field_tag"),
        field_tag_json:()=>import("./field_tag_json"),
        field_text:()=>import("./field_text"),
        field_text_rich:()=>import("./field_text_rich"),
        field_ts:()=>import("./field_ts"),
        field_year:()=>import("./field_year"),
        metaTable:()=>import("@/components/common/meta-table"),
    },
    computed:{
        editFieldsArray(){
            return this.fields.reduce((arr,row)=>{

                row.reduce((arr,field)=>{
                    arr.push(field);
                    return arr;
                },arr);

                return arr;
            },[])
        },
        formData(){
            return this.editFieldsArray.reduce((obj,field)=>{
                obj[field] = this.record[field];
                return obj;
            },{});
        },
        needInjectLabelComponents(){
            return filterLabelComponents(this.field_list,this.editFieldsArray,this.mode);
        },
        needInjectEditorComponents(){
            return this.editFieldsArray.filter((field)=>{
                return this.field_list[field].editorComponent && this.field_list[field].editorComponent.component;
            }).map((field)=>{
                return this.field_list[field].editorComponent;
            });
        },
        hasInjectComponent(){
            return this.needInjectLabelComponents.list.length || this.needInjectEditorComponents.length;
        },
        componentsInjected(){
            return this.labelComponentsInjected && this.editorComponentsInjected;
        },
    },
    methods:{
        injectLabelComponents(){
            if(!this.needInjectLabelComponents.list.length){
                return this.labelComponentsInjected = true;
            }

            injectComponents(this,this.needInjectLabelComponents.list).then(()=>{
                this.labelComponentsInjected = true;
            });
        },
        injectEditorComponents(){
            if(!this.needInjectEditorComponents.length){
                return this.editorComponentsInjected = true;
            }

            injectComponents(this,this.needInjectEditorComponents).then(()=>{
                this.editorComponentsInjected = true;
            });
        },
        validate(){
            let keys = Object.keys(this.validators);

            let promises = keys.map((field)=>{
                return this.validateField(field,this.formData[field]);
            });

            if(!this.hasValidateListener){
                keys.forEach((field)=>{
                    this.validators[field].unwatch = this.addValidateInputListener(field);
                })
                this.hasValidateListener = true;
            }
            
            return Promise.all(promises).then(()=>{
                return JSON.parse(JSON.stringify(this.formData));
            }).catch((err)=>{
                return Promise.reject(err);
            })
        },
        validateField(field,value){
            return new Promise((resolve,reject)=>{
                let asyncValidator = this.validators[field]['validator'];
                asyncValidator.validate({[field]:value},(errors,fields)=>{
                    if(errors){
                        this.validators[field]['hasErr'] = true;
                        this.validators[field]['errMsg'] = errors[0]['message'];
                        reject(errors[0]['message']);
                    }else{
                        this.validators[field]['hasErr'] = false;
                        this.validators[field]['errMsg'] = '';
                        resolve();
                    }
                })
            })

        },
        addValidateInputListener(field){
            return this.$watch(()=>{
                return this.record[field]
            },(value)=>{
                this.validateField.call(this,field,value).catch(()=>{})
            })
        },
        resetRelates(){
            this.recordUnwatchs.forEach((unwatch)=>{
                unwatch && unwatch();
            });
            this.recordUnwatchs = [];

            this.editFieldsArray.forEach((field)=>{
                const editorComponent = this.field_list[field].editorComponent;
                if(!editorComponent || !editorComponent.config || !editorComponent.config.relates){
                    return;
                }
                const relates = this.field_list[field].editorComponent.config.relates;
                observe_relates(relates,this.record);

                relates.forEach((relateItem)=>{
                    if(typeof relateItem.handler !== 'function'){
                        return;
                    }

                    let callback = function(newVal,oldVal){
                        if(this.$refs[field]){
                            relateItem.handler.call(this.$refs[field],newVal,this.field_list[field],oldVal);
                        }else{
                            setTimeout(()=>{
                                callback.call(this,newVal,oldVal)
                            },0)
                        }
                    }

                    const unwatch = this.$watch(()=>{
                        if(Array.isArray(relateItem.relateField)){
                            return relateItem.relateField.reduce((obj,field)=>{
                                obj[field] = this.record[field];
                                return obj;
                            },{});
                        }else{
                            return this.record[relateItem.relateField]
                        }
                    },callback,relateItem.config);

                    this.recordUnwatchs.push(unwatch);

                });

            });

        },
        initValidate(){
            this.fields.forEach((row)=>{
                row.forEach((field)=>{
                    if(this.field_list[field].validator){
                        let asyncValidator = new AsyncValidator({[field]:this.field_list[field].validator});

                        this.$set(this.validators,field,{
                            hasErr:false,
                            errMsg:'',
                            validator:asyncValidator,
                            unwatch:null,
                        })

                        if(this.autoValidate){
                            this.validators[field].unwatch = this.addValidateInputListener(field);
                        }
                    }
                })
            });

            if(this.autoValidate){
                this.hasValidateListener = true;
            }
        }
    },
    watch:{
        record:{
            immediate:true,
            handler(){
                Object.keys(this.validators).forEach((field)=>{
                    this.validators[field].unwatch && this.validators[field].unwatch();
                });

                this.validators = {};
                this.hasValidateListener = false;
                this.initValidate();
            }
        },
        fields:{
            immediate:true,
            handler(){
                this.labelComponentsInjected = false;
                this.editorComponentsInjected = false;
                this.injectLabelComponents();
                this.injectEditorComponents();
            },
        },
    },
    props:{
        fields:{
            type:Array,
            required:true
        },
        record:{
            type:Object,
            required:true,
        },
        field_list:{
            type:Object,
            required:true,
        },
        autoValidate:{
            type:Boolean,
            default:false,
        },
        mode:{
            type:String,
            default:"create"
        }
    },
    created(){
        this.$watch(()=>{
            return {
                record:this.record,
                fields:this.fields,
            };
        },this.resetRelates,{
            immediate:true,
        })
    },


}
</script>

<style scoped>
.form-helper{
    margin-top:5px;
    margin-bottom:5px;
    color:#737373;
    font-size:12px;
    line-height:1.42;
}
.text-danger{
    color:#FF4949;
}
</style>