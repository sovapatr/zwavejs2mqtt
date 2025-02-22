<template>
  <v-dialog v-model="value" max-width="500px" persistent>
    <v-card>
      <v-card-title>
        <span class="headline">New Association</span>
      </v-card-title>

      <v-card-text>
        <v-container grid-list-md>
          <v-form v-model="valid" ref="form" lazy-validation>
            <v-row>
              <v-col cols="12">
                <v-select
                  label="Node Endpoint"
                  hint="Used to filter available groups"
                  v-model="group.endpoint"
                  persistent-hint
                  :items="endpoints"
                ></v-select>
              </v-col>

              <v-col cols="12">
                <v-select
                  label="Group"
                  hint="Node/Endpoint Group association to Add/Remove"
                  persistent-hint
                  v-model="group.group"
                  :items="endpointGroups"
                  :rules="[required]"
                  return-object
                >
                  <template v-slot:selection="{ item }">
                    {{ item.text }}
                  </template>
                  <template v-slot:item="{ item, attrs, on }">
                    <v-list-item v-on="on" v-bind="attrs" two-line>
                      <v-list-item-content>
                        <v-list-item-title>{{ item.text }}</v-list-item-title>
                        <v-list-item-subtitle>{{
                          item.endpoint >= 0 ? 'Endpoint ' + item.endpoint : ''
                        }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                  </template>
                </v-select>
              </v-col>

              <v-col cols="12">
                <v-combobox
                  label="Target Node"
                  v-model="group.target"
                  :items="filteredNodes"
                  return-object
                  :rules="[required]"
                  hint="Node to add to the association group"
                  persistent-hint
                  item-text="_name"
                ></v-combobox>
              </v-col>

              <v-col v-if="group.group && group.group.multiChannel" cols="12">
                <v-select
                  v-model.number="group.targetEndpoint"
                  persistent-hint
                  label="Target Endpoint"
                  hint="Target node endpoint"
                  :items="targetEndpoints"
                ></v-select>
              </v-col>
            </v-row>
          </v-form>
        </v-container>
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" text @click="$emit('close')">Cancel</v-btn>
        <v-btn
          color="blue darken-1"
          text
          @click="$refs.form.validate() && $emit('add', group)"
          >ADD</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: {
    value: Boolean,
    nodes: Array,
    node: Object
  },
  watch: {
    value () {
      this.$refs.form && this.$refs.form.resetValidation()
      this.resetGroup()
    }
  },
  computed: {
    filteredNodes () {
      return this.nodes.filter(n => n.id !== this.node.id)
    },
    endpoints () {
      return this.getEndpointItems(this.node)
    },
    targetEndpoints () {
      return this.getEndpointItems(this.group.target)
    },
    endpointGroups () {
      let groups = []
      try {
        groups = this.node.groups
        const endpoint = this.group.endpoint
        groups = groups.filter(
          g =>
            g.endpoint === endpoint ||
            (endpoint === null && g.endpoint === 0 && !g.multiChannel)
        )
      } catch (error) {}

      return groups
    }
  },
  data () {
    return {
      valid: true,
      group: {},
      defaultGroup: { endpoint: null },
      required: v => !!v || 'This field is required'
    }
  },
  methods: {
    resetGroup () {
      this.group = Object.assign({}, this.defaultGroup)
    },
    getEndpointItems (node) {
      const endpoints = [
        { text: 'No endpoint', value: null }, // cannot use undefined here or it will return the value
        { text: 'Endpoint 0', value: 0 }
      ]

      if (node && node.endpointIndizes) {
        for (const i of node.endpointIndizes) {
          endpoints.push({ text: 'Endpoint ' + i, value: i })
        }
      }

      return endpoints
    }
  }
}
</script>
