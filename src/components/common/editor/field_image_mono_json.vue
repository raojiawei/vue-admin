<template>
    <field_image_mono
        v-model="model"
        v-bind="$attrs"
    ></field_image_mono>
</template>

<script>
import _json_object_mixin from "./_json_object_mixin"
import _mono_json_struct_object_mixin from "./_mono_json_struct_object_mixin"

export default{
    name:"field_image_mono_json",
    inheritAttrs:true,
    components:{
        field_image_mono:()=>import("./field_image_mono"),
    },
    mixins:[
        _json_object_mixin,
        _mono_json_struct_object_mixin,
    ],
}
</script>