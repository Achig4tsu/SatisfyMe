<template>
  <div>
    <div>
      <h1>Jeu des Cercles</h1>
      <p>Faites passer la balle à travers les cercles en rotation pour gagner !</p>
    </div>

    <div>
      <NuxtLink to="/play">
        <Button>Commencer</Button>
      </NuxtLink>

      <Button @click="drawerOpen = true">Paramètres</Button>
    </div>

    <Drawer v-model:open="drawerOpen">
      <DrawerContent>
        <DrawerHeader>
          <DrawerTitle>Paramètres du jeu</DrawerTitle>
          <DrawerDescription>Configurez les options du jeu</DrawerDescription>
        </DrawerHeader>
        <div>
          <div>
            <!-- Placeholder for settings -->
          </div>
        </div>
      </DrawerContent>
    </Drawer>
  </div>
</template>

<script setup>
import { ref } from 'vue';
const drawerOpen = ref(false);
</script>

<style scoped>
/* Removed all styles */
</style>
