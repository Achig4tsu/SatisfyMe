<script setup lang="ts">
import {
  Sidebar,
  SidebarContent,
  SidebarHeader,
  SidebarFooter,
  SidebarGroup,
  SidebarGroupLabel,
  SidebarGroupContent
} from '@/components/ui/sidebar'
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem } from '@/components/ui/select'
import { ref } from 'vue'

const props = defineProps({
  params: {
    type: Object,
    default: () => ({
      circleCount: 2,
      rotationMode: 'same',
      circleColor: '#000000',
      ballColor: '#ffffff',
      initGap: 10,
      ballRadius: 6,
      circleWidth: 4,
      rotateSpeed: 0.001,
      gravity: 0.05,
      gravityPower: 0.1
    })
  },
  applyParams: {
    type: Function,
    default: () => {}
  }
})

const emits = defineEmits(['update:devMode', 'update:showDebug'])
const devMode = defineModel('devMode', { type: Boolean, default: false })
const showDebug = defineModel('showDebug', { type: Boolean, default: false })

// Section active
const activeSection = ref('general')

// Helper function to handle input events
function handleInput(event: Event, callback: Function) {
  const target = event.target as HTMLInputElement;
  if (target) {
    callback(target.value);
  }
}

// Helper function to handle click events
function handleClick(event: MouseEvent, callback: Function) {
  callback();
}

// Wrapper function for Select's update:modelValue
function handleSelectUpdate(value: any) {
  props.applyParams(value);
}
</script>

<template>
  <Sidebar>
    <SidebarHeader>
      <h1>Paramètres du jeu</h1>
    </SidebarHeader>

    <div>
      <button 
        v-for="section in ['general', 'avancé', 'dev']" 
        :key="section"
        @click="activeSection = section"
      >
        {{ section.charAt(0).toUpperCase() + section.slice(1) }}
      </button>
    </div>

    <SidebarContent>
      <Transition name="slide-fade" mode="out-in">
        <SidebarGroup v-if="activeSection === 'general'" key="general">
          <SidebarGroupLabel>Configuration de base</SidebarGroupLabel>
          <SidebarGroupContent>
            <div>
              <div>
                <label>Nombre de cercles</label>
                <div>
                  <input 
                    type="range" 
                    min="2" 
                    max="12" 
                    v-model.number="props.params.circleCount" 
                    @input="(e) => handleInput(e, props.applyParams)" 
                  />
                  <span>{{ props.params.circleCount }}</span>
                </div>
              </div>

              <div>
                <label>Rotation des cercles</label>
                <Select v-model="props.params.rotationMode" @update:modelValue="handleSelectUpdate">
                  <SelectTrigger>
                    <SelectValue />
                  </SelectTrigger>
                  <SelectContent>
                    <SelectItem value="same">Tous dans le même sens</SelectItem>
                    <SelectItem value="opposite">Alterné</SelectItem>
                    <SelectItem value="random">Aléatoire</SelectItem>
                  </SelectContent>
                </Select>
              </div>

              <div>
                <label>Couleur des cercles</label>
                <input 
                  type="color" 
                  v-model="props.params.circleColor" 
                  @input="(e) => handleInput(e, props.applyParams)" 
                />
              </div>

              <div>
                <label>Couleur de la balle</label>
                <input 
                  type="color" 
                  v-model="props.params.ballColor" 
                  @input="(e) => handleInput(e, props.applyParams)"
                />
              </div>
            </div>
          </SidebarGroupContent>
        </SidebarGroup>

        <SidebarGroup v-else-if="activeSection === 'avancé'" key="advanced">
          <SidebarGroupLabel>Paramètres avancés</SidebarGroupLabel>
          <SidebarGroupContent>
            <div>
              <div>
                <label>Espace initial</label>
                <div>
                  <input 
                    type="range" 
                    min="10" 
                    max="120" 
                    v-model.number="props.params.initGap" 
                    @input="(e) => handleInput(e, props.applyParams)" 
                  />
                  <span>{{ props.params.initGap }}px</span>
                </div>
              </div>

              <div>
                <label>Taille de la balle</label>
                <div>
                  <input 
                    type="range" 
                    min="6" 
                    max="32" 
                    v-model.number="props.params.ballRadius" 
                    @input="(e) => handleInput(e, props.applyParams)" 
                  />
                  <span>{{ props.params.ballRadius }}px</span>
                </div>
              </div>

              <div>
                <label>Épaisseur des cercles</label>
                <div>
                  <input 
                    type="range" 
                    min="4" 
                    max="40" 
                    v-model.number="props.params.circleWidth" 
                    @input="(e) => handleInput(e, props.applyParams)" 
                  />
                  <span>{{ props.params.circleWidth }}px</span>
                </div>
              </div>

              <div>
                <label>Vitesse de rotation</label>
                <div>
                  <input 
                    type="range" 
                    min="0.001" 
                    max="0.05" 
                    step="0.001" 
                    v-model.number="props.params.rotateSpeed" 
                    @input="(e) => handleInput(e, props.applyParams)" 
                  />
                  <span>{{ props.params.rotateSpeed.toFixed(3) }}</span>
                </div>
              </div>
            </div>
          </SidebarGroupContent>
        </SidebarGroup>

        <SidebarGroup v-else-if="activeSection === 'dev'" key="dev">
          <SidebarGroupLabel>Mode développeur</SidebarGroupLabel>
          <SidebarGroupContent>
            <div>
              <div>
                <label>Mode développeur</label>
                <input type="checkbox" v-model="devMode" @change="$emit('update:devMode', devMode)" />
              </div>

              <div v-if="devMode">
                <div>
                  <label>Afficher debug</label>
                  <input type="checkbox" v-model="showDebug" @change="$emit('update:showDebug', showDebug)" />
                </div>

                <div>
                  <label>Gravité</label>
                  <div>
                    <input 
                      type="range" 
                      min="0.05" 
                      max="0.5" 
                      step="0.01" 
                      v-model.number="props.params.gravity" 
                      @input="(e) => handleInput(e, props.applyParams)" 
                    />
                    <span>{{ props.params.gravity.toFixed(2) }}</span>
                  </div>
                </div>

                <div>
                  <label>Puissance de la gravité</label>
                  <div>
                    <input 
                      type="range" 
                      min="0.1" 
                      max="3" 
                      step="0.01" 
                      v-model.number="props.params.gravityPower" 
                      @input="(e) => handleInput(e, props.applyParams)" 
                    />
                    <span>{{ props.params.gravityPower.toFixed(2) }}</span>
                  </div>
                </div>
              </div>
            </div>
          </SidebarGroupContent>
        </SidebarGroup>
      </Transition>
    </SidebarContent>

    <SidebarFooter>
      <div>
        <span>App TikTok - Mini-jeu</span>
        <button @click="(e) => handleClick(e, props.applyParams)">Appliquer</button>
      </div>
    </SidebarFooter>
  </Sidebar>
</template>

<style scoped>
/* Removed all styles */
</style>
