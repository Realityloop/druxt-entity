<template>
  <component
    :is="component"
    v-if="entity && schema"
    v-bind="props"
  >
    <druxt-field
      v-for="(field, key) of fields"
      :key="key"
      v-bind="field"
    />
  </component>
</template>

<script>
import { DruxtRouterEntityMixin } from 'druxt-router'
import { DruxtSchemaMixin } from 'druxt-schema'
import { mapActions } from 'vuex'

export default {
  name: 'DruxtEntity',

  mixins: [DruxtRouterEntityMixin, DruxtSchemaMixin],

  computed: {
    component() {
      for (const suggestion of this.suggestions) {
        if (typeof this.$options.components[suggestion] !== 'undefined') {
          return suggestion
        }
      }

      return 'div'
    },

    suggestions() {
      const suggestions = []
      if (!this.schema) return suggestions

      const prefix = 'DruxtEntity'

      const type = this.schema.resourceType.split('--').map(part => part.charAt(0).toUpperCase() + part.slice(1)).join('')
      suggestions.push(prefix + type)

      return suggestions
    },

    fields() {
      if (!this.entity) return false

      const data = {
        ...this.entity.attributes,
        ...this.entity.relationships
      }

      const fields = {}
      for (const field of this.schema.fields) {
        // Filter out empty fields.
        if (typeof data[field.id] === 'undefined' || !data[field.id] || (data[field.id].data && !data[field.id].data.length)) continue

        fields[field.id] = {
          data: data[field.id],
          schema: field,
          relationship: !!this.entity.relationships[field.id]
        }
      }

      return fields
    },

    props() {
      if (this.component === 'div') return false

      return {
        entity: this.entity,
        fields: this.fields,
        schema: this.schema
      }
    }
  }
}
</script>