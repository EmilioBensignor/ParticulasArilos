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

// Configuración fija para 1920px
const CANVAS_WIDTH = 1920
const CANVAS_HEIGHT = 300
const VISIBLE_HEIGHT = 300
const BASE_PARTICLE_SIZE = 8
const NUM_COLS = 80
const NUM_ROWS = 6

const pomegranateColors = [
    'rgba(188, 32, 38, 0.9)',    // Rojo rubí
    'rgba(220, 48, 48, 0.95)',   // Rojo brillante
    'rgba(165, 28, 32, 0.85)',   // Rojo granate
    'rgba(200, 38, 44, 0.9)',    // Rojo cereza
    'rgba(142, 24, 28, 0.88)',   // Rojo oscuro
]

class Particle {
    constructor({ x, y, radius, color, minDist, pushFactor, pullFactor, dampFactor }) {
        this.x = x
        this.y = y
        this.ix = x
        this.iy = y
        this.ax = 0
        this.ay = 0
        this.vx = 0
        this.vy = 0
        this.radius = radius
        this.color = color
        this.minDist = minDist
        this.pushFactor = pushFactor
        this.pullFactor = pullFactor
        this.dampFactor = dampFactor

        this.scaleX = 0.9 + Math.random() * 0.2
        this.scaleY = 0.9 + Math.random() * 0.2
        this.rotation = Math.random() * Math.PI * 2
    }

    update() {
        let dx, dy, dd, distDelta

        dx = this.ix - this.x
        dy = this.iy - this.y
        dd = Math.sqrt(dx * dx + dy * dy)

        this.ax = dx * this.pullFactor
        this.ay = dy * this.pullFactor

        dx = this.x - cursor.x
        dy = this.y - cursor.y
        dd = Math.sqrt(dx * dx + dy * dy)

        distDelta = this.minDist - dd

        if (dd < this.minDist) {
            this.ax += (dx / dd) * distDelta * this.pushFactor
            this.ay += (dy / dd) * distDelta * this.pushFactor
        }

        this.vx += this.ax
        this.vy += this.ay

        this.vx *= this.dampFactor
        this.vy *= this.dampFactor

        this.x += this.vx
        this.y += this.vy
    }

    draw(context) {
        context.save()
        context.translate(this.x, this.y)
        context.rotate(this.rotation)
        context.scale(this.scaleX, this.scaleY)

        // Forma base del arilo
        context.beginPath()
        context.fillStyle = this.color
        context.arc(0, 0, this.radius, 0, Math.PI * 2)
        context.fill()

        // Brillo
        const gradient = context.createRadialGradient(
            -this.radius * 0.3, -this.radius * 0.3, this.radius * 0.1,
            0, 0, this.radius
        )
        gradient.addColorStop(0, 'rgba(255, 255, 255, 0.3)')
        gradient.addColorStop(1, 'rgba(255, 255, 255, 0)')
        context.fillStyle = gradient
        context.fill()

        // Textura
        context.strokeStyle = 'rgba(120, 20, 20, 0.1)'
        context.lineWidth = 0.5
        for (let i = 0; i < 3; i++) {
            context.beginPath()
            context.arc(
                Math.random() * this.radius * 0.4,
                Math.random() * this.radius * 0.4,
                this.radius * 0.4,
                0,
                Math.PI * 2
            )
            context.stroke()
        }

        context.restore()
    }
}

const initParticles = () => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    const particleRadius = (BASE_PARTICLE_SIZE * CANVAS_WIDTH) / 1000
    const randomPosition = particleRadius * 2

    particles = []

    const verticalOffset = (CANVAS_HEIGHT - VISIBLE_HEIGHT) / 2
    const verticalMargin = particleRadius * 3
    const usableHeight = VISIBLE_HEIGHT - (verticalMargin * 2)
    const rowSpacing = usableHeight / (NUM_ROWS - 1)

    for (let i = 0; i < NUM_ROWS; i++) {
        for (let j = 0; j < NUM_COLS; j++) {
            const x = (j * (CANVAS_WIDTH / (NUM_COLS - 1)))
            const y = verticalOffset + verticalMargin + (i * rowSpacing)

            const particle = new Particle({
                x: x + (Math.random() * randomPosition * 2 - randomPosition),
                y: y + (Math.random() * randomPosition * 2 - randomPosition),
                radius: particleRadius * (0.8 + Math.random() * 0.4),
                color: pomegranateColors[Math.floor(Math.random() * pomegranateColors.length)],
                minDist: (CANVAS_WIDTH / 20) + Math.random() * (CANVAS_WIDTH / 40),
                pushFactor: 0.03 + Math.random() * 0.04,
                pullFactor: 0.01,
                dampFactor: 0.90 + Math.random() * 0.01
            })

            particles.push(particle)
        }
    }
}

const animate = () => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    const context = canvasEl.getContext('2d', {
        alpha: false,
        desynchronized: true
    })
    if (!context) return

    context.fillStyle = 'white'
    context.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT)

    particles.forEach(particle => {
        particle.update()
        particle.draw(context)
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

const handleTouch = (e) => {
    e.preventDefault()
    const touch = e.touches[0]
    if (touch) {
        const rect = canvas.value.getBoundingClientRect()
        const scaleX = CANVAS_WIDTH / rect.width
        const scaleY = CANVAS_HEIGHT / rect.height

        // Ajustamos la sensibilidad para mobile
        const touchX = (touch.clientX - rect.left) * scaleX
        const touchY = (touch.clientY - rect.top) * scaleY

        cursor.x = touchX
        cursor.y = touchY
    }
}

const handleMouseMove = (e) => {
    if (e.touches) return // Ignoramos si es un evento touch
    e.preventDefault()
    updateCursorPosition(e.clientX, e.clientY)
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

    // Optimizaciones del contexto
    const context = canvasEl.getContext('2d', {
        alpha: false,
        desynchronized: true
    })
    if (context) {
        context.imageSmoothingEnabled = true
        context.imageSmoothingQuality = 'high'
    }

    initParticles()
}

onMounted(() => {
    nextTick(() => {
        resizeCanvas()
        window.addEventListener('resize', resizeCanvas)

        const canvasEl = canvas.value
        if (canvasEl) {
            // Mouse events
            canvasEl.addEventListener('mousemove', handleMouseMove, { passive: false })
            canvasEl.addEventListener('mouseleave', handleEnd)

            // Touch events con opciones optimizadas
            canvasEl.addEventListener('touchstart', handleTouch, {
                passive: false,
                capture: true
            })
            canvasEl.addEventListener('touchmove', handleTouch, {
                passive: false,
                capture: true
            })
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
        canvasEl.removeEventListener('mousemove', handleMouseMove)
        canvasEl.removeEventListener('mouseleave', handleEnd)
        canvasEl.removeEventListener('touchstart', handleTouch)
        canvasEl.removeEventListener('touchmove', handleTouch)
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
    touch-action: none;
}
</style>