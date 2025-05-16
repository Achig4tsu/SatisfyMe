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
  params: Object,
  applyParams: Function
})

const emits = defineEmits(['update:devMode', 'update:showDebug'])
const devMode = defineModel('devMode', { type: Boolean, default: false })
const showDebug = defineModel('showDebug', { type: Boolean, default: false })

// Section active
const activeSection = ref('general')
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
                    @input="props.applyParams" 
                  />
                  <span>{{ props.params.circleCount }}</span>
                </div>
              </div>

              <div>
                <label>Rotation des cercles</label>
                <Select v-model="props.params.rotationMode" @update:modelValue="props.applyParams">
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
                  @input="props.applyParams" 
                />
              </div>

              <div>
                <label>Couleur de la balle</label>
                <input 
                  type="color" 
                  v-model="props.params.ballColor" 
                  @input="props.applyParams"
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
                    @input="props.applyParams" 
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
                    @input="props.applyParams" 
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
                    @input="props.applyParams" 
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
                    @input="props.applyParams" 
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
                      @input="props.applyParams" 
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
                      @input="props.applyParams" 
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
        <button @click="props.applyParams">Appliquer</button>
      </div>
    </SidebarFooter>
  </Sidebar>
</template>

<style scoped>
/* Removed all styles */
</style>
