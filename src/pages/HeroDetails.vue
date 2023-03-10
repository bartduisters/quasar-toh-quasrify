<template>
  <q-form v-if="hero" class="details-container" ref="formRef" greedy>
    <div class="title">{{ hero.name }} details!</div>

    <div>id: {{ hero.number }}</div>
    <q-input
      outlined
      dense
      :label="'name'"
      v-model="hero.name"
      :rules="[required('Name')]"
      lazy-rules
    />

    <ButtonGroup class="button-group">
      <StyledButton class="back-button" @click="moveBack()">Back</StyledButton>
      <StyledButton primary @click="saveHero()">Save</StyledButton>
      <StyledButton negative @click="removeHero()">Delete</StyledButton>
    </ButtonGroup>
  </q-form>

  <div v-else class="title">Hero not found!</div>
</template>

<script lang="ts">
import { defineComponent, onBeforeMount, ref, Ref } from 'vue';
import StyledButton from 'components/StyledButton.vue';
import ButtonGroup from 'components/ButtonGroup.vue';
import { Hero } from 'components/models';
import { useRoute, useRouter } from 'vue-router';
import { useHeroes } from 'src/services/hero.service';
import { useValidators } from 'src/services/validator.composable';
import { QForm } from 'quasar';

export default defineComponent({
  components: {
    StyledButton,
    ButtonGroup,
  },
  setup() {
    const route = useRoute();
    const router = useRouter();
    const hero: Ref<Hero> = ref() as Ref<Hero>;
    const { editHero, findHero, deleteHero } = useHeroes();
    const { required } = useValidators();

    onBeforeMount(() => {
      const { id } = route.params;
      if (id) {
        const matchingHero = findHero(id.toString());
        if (matchingHero) hero.value = matchingHero;
      }
    });

    const moveBack = () => void router.go(-1);
    const saveHero = () => {
      void formRef.value.validate().then((valid) => {
        if (valid) {
          editHero(hero.value);
          moveBack();
        }
      });
    };
    const removeHero = () => {
      deleteHero(hero.value);
      moveBack();
    };

    const formRef = ref({} as QForm);

    return {
      hero,
      moveBack,
      saveHero,
      removeHero,
      required,
      formRef,
    };
  },
});
</script>

<style lang="scss" scoped>
.title {
  margin-top: 1rem;
  margin-bottom: 1rem;
}

.button-group {
  margin-top: 1rem;
}

.details-container {
  width: 20rem;
}
</style>
