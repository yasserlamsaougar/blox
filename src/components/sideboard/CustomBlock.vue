<!-- Composant formulaire de creation de nouveau bloc -->
<template>
  <v-layout
    row
    justify-center>
    <v-dialog
      v-model="showDialog"
      persistent
      max-width="500px">
      <v-btn slot="activator">Nouveau Bloc</v-btn>
      <v-card>
        <v-card-title>
          <span class="headline">Création d'un nouveau bloc</span>
        </v-card-title>
        <v-form
          ref="formCustomBlock"
          v-model="valid"
          lazy-validation>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex
                  xs12
                  sm8>
                  <v-text-field
                    v-model="blockName"
                    :rules="[() => !!blockName || 'Veuillez entrer un nom']"
                    label="Nom"
                    hint="Nom du bloc"
                    required
                  />
                </v-flex>
                <v-flex
                  xs12
                  sm4>
                  <v-autocomplete
                    v-model="blockType"
                    :rules="[() => !!blockType || 'Veuillez choisir un type']"
                    :items="['input', 'middleware', 'output']"
                    label="Type"
                    hint="Spéficiez le type de bloc"
                    attach
                    required
                  />
                </v-flex>
                <v-flex xs12>
                  <v-text-field
                    v-model="blockClazz"
                    :rules="[() => !!blockClazz || 'Veuillez choisir une Class Java']"
                    label="Class"
                    hint="Class Java qui définie le bloc"
                    required
                  />
                </v-flex>
                <v-flex xs12>
                  <h3>Config:</h3>
                </v-flex>
                <v-layout
                  v-for="(param, index) in blockConfig"
                  :key="param.id">
                  <v-flex xs4>
                    <v-text-field
                      v-model="param.name"
                      :rules="[() => !!param.name || 'Veuillez entrer un nom de paramètre']"
                      label="Paramètre"
                      hide-details
                    />
                  </v-flex>
                  <v-flex xs7>
                    <v-text-field
                      v-model="param.value"
                      :rules="[() => !!param.value || 'Veuillez entrer une valeur']"
                      label="Valeur"
                      hide-details
                    />
                  </v-flex>
                  <v-tooltip
                    xs1
                    bottom>
                    <v-btn
                      slot="activator"
                      icon
                      @click="blockConfig.splice(index, 1)">
                      <v-icon>clear</v-icon>
                    </v-btn>
                    <span>Supprimer</span>
                  </v-tooltip>
                </v-layout>
                <v-btn
                  class="BtnNoTextTransform"
                  flat
                  small
                  right
                  @click="addNewParamsInConfig">Ajouter un paramètre...</v-btn>
              </v-layout>
            </v-container>
            <small>*Champs obligatoires</small>
          </v-card-text>
        </v-form>
        <v-card-actions>
          <v-spacer/>
          <v-btn
            flat
            @click.native="clearCustomBlockForm">Fermer</v-btn>
          <v-btn
            :disabled="!valid"
            flat
            @click.native="createBlock">Créer</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
import { mapMutations } from 'vuex';
import { forEach, cloneDeep } from 'lodash';
import Block from '../../models/block';

export default {
  name: 'ZCustomBlock',
  inject: ['uuidService', 'jsPlumbService'],
  data() {
    return {
      showDialog: false,
      valid: false,
      blockName: '',
      blockType: '',
      blockClazz: '',
      blockConfig: [],
      nextParamsId: 0,
    };
  },
  methods: {
    addNewParamsInConfig() {
      this.blockConfig.push({
        id: this.nextParamsId += 1,
        name: '',
        value: '',
      });
    },
    createBlock() {
      if (this.$refs.formCustomBlock.validate()) {
        const config = {};
        forEach(this.blockConfig, (e) => {
          config[e.name] = e.value;
        });

        const block = new Block(
          this.uuidService.uuid(),
          this.blockName,
          this.blockType,
          this.blockClazz,
          cloneDeep(config),
        );
        const blockPositionX = window.scrollX + (window.innerWidth / 2);
        const blockPositionY = window.scrollY + (window.innerHeight / 2.3);
        block.setPosition(blockPositionX, blockPositionY);
        this.addBlockToWorkflow(block);
        this.clearCustomBlockForm();
      }
    },
    clearCustomBlockForm() {
      this.$refs.formCustomBlock.reset();
      this.blockConfig = [];
      this.showDialog = false;
    },
    ...mapMutations([
      'addBlockToWorkflow',
    ]),
  },
};
</script>

<style scoped>
.BtnNoTextTransform {
  text-transform: none;
}
</style>
