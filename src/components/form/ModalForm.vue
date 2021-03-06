<!-- Form to modify a bloc -->
<template>
  <v-layout
    row
    justify-center>
    <v-dialog
      v-model="modalFormOpen"
      max-width="600">
      <v-card>
        <v-card-title>
          <span class="headline">Modification</span>
        </v-card-title>
        <v-form
          ref="formCustomBlock"
          v-model="valid"
          lazy-validation>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12>
                  <h3>Information:</h3>
                </v-flex>
                <v-flex xs12>
                  <v-text-field
                    v-model="blockCopy.name"
                    :rules="[() => !!blockCopy.name || 'Veuillez entrer un nom']"
                    label="Nom"
                    hint="Nom du bloc"
                    required
                  />
                  <v-text-field
                    v-model="blockCopy.clazz"
                    :rules="[() => !!blockCopy.clazz || 'Veuillez entrer une classe']"
                    label="Classe"
                    hint="Classe du bloc"
                    required
                  />
                  <h3>Config:</h3>
                  <div
                    v-for="(value, key) in blockCopy.fields"
                    :key="key">
                    <v-text-field
                      v-if="!blockCopy.suggestions[key].length"
                      :label="key | capitalize"
                      v-model="blockCopy.fields[key]"
                      :hint="blockCopy.details[key].description | capitalize"
                    />
                    <v-combobox
                      v-if="blockCopy.suggestions[key].length"
                      :label="key | capitalize"
                      :solo="true"
                      v-model="blockCopy.fields[key]"
                      :hint="blockCopy.details[key].description | capitalize"
                      :items="blockCopy.suggestions[key]"
                    />
                    <editor
                      v-if="blockCopy.editables[key]"
                      ref="editor"
                      v-model="blockCopy.editables[key].value"
                      :options="options"
                      :lang="blockCopy.editables[key].language"
                      theme="twilight"
                      height="150"
                    />
                  </div>
                </v-flex>
              </v-layout>
            </v-container>
            <small>*Champs obligatoires</small>
          </v-card-text>
        </v-form>
        <v-card-actions>
          <v-spacer/>
          <v-btn
            flat
            @click.native="close">Fermer</v-btn>
          <v-btn
            :disabled="!valid"
            flat
            @click.native="save">Sauvegarder</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
import { mapMutations, mapState } from 'vuex';
import { cloneDeep } from 'lodash';
import editor from 'vue2-ace-editor';

export default {
  name: 'ZModalForm',
  components: {
    editor,
  },
  filters: {
    capitalize(value) {
      if (!value) return '';
      const valueToString = value.toString();
      return valueToString.charAt(0).toUpperCase() + valueToString.slice(1);
    },
  },
  data() {
    return {
      valid: false,
      content: '',
      options: {
        enableBasicAutocompletion: true,
        highlightActiveLine: true,
      },
    };
  },
  computed: {
    blockCopy() {
      return cloneDeep(this.currentBlock);
    },
    modalFormOpen: {
      get() {
        return this.$store.state.modalFormOpen;
      },
      set() {
        this.setModalFormOpen(false);
      },
    },
    ...mapState([
      'currentBlock',
    ]),
  },
  methods: {
    close() {
      this.setModalFormOpen(false);
    },
    save() {
      if (this.$refs.formCustomBlock.validate()) {
        this.changeCurrentBlockFields(this.blockCopy);
        this.setModalFormOpen(false);
      }
    },
    ...mapMutations([
      'setModalFormOpen',
      'changeCurrentBlockFields',
    ]),
  },
};
</script>

<style scoped>
.headline {
  overflow: hidden;
  overflow-wrap: break-word;
}
.ace_editor {
  margin-top: 10px;
}
</style>
