<template>
    <div class="config-editor">
        <component v-for="field in fields" :key="field.paramName" :is="componentFromType(field.type)" :schema="field" @update="updateModel"></component>
    </div>
</template>

<script>
    import InputString from './ConfigFields/InputString.vue';
    import InputBoolean from './ConfigFields/InputBoolean.vue';
    import InputNumber from './ConfigFields/InputNumber.vue';
    import InputFlag from './ConfigFields/InputFlag.vue';
    import InputSet from './ConfigFields/InputSet.vue';
    import InputEnum from './ConfigFields/InputEnum.vue';
    import InputDictionary from './ConfigFields/InputDictionary.vue';

    export default {
      name: 'config-editor',
      props: {
        fields: {
          type: Array,
          required: true
        },
        model: {
          type: Object,
          required: true
        }
      },
      methods: {
        componentFromType(type) {
          switch(type) {
            case 'string':
            case 'bigNumber':
              return InputString;
            case 'boolean':
              return InputBoolean;
            case 'number':
            case 'smallNumber':
              return InputNumber;
            case 'flag':
              return InputFlag;
            case 'enum':
              return InputEnum;
            case 'hashSet':
              return InputSet;
            case 'dictionary':
              return InputDictionary;
          }
        },
        updateModel(value, field) {
          const fieldSchema = this.fields.find(fieldSchema => fieldSchema.paramName === field);

          if (fieldSchema && typeof fieldSchema.defaultValue !== 'undefined' && this.isEqual(value, fieldSchema.defaultValue)) {
            delete this.model[field];
          } else {
            this.model[field] = value;
          }
        },
        isEqual(a, b) {
          if (typeof a !== typeof b) return false;

          switch (typeof a) {
            case 'number':
            case 'string':
              return a === b;
            case 'object':
              if (Array.isArray(a) && Array.isArray(b)) {
                return a.length === b.length && a.every((item, index) => item === b[index]);
              }

              if ('' + a === '[object Object]' && '' + b === '[object Object]') {
                return Object.keys(a).length === Object.keys(b).length && Object.keys(a).every(key => a[key] === b[key]);
              }

              return a === b;
          }

          return false;
        }
      }
    }
</script>

<style lang="scss">
    .config-editor {

    }
</style>