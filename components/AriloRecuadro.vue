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

const CANVAS_WIDTH = 2000
const CANVAS_HEIGHT = 600
const VISIBLE_HEIGHT = 450
const BASE_PARTICLE_SIZE = 5.5
const SIDE_COLS = 7 // Menos columnas para los laterales
const BOTTOM_COLS = 25 // Mantener columnas para abajo
const SIDE_ROWS = 8 // Más filas para los laterales
const BOTTOM_ROWS = 3 // Menos filas para abajo
const SIDE_MARGIN = 50 // Margen lateral
const CENTER_GAP = 1050 // Espacio en el centro para el hero

class Particle {
    constructor({ x, y, radius, minDist, pushFactor, pullFactor, dampFactor }) {
        this.x = x
        this.y = y
        this.ix = x
        this.iy = y
        this.ax = 0
        this.ay = 0
        this.vx = 0
        this.vy = 0
        this.radius = radius
        this.minDist = minDist
        this.pushFactor = pushFactor
        this.pullFactor = pullFactor
        this.dampFactor = dampFactor

        this.scaleX = 0.9 + Math.random() * 0.2
        this.scaleY = 0.9 + Math.random() * 0.2
        this.rotation = Math.random() * Math.PI * 2

        // Array con las rutas de las imágenes
        const ariloImages = [
            '/images/Arilo.png',
            '/images/Arilo-3.png'
        ]
        // Seleccionar una imagen aleatoria
        const randomImage = ariloImages[Math.floor(Math.random() * ariloImages.length)]

        // Cargar imagen del arilo
        this.image = new Image()
        this.image.src = randomImage
        this.imageLoaded = false
        this.image.onload = () => {
            this.imageLoaded = true
        }
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
        if (!this.imageLoaded) return

        context.save()
        context.translate(this.x, this.y)
        context.rotate(this.rotation)
        context.scale(this.scaleX, this.scaleY)

        // Dibujar la imagen centrada
        const size = this.radius * 2
        context.drawImage(
            this.image,
            -size,
            -size,
            size * 2,
            size * 2
        )

        context.restore()
    }
}

const initParticles = () => {
    const canvasEl = canvas.value
    if (!canvasEl) return

    const particleRadius = (BASE_PARTICLE_SIZE * CANVAS_WIDTH) / 1000
    const randomPosition = particleRadius * 2
    particles = []

    // Función helper para crear partículas
    const createParticle = (x, y) => {
        return new Particle({
            x: x + (Math.random() * randomPosition * 2 - randomPosition),
            y: y + (Math.random() * randomPosition * 2 - randomPosition),
            radius: particleRadius * (0.8 + Math.random() * 0.4),
            minDist: (CANVAS_WIDTH / 20) + Math.random() * (CANVAS_WIDTH / 40),
            pushFactor: 0.03 + Math.random() * 0.04,
            pullFactor: 0.01,
            dampFactor: 0.90 + Math.random() * 0.01
        })
    }

    // Calcular espaciados
    const verticalOffset = (CANVAS_HEIGHT - VISIBLE_HEIGHT) / 2
    const sideWidth = (CANVAS_WIDTH - CENTER_GAP) / 2

    // Arilos laterales izquierdos
    const leftColSpacing = sideWidth / (SIDE_COLS - 1)
    const sideRowSpacing = VISIBLE_HEIGHT / (SIDE_ROWS - 1)

    for (let i = 0; i < SIDE_ROWS; i++) {
        for (let j = 0; j < SIDE_COLS; j++) {
            const x = SIDE_MARGIN + (j * leftColSpacing)
            const y = verticalOffset + (i * sideRowSpacing)
            particles.push(createParticle(x, y))
        }
    }

    // Arilos laterales derechos
    const rightStart = CANVAS_WIDTH - sideWidth
    for (let i = 0; i < SIDE_ROWS; i++) {
        for (let j = 0; j < SIDE_COLS; j++) {
            const x = rightStart + (j * leftColSpacing)
            const y = verticalOffset + (i * sideRowSpacing)
            particles.push(createParticle(x, y))
        }
    }

    // Arilos inferiores
    const bottomStart = CANVAS_HEIGHT - verticalOffset - (VISIBLE_HEIGHT / 4)
    const bottomColSpacing = CANVAS_WIDTH / (BOTTOM_COLS - 1)
    const bottomRowSpacing = (VISIBLE_HEIGHT / 4) / (BOTTOM_ROWS - 1)

    for (let i = 0; i < BOTTOM_ROWS; i++) {
        for (let j = 0; j < BOTTOM_COLS; j++) {
            const x = (j * bottomColSpacing)
            const y = bottomStart + (i * bottomRowSpacing)
            particles.push(createParticle(x, y))
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
    height: 450px;
    position: absolute;
    top: 0;
    background: transparent;
    overflow: hidden;
    pointer-events: none;
}

canvas {
    position: absolute;
    top: 50%;
    left: 0;
    width: 100%;
    height: 450px;
    transform: translateY(-50%);
    object-fit: cover;
    pointer-events: auto;
}
</style>