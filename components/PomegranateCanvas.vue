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
let isInView = false
let mouseIsOver = false

// Configuración
const CANVAS_WIDTH = 1920
const CANVAS_HEIGHT = 300
const VISIBLE_HEIGHT = 300
const BASE_PARTICLE_SIZE = 8
const NUM_COLS = 80
const NUM_ROWS = 6
const PATH_WIDTH = 180 // Ligeramente más estrecho para mejor control
const PUSH_STRENGTH = 1.5 // Suavizado para mejor control
const RETURN_SPEED = 0.015 // Velocidad de retorno ajustada

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
        this.targetX = x
        this.targetY = y

        this.scaleX = 0.9 + Math.random() * 0.2
        this.scaleY = 0.9 + Math.random() * 0.2
        this.rotation = Math.random() * Math.PI * 2
    }

    update() {
        // Reset aceleración
        this.ax = 0
        this.ay = 0

        if (isInView) {
            // Efecto del mouse
            const dx = this.x - cursor.x
            const dy = this.y - cursor.y
            const dd = Math.sqrt(dx * dx + dy * dy)

            if (dd < this.minDist && mouseIsOver) {
                // Efecto repulsivo del mouse
                const force = (1 - dd / this.minDist) * this.pushFactor
                this.ax += (dx / dd) * force
                this.ay += (dy / dd) * force

                // Crear efecto de camino
                const distFromCenterX = Math.abs(this.x - CANVAS_WIDTH / 2)
                if (distFromCenterX < PATH_WIDTH) {
                    const direction = this.x < CANVAS_WIDTH / 2 ? -1 : 1
                    this.ax += direction * PUSH_STRENGTH * (1 - distFromCenterX / PATH_WIDTH)
                }
            }
        }

        // Fuerza de retorno a posición inicial
        const dx = this.ix - this.x
        const dy = this.iy - this.y
        this.ax += dx * RETURN_SPEED
        this.ay += dy * RETURN_SPEED

        // Actualizar velocidad y posición
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
                dampFactor: 0.92 + Math.random() * 0.01
            })

            particles.push(particle)
        }
    }
}

const animate = () => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    const context = canvasEl.getContext('2d')
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

let observer = null

const setupIntersectionObserver = () => {
    if (!container.value) return

    observer = new IntersectionObserver(
        (entries) => {
            const [entry] = entries
            isInView = entry.isIntersecting
            if (!isInView) {
                mouseIsOver = false
                cursor.x = 9999
                cursor.y = 9999
            }
        },
        {
            threshold: 0.2, // 20% del elemento visible
            rootMargin: '50px' // Margen adicional para detectar antes
        }
    )

    observer.observe(container.value)
}

const handleMouseMove = (e) => {
    if (!isInView) return
    e.preventDefault()
    mouseIsOver = true
    updateCursorPosition(e.clientX, e.clientY)
}

const handleTouchMove = (e) => {
    if (!isInView) return
    e.preventDefault()
    mouseIsOver = true
    const touch = e.touches[0]
    updateCursorPosition(touch.clientX, touch.clientY)
}

const handleMouseLeave = () => {
    mouseIsOver = false
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
        setupIntersectionObserver()
        window.addEventListener('resize', resizeCanvas)

        const canvasEl = canvas.value
        if (canvasEl) {
            canvasEl.addEventListener('mousemove', handleMouseMove, { passive: false })
            canvasEl.addEventListener('mouseenter', () => mouseIsOver = true)
            canvasEl.addEventListener('mouseleave', handleMouseLeave)
            canvasEl.addEventListener('touchmove', handleTouchMove, { passive: false })
            canvasEl.addEventListener('touchend', handleMouseLeave)
            canvasEl.addEventListener('touchcancel', handleMouseLeave)
        }

        animate()
    })
})

onUnmounted(() => {
    window.removeEventListener('resize', resizeCanvas)
    if (observer) {
        observer.disconnect()
    }

    const canvasEl = canvas.value
    if (canvasEl) {
        canvasEl.removeEventListener('mousemove', handleMouseMove)
        canvasEl.removeEventListener('mouseenter', () => mouseIsOver = true)
        canvasEl.removeEventListener('mouseleave', handleMouseLeave)
        canvasEl.removeEventListener('touchmove', handleTouchMove)
        canvasEl.removeEventListener('touchend', handleMouseLeave)
        canvasEl.removeEventListener('touchcancel', handleMouseLeave)
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