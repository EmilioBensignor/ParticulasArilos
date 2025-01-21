<!-- components/PomegranateCanvas.vue -->
<template>
    <ClientOnly>
        <div ref="container" class="canvas-wrapper">
            <canvas ref="canvas"></canvas>
        </div>
    </ClientOnly>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue'

const container = ref(null)
const canvas = ref(null)
let particles = []
const cursor = { x: 9999, y: 9999 }
let animationFrameId = null
let ctx = null

// Configuración simplificada
const CANVAS_WIDTH = 1920
const CANVAS_HEIGHT = 300
const BASE_PARTICLE_SIZE = 8
const NUM_COLS = 60 // Reducido para mejor performance
const NUM_ROWS = 6

const pomegranateColors = [
    'rgba(188, 32, 38, 0.9)',    // Rojo rubí
    'rgba(220, 48, 48, 0.95)',   // Rojo brillante
    'rgba(165, 28, 32, 0.85)',   // Rojo granate
    'rgba(200, 38, 44, 0.9)',    // Rojo cereza
    'rgba(142, 24, 28, 0.88)',   // Rojo oscuro
]

class Particle {
    constructor({ x, y, radius, color }) {
        this.x = x
        this.y = y
        this.ix = x
        this.iy = y
        this.vx = 0
        this.vy = 0
        this.radius = radius
        this.color = color
        this.pushFactor = 0.8
        this.returnFactor = 0.08
        this.friction = 0.85

        this.scaleX = 0.9 + Math.random() * 0.2
        this.scaleY = 0.9 + Math.random() * 0.2
        this.rotation = Math.random() * Math.PI * 2
    }

    update() {
        // Calcular distancia al cursor
        const dx = this.x - cursor.x
        const dy = this.y - cursor.y
        const distance = Math.sqrt(dx * dx + dy * dy)
        const minDistance = 120 // Radio de influencia

        // Aplicar fuerza repulsiva si está cerca del cursor
        if (distance < minDistance) {
            const force = (minDistance - distance) / minDistance
            this.vx += (dx / distance) * force * this.pushFactor
            this.vy += (dy / distance) * force * this.pushFactor
        }

        // Fuerza de retorno al punto inicial
        const homeX = this.ix - this.x
        const homeY = this.iy - this.y
        this.vx += homeX * this.returnFactor
        this.vy += homeY * this.returnFactor

        // Aplicar fricción
        this.vx *= this.friction
        this.vy *= this.friction

        // Actualizar posición
        this.x += this.vx
        this.y += this.vy
    }

    draw() {
        ctx.save()
        ctx.translate(this.x, this.y)
        ctx.rotate(this.rotation)
        ctx.scale(this.scaleX, this.scaleY)

        // Forma base
        ctx.beginPath()
        ctx.fillStyle = this.color
        ctx.arc(0, 0, this.radius, 0, Math.PI * 2)
        ctx.fill()

        // Brillo
        const gradient = ctx.createRadialGradient(
            -this.radius * 0.3, -this.radius * 0.3, this.radius * 0.1,
            0, 0, this.radius
        )
        gradient.addColorStop(0, 'rgba(255, 255, 255, 0.3)')
        gradient.addColorStop(1, 'rgba(255, 255, 255, 0)')
        ctx.fillStyle = gradient
        ctx.fill()

        ctx.restore()
    }
}

const initParticles = () => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    ctx = canvasEl.getContext('2d')
    const particleRadius = (BASE_PARTICLE_SIZE * CANVAS_WIDTH) / 1000
    const randomOffset = particleRadius

    particles = []
    const marginY = particleRadius * 2
    const usableHeight = CANVAS_HEIGHT - (marginY * 2)
    const rowSpacing = usableHeight / (NUM_ROWS - 1)
    const colSpacing = CANVAS_WIDTH / (NUM_COLS - 1)

    for (let i = 0; i < NUM_ROWS; i++) {
        for (let j = 0; j < NUM_COLS; j++) {
            const x = j * colSpacing
            const y = marginY + (i * rowSpacing)

            particles.push(new Particle({
                x: x + (Math.random() * randomOffset * 2 - randomOffset),
                y: y + (Math.random() * randomOffset * 2 - randomOffset),
                radius: particleRadius * (0.8 + Math.random() * 0.4),
                color: pomegranateColors[Math.floor(Math.random() * pomegranateColors.length)]
            }))
        }
    }
}

const animate = () => {
    if (!ctx) return

    ctx.fillStyle = 'white'
    ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT)

    particles.forEach(particle => {
        particle.update()
        particle.draw()
    })

    animationFrameId = requestAnimationFrame(animate)
}

const updateCursorPosition = (clientX, clientY) => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    const rect = canvasEl.getBoundingClientRect()
    const scaleX = CANVAS_WIDTH / rect.width
    const scaleY = CANVAS_HEIGHT / rect.height

    cursor.x = (clientX - rect.left) * scaleX
    cursor.y = (clientY - rect.top) * scaleY
}

const handleInteraction = (e) => {
    e.preventDefault()
    const point = e.touches ? e.touches[0] : e
    updateCursorPosition(point.clientX, point.clientY)
}

const handleEnd = () => {
    cursor.x = 9999
    cursor.y = 9999
}

const resizeCanvas = () => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    canvasEl.width = CANVAS_WIDTH
    canvasEl.height = CANVAS_HEIGHT

    initParticles()
}

onMounted(() => {
    nextTick(() => {
        resizeCanvas()
        window.addEventListener('resize', resizeCanvas)

        const canvasEl = canvas.value
        if (canvasEl) {
            // Unificamos el manejo de eventos touch y mouse
            canvasEl.addEventListener('mousemove', handleInteraction, { passive: false })
            canvasEl.addEventListener('touchmove', handleInteraction, { passive: false })
            canvasEl.addEventListener('mouseleave', handleEnd)
            canvasEl.addEventListener('touchend', handleEnd)
            canvasEl.addEventListener('touchcancel', handleEnd)
        }

        animate()
    })
})

onUnmounted(() => {
    window.removeEventListener('resize', resizeCanvas)

    const canvasEl = canvas.value
    if (canvasEl) {
        canvasEl.removeEventListener('mousemove', handleInteraction)
        canvasEl.removeEventListener('touchmove', handleInteraction)
        canvasEl.removeEventListener('mouseleave', handleEnd)
        canvasEl.removeEventListener('touchend', handleEnd)
        canvasEl.removeEventListener('touchcancel', handleEnd)
    }
    if (animationFrameId) {
        cancelAnimationFrame(animationFrameId)
    }
})
</script>

<style scoped>
.canvas-wrapper {
    width: 100%;
    height: 300px;
    position: relative;
    background: white;
    overflow: hidden;
}

canvas {
    position: absolute;
    top: 50%;
    left: 0;
    width: 100%;
    height: 300px;
    transform: translateY(-50%);
    object-fit: cover;
}
</style>