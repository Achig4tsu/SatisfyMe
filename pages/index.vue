<template>


    <div class="container mx-auto flex flex-col items-center justify-center h-screen"> 
        <h1 class="text-5xl font-bold my-8">SatisfyMe</h1>
        <div class="Button-Section flex flex-col items-center justify-center gap-3 w-full max-w-md">        
            
            <NuxtLink to="/play" class="w-full">
                <Button class="w-full">Démarrer</Button>
            </NuxtLink>
            <div class="Drawer-Section w-full">
                 <Drawer>
                    <DrawerTrigger as-child>
                    <Button variant="outline" class="w-full">
                        Paramètres
                    </Button>
                    </DrawerTrigger>
                    <DrawerContent>
                    <div class="mx-auto w-full max-w-sm">
                        <DrawerHeader>
                        <DrawerTitle>Paramètres</DrawerTitle>
                        <DrawerDescription>Modifiez vos paramètres ici.</DrawerDescription>
                        </DrawerHeader>
                        <div class="p-4 pb-0">
                        <div class="flex flex-col gap-4 justify-center space-x-2">
                            <Select>
                                <SelectTrigger class="w-full">
                                <SelectValue placeholder="Choisissez une langue." />
                                </SelectTrigger>
                                <SelectContent>
                                <SelectGroup>
                                    <SelectItem value="french">
                                    Français
                                    </SelectItem>
                                    <SelectItem value="english">
                                    English
                                    </SelectItem>
                                </SelectGroup>
                                </SelectContent>
                            </Select>

                            <Separator />

                            
                            <ToggleGroup type="single" class="w-full" defaultValue="system" variant="outline" aria-label="Theme">
                                <ToggleGroupItem value="light" class="w-full">
                                <Sun /> Light
                                </ToggleGroupItem>
                                <ToggleGroupItem value="dark" class="w-full">
                                <MoonStar /> Dark
                                </ToggleGroupItem>
                                <ToggleGroupItem value="system" class="w-full">
                                <LaptopMinimal /> System
                                </ToggleGroupItem>
                            </ToggleGroup>
                            

                        </div>
                        <div class="p-4 pb-0">
                        </div>

                        </div>
                        <DrawerFooter>
                        <DrawerClose as-child>
                            <Button>Appliquer</Button>
                        </DrawerClose>
                        <DrawerClose as-child>
                            <Button variant="destructive">
                            Cancel
                            </Button>
                        </DrawerClose>
                        </DrawerFooter>
                    </div>
                    </DrawerContent>
                </Drawer>
            </div>

        </div>

    </div>  
</template>

<script setup lang="ts">
import { Minus, Plus } from 'lucide-vue-next'
import { Sun } from 'lucide-vue-next';
import { LaptopMinimal } from 'lucide-vue-next';
import { MoonStar } from 'lucide-vue-next';
import { ref } from 'vue'
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,100..1000;1,9..40,100..1000&display=swap');


*{
  font-family: 'DM Sans', sans-serif;
}
</style>