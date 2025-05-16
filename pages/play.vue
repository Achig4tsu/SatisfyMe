<template>
  <div>
    <SidebarProvider>
      <AppSidebar :params="params" :applyParams="applyParams" v-model:open="sidebarOpen" v-model:devMode="devMode" v-model:showDebug="showDebug" />
      <main>
        <SidebarTrigger />
        <slot />
      </main>
    </SidebarProvider>

    <div>
      <h1>Mini-jeu : Passe la balle dans les fentes !</h1>

      <div>
        <canvas ref="canvas" width="400" height="400"></canvas>

        <Transition name="fade">
          <div v-if="gameOver">
            <p>Partie terminée !</p>
            <button @click="restart">Rejouer</button>
          </div>
        </Transition>
      </div>

      <div>
        <button @click="goHome">Retour à l'accueil</button>
      </div>

      <div>
        <p><strong>Instructions :</strong></p>
        <ul>
          <li>Cliquez et faites glisser pour lancer la balle</li>
          <li>Appuyez sur <span>Espace</span> pour activer la gravité</li>
          <li>Faites passer la balle par les fentes pour éliminer les cercles</li>
          <li>Configurez le jeu en ouvrant le menu latéral</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { SidebarProvider, SidebarTrigger } from '@/components/ui/sidebar'
import AppSidebar from '@/components/AppSidebar.vue'
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const goHome = () => router.push('/')

const canvas = ref<HTMLCanvasElement | null>(null)
const gameArea = ref<HTMLDivElement | null>(null)
const gameOver = ref(false)
const sidebarOpen = ref(false)
const devMode = ref(false)
const showDebug = ref(false)

const params = ref({
  initGap: 60,
  circleCount: 8,
  rotationMode: 'random', // 'same', 'opposite', 'random'
  circleColor: '#4f46e5',
  ballColor: '#fff',
  ballRadius: 12,
  gravity: 0.18,
  circleWidth: 18,
  rotateSpeed: 0.01,
  gravityPower: 1,
})

let BALL_RADIUS = params.value.ballRadius
let GRAVITY = params.value.gravity
let CIRCLE_WIDTH = params.value.circleWidth
let ROTATE_SPEED = params.value.rotateSpeed
let GRAVITY_POWER = params.value.gravityPower
const GAP_ANGLE = Math.PI / 4 // 45°
const BALL_SPEED = 3

let ctx: CanvasRenderingContext2D | null = null
let animationId: number | null = null
let circles: Array<any> = []
let ball = { x: 200, y: 200, vx: 0, vy: 0 }
let isMoving = false
let gravityMode = false

// Ajout de la physique réaliste (parabole, rebond, frottements)
let lastTimestamp = 0
const FRICTION = 0.995
const RESTITUTION = 0.78 // rebond
const AIR_RESISTANCE = 0.999

function randomAngle() {
  return Math.random() * Math.PI * 2
}

function getRotationSpeed(i: number) {
  if (params.value.rotationMode === 'same') return ROTATE_SPEED
  if (params.value.rotationMode === 'opposite') return i % 2 === 0 ? ROTATE_SPEED : -ROTATE_SPEED
  return (Math.random() > 0.5 ? 1 : -1) * (ROTATE_SPEED + Math.random() * 0.01)
}

function resetGame() {
  BALL_RADIUS = params.value.ballRadius
  GRAVITY = params.value.gravity
  CIRCLE_WIDTH = params.value.circleWidth
  ROTATE_SPEED = params.value.rotateSpeed
  GRAVITY_POWER = params.value.gravityPower
  circles = []
  for (let i = 0; i < params.value.circleCount; i++) {
    // Calcul du rayon, s'assure qu'il reste positif
    const r = Math.max(10, 180 - i * (CIRCLE_WIDTH + 6) - params.value.initGap)
    circles.push({
      r,
      angle: randomAngle(),
      speed: getRotationSpeed(i),
      gap: GAP_ANGLE + Math.random() * (Math.PI / 8),
      color: params.value.circleColor,
      visible: true,
    })
  }
  ball.x = 200
  ball.y = 200
  ball.vx = 0
  ball.vy = 0
  isMoving = false
  gravityMode = false
  gameOver.value = false
}

function applyParams() {
  BALL_RADIUS = params.value.ballRadius
  GRAVITY = params.value.gravity
  CIRCLE_WIDTH = params.value.circleWidth
  ROTATE_SPEED = params.value.rotateSpeed
  GRAVITY_POWER = params.value.gravityPower
  resetGame()
  draw()
}

function draw() {
  if (!ctx) return
  ctx.clearRect(0, 0, 400, 400)
  for (let i = 0; i < circles.length; i++) {
    const c = circles[i]
    if (!c.visible) continue
    ctx.save()
    ctx.translate(200, 200)
    ctx.rotate(c.angle)
    ctx.beginPath()
    ctx.strokeStyle = c.color
    ctx.lineWidth = CIRCLE_WIDTH
    ctx.arc(0, 0, c.r, c.gap / 2, 2 * Math.PI - c.gap / 2)
    ctx.stroke()
    ctx.restore()
  }
  // Bouclier (rouge) autour de la balle
  if (devMode.value && showDebug.value) {
    ctx.save()
    ctx.beginPath()
    ctx.arc(ball.x, ball.y, BALL_RADIUS + 1, 0, 2 * Math.PI)
    ctx.strokeStyle = 'red'
    ctx.lineWidth = 1
    ctx.setLineDash([4, 2])
    ctx.stroke()
    ctx.restore()
  }
  // Trajectoire prévisionnelle (bleu)
  if (devMode.value && showDebug.value && isMoving) {
    let simBall = { x: ball.x, y: ball.y, vx: ball.vx, vy: ball.vy }
    ctx.save()
    ctx.beginPath()
    ctx.moveTo(simBall.x, simBall.y)
    for (let t = 0; t < 120; t++) {
      // Simule la physique simplifiée (sans collision cercles)
      const Cd = 0.47, rho = 1.22, mass = 1
      const area = Math.PI * BALL_RADIUS * BALL_RADIUS / 10000
      let fx = -0.5 * Cd * rho * area * simBall.vx * Math.abs(simBall.vx)
      let fy = -0.5 * Cd * rho * area * simBall.vy * Math.abs(simBall.vy)
      let ax = fx / mass
      let ay = (gravityMode ? GRAVITY * GRAVITY_POWER : 0) + fy / mass
      simBall.vx += ax * (16 / 1000)
      simBall.vy += ay * (16 / 1000)
      simBall.x += simBall.vx * 1
      simBall.y += simBall.vy * 1
      ctx.lineTo(simBall.x, simBall.y)
      if (simBall.x < BALL_RADIUS || simBall.x > 400 - BALL_RADIUS || simBall.y < BALL_RADIUS || simBall.y > 400 - BALL_RADIUS) break
    }
    ctx.strokeStyle = 'blue'
    ctx.lineWidth = 1
    ctx.setLineDash([2, 2])
    ctx.stroke()
    ctx.restore()
  }
  // Balle
  ctx.save()
  ctx.beginPath()
  ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, 2 * Math.PI)
  ctx.fillStyle = params.value.ballColor
  ctx.shadowColor = '#0008'
  ctx.shadowBlur = 8
  ctx.fill()
  ctx.restore()
}

function updatePhysics(dt: number) {
  for (const c of circles) {
    if (c.visible) c.angle += c.speed * dt / 16
  }
  if (isMoving) {
    const Cd = 0.47
    const rho = 1.22
    const mass = 1
    const area = Math.PI * BALL_RADIUS * BALL_RADIUS / 10000
    let fx = -0.5 * Cd * rho * area * ball.vx * Math.abs(ball.vx)
    let fy = -0.5 * Cd * rho * area * ball.vy * Math.abs(ball.vy)
    let ax = fx / mass
    let ay = (gravityMode ? GRAVITY * GRAVITY_POWER : 0) + fy / mass
    ball.vx += ax * (dt / 1000)
    ball.vy += ay * (dt / 1000)
    // Découpage du déplacement en sous-steps pour éviter le tunneling
    const steps = 8
    for (let step = 0; step < steps; step++) {
      ball.x += ball.vx * dt / 16 / steps
      ball.y += ball.vy * dt / 16 / steps
      // Bouclier : collision stricte avec cercles
      let shieldCollided = false
      for (let i = 0; i < circles.length; i++) {
        const c = circles[i]
        if (!c.visible) continue
        let dx = ball.x - 200
        let dy = ball.y - 200
        let dist = Math.sqrt(dx * dx + dy * dy)
        const minDist = c.r - CIRCLE_WIDTH / 2 - (BALL_RADIUS + 1)
        const maxDist = c.r + CIRCLE_WIDTH / 2 + (BALL_RADIUS + 1)
        let iter = 0
        while (dist > minDist && dist < maxDist && iter < 5) {
          const angle = Math.atan2(dy, dx)
          let a = (angle - c.angle) % (2 * Math.PI)
          if (a < 0) a += 2 * Math.PI
          if (a > Math.PI) a -= 2 * Math.PI
          if (Math.abs(a) < c.gap / 2) {
            c.visible = false
            break
          } else {
            // Bouclier touche : applique rebond
            const norm = { x: dx / dist, y: dy / dist }
            const vDotN = ball.vx * norm.x + ball.vy * norm.y
            if (vDotN < 0) {
              ball.vx -= (1 + RESTITUTION) * vDotN * norm.x
              ball.vy -= (1 + RESTITUTION) * vDotN * norm.y
              ball.vx *= FRICTION
              ball.vy *= FRICTION
              if (gravityMode) ball.vy += GRAVITY * GRAVITY_POWER * 2
            }
            // Repousse la balle hors du bouclier
            const overlap = (CIRCLE_WIDTH / 2 + (BALL_RADIUS + 1)) - Math.abs(dist - c.r)
            if (overlap > 0) {
              ball.x += norm.x * overlap * 1.2
              ball.y += norm.y * overlap * 1.2
              dx = ball.x - 200
              dy = ball.y - 200
              dist = Math.sqrt(dx * dx + dy * dy)
            }
            shieldCollided = true
          }
          iter++ 
        }
        if (shieldCollided) break
      }
    }
    // Win
    if (circles.every(c => !c.visible)) {
      isMoving = false
      gameOver.value = true
    }
    // Collision bords (inspiré CodePen)
    if (ball.x > 400 - BALL_RADIUS) {
      ball.vx *= -RESTITUTION
      ball.x = 400 - BALL_RADIUS
    }
    if (ball.y > 400 - BALL_RADIUS) {
      ball.vy *= -RESTITUTION
      ball.y = 400 - BALL_RADIUS
      if (Math.abs(ball.vy) < 0.5) ball.vy = 0
    }
    if (ball.x < BALL_RADIUS) {
      ball.vx *= -RESTITUTION
      ball.x = BALL_RADIUS
    }
    if (ball.y < BALL_RADIUS) {
      ball.vy *= -RESTITUTION
      ball.y = BALL_RADIUS
    }
    if (Math.abs(ball.vx) < 0.01 && Math.abs(ball.vy) < 0.01 && gravityMode) {
      isMoving = false
      gameOver.value = true
    }
  }
}

function loop(ts = 0) {
  const dt = Math.min(32, ts - lastTimestamp || 16)
  lastTimestamp = ts
  updatePhysics(dt)
  draw()
  if (!gameOver.value) {
    animationId = requestAnimationFrame(loop)
  }
}

function shootBall(e: MouseEvent) {
  if (isMoving || gameOver.value) return
  const rect = canvas.value!.getBoundingClientRect()
  const mx = e.clientX - rect.left
  const my = e.clientY - rect.top
  const dx = mx - 200
  const dy = my - 200
  const len = Math.sqrt(dx * dx + dy * dy)
  if (len < 10) return
  // Lancement avec parabole (angle et force)
  const force = Math.min(1, len / 200)
  const angle = Math.atan2(dy, dx)
  ball.vx = Math.cos(angle) * BALL_SPEED * 2 * force
  ball.vy = Math.sin(angle) * BALL_SPEED * 2 * force
  isMoving = true
  gravityMode = false
}

function restart() {
  applyParams()
  lastTimestamp = 0
  animationId = requestAnimationFrame(loop)
}

function onKeydown(e: KeyboardEvent) {
  if (e.code === 'Space' && isMoving && !gravityMode) {
    gravityMode = true
  }
}

onMounted(() => {
  ctx = canvas.value?.getContext('2d') || null
  applyParams()
  canvas.value?.addEventListener('mousedown', shootBall)
  window.addEventListener('keydown', onKeydown)
  lastTimestamp = 0
  animationId = requestAnimationFrame(loop)
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
  canvas.value?.removeEventListener('mousedown', shootBall)
  window.removeEventListener('keydown', onKeydown)
})
</script>

<style scoped>
/* Removed all styles */
</style>
