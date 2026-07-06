<script setup>
/**
 * BrandBook.vue — v2
 * ─────────────────────────────────────────────────────────────
 * Manual de marca interativo — Ramon Bastos Advocacia Agro.
 *
 * Todos os arquivos referenciados em /IMG são reais e já foram
 * entregues (LogoBrancaOficial.svg, logo1final.svg, logo2final.svg,
 * logo3final.svg, logoname1.svg, logoC1.svg, logoG1.svg).
 *
 * Novidades desta versão:
 *  • Barra de progresso de leitura + navegação lateral (desktop)
 *    e gaveta lateral com pontos de progresso (mobile), 100% touch.
 *  • Cursor magnético e trilha suave (somente ponteiro fino / sem
 *    "prefers-reduced-motion").
 *  • Campo de espigas ambiente na capa (parallax + drift), com
 *    tilt 3D do símbolo ao mover o mouse / inclinar o celular.
 *  • Construção do símbolo: arraste diretamente na régua, play
 *    com velocidade e direção, ao invés de um <input range> solto.
 *  • Seção de versões agora é um verdadeiro laboratório de fundo:
 *    presets + seletor de cor livre (<input type="color">) +
 *    checador de contraste WCAG ao vivo, decidindo sozinho se o
 *    logo deve virar claro ou escuro sobre a cor escolhida.
 *  • Paleta com botão "usar como fundo" que manda a cor direto
 *    pro laboratório de versões, e cartela de gradientes.
 *  • Tamanho mínimo agora é um slider ao vivo com aviso de quebra.
 *  • Tipografia com slider de peso e exemplo de pareamento real.
 *  • Aplicações (mockups) herdam a mesma cor escolhida no laboratório.
 * ─────────────────────────────────────────────────────────────
 */
import { ref, reactive, computed, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  brandName: { type: String, default: 'Ramon Bastos' },
  brandSuffix: { type: String, default: 'Advocacia Agro' },
  assetsBase: { type: String, default: '/IMG' }
})

const asset = (name) => `${props.assetsBase}/${name}`

/* ═════════════════════ navegação / scrollspy ═════════════════════ */
const sections = [
  { id: 'conceito', label: 'Conceito' },
  { id: 'construcao', label: 'Construção' },
  { id: 'versoes', label: 'Laboratório de fundo' },
  { id: 'protecao', label: 'Área de proteção' },
  { id: 'tamanho', label: 'Tamanho mínimo' },
  { id: 'cores', label: 'Cores' },
  { id: 'tipografia', label: 'Tipografia' },
  { id: 'incorretos', label: 'Usos incorretos' },
  { id: 'aplicacoes', label: 'Aplicações' },
  { id: 'arquivos', label: 'Arquivos' }
]
const activeSection = ref(sections[0].id)
const rootEl = ref(null)
const mobileNavOpen = ref(false)
let spyObserver = null

function scrollToSection(id) {
  document.getElementById(id)?.scrollIntoView({ behavior: prefersReducedMotion.value ? 'auto' : 'smooth', block: 'start' })
  mobileNavOpen.value = false
}

const activeSectionIndex = computed(() => sections.findIndex((s) => s.id === activeSection.value))

/* ═════════════════════ motion / reveal genérico ═════════════════════ */
const prefersReducedMotion = ref(false)
const isFinePointer = ref(false)
let motionQuery
const revealed = reactive(new Set())
let revealObserver = null

function isRevealed(key) {
  return revealed.has(key)
}

/* ═════════════════════ progresso de leitura ═════════════════════ */
const scrollProgress = ref(0)
function updateScrollProgress() {
  const el = document.scrollingElement || document.documentElement
  const max = el.scrollHeight - el.clientHeight
  scrollProgress.value = max > 0 ? Math.min(100, Math.max(0, (el.scrollTop / max) * 100)) : 0
}

/* ═════════════════════ cursor magnético + trilha ═════════════════════ */
const cursor = reactive({ x: 0, y: 0, active: false, hover: false })
function onWindowPointerMove(e) {
  if (!isFinePointer.value) return
  cursor.x = e.clientX
  cursor.y = e.clientY
  cursor.active = true
}
function onMagnetMove(e) {
  if (prefersReducedMotion.value || !isFinePointer.value) return
  const el = e.currentTarget
  const rect = el.getBoundingClientRect()
  const relX = e.clientX - rect.left - rect.width / 2
  const relY = e.clientY - rect.top - rect.height / 2
  el.style.setProperty('--mx', `${relX * 0.22}px`)
  el.style.setProperty('--my', `${relY * 0.22}px`)
  cursor.hover = true
}
function onMagnetLeave(e) {
  e.currentTarget.style.setProperty('--mx', `0px`)
  e.currentTarget.style.setProperty('--my', `0px`)
  cursor.hover = false
}

/* ═════════════════════ capa: campo de espigas + tilt 3D ═════════════════════ */
const heroTilt = reactive({ x: 0, y: 0 })
function onHeroMove(e) {
  if (prefersReducedMotion.value) return
  const rect = e.currentTarget.getBoundingClientRect()
  const px = (e.clientX - rect.left) / rect.width - 0.5
  const py = (e.clientY - rect.top) / rect.height - 0.5
  heroTilt.x = px * 14
  heroTilt.y = py * 14
}
function onHeroLeave() {
  heroTilt.x = 0
  heroTilt.y = 0
}
function onHeroTiltDevice(e) {
  if (prefersReducedMotion.value || !e.gamma) return
  heroTilt.x = Math.max(-14, Math.min(14, e.gamma / 3))
  heroTilt.y = Math.max(-14, Math.min(14, (e.beta - 45) / 4))
}
const heroParticles = Array.from({ length: 14 }).map((_, i) => ({
  id: i,
  left: `${(i * 7.3 + Math.random() * 6) % 100}%`,
  delay: `${(i * 0.6) % 8}s`,
  duration: `${11 + ((i * 3) % 9)}s`,
  size: 6 + ((i * 5) % 12),
  drift: `${(i % 2 === 0 ? 1 : -1) * (20 + (i % 3) * 10)}px`
}))

const heroStats = [
  { value: '6', label: 'cores oficiais' },
  { value: '2', label: 'famílias tipográficas' },
  { value: '5', label: 'versões de logotipo' }
]

/* ═════════════════════ 3. construção geométrica (arrastável) ═════════════════════ */
const constructionStep = ref(0)
const constructionMax = 4
const constructionLabels = [
  'Guia circular de base',
  'Traço do monograma R',
  'Traço do monograma B',
  'Espaçamento e proporção final'
]
const playing = ref(false)
const direction = ref(1)
const speed = ref(1)
const speedOptions = [
  { id: 0.5, label: '0.5×' },
  { id: 1, label: '1×' },
  { id: 2, label: '2×' }
]
let constructionAutoTimer = null
const constructionTrackEl = ref(null)

function playConstruction() {
  stopConstruction()
  playing.value = true
  if (constructionStep.value >= constructionMax && direction.value === 1) constructionStep.value = 0
  if (constructionStep.value <= 0 && direction.value === -1) constructionStep.value = constructionMax
  constructionAutoTimer = setInterval(() => {
    const next = constructionStep.value + direction.value
    if (next < 0 || next > constructionMax) return stopConstruction()
    constructionStep.value = next
  }, 900 / speed.value)
}
function stopConstruction() {
  if (constructionAutoTimer) clearInterval(constructionAutoTimer)
  constructionAutoTimer = null
  playing.value = false
}
function toggleDirection() {
  direction.value *= -1
  if (playing.value) playConstruction()
}
function resetConstruction() {
  stopConstruction()
  constructionStep.value = 0
}
function updateStepFromPointer(e) {
  if (!constructionTrackEl.value) return
  const rect = constructionTrackEl.value.getBoundingClientRect()
  const ratio = Math.min(1, Math.max(0, (e.clientX - rect.left) / rect.width))
  constructionStep.value = Math.round(ratio * constructionMax)
}
function onTrackPointerDown(e) {
  stopConstruction()
  updateStepFromPointer(e)
  window.addEventListener('pointermove', updateStepFromPointer)
  window.addEventListener('pointerup', onTrackPointerUp)
}
function onTrackPointerUp() {
  window.removeEventListener('pointermove', updateStepFromPointer)
  window.removeEventListener('pointerup', onTrackPointerUp)
}

/* ═════════════════════ 4. laboratório de fundo (versões) ═════════════════════ */
const logoVariants = [
  { id: 'principal', label: 'Principal (horizontal)', file: 'logoname1.svg', use: 'Aplicação padrão: site, papelaria, assinatura de e-mail.' },
  { id: 'vertical', label: 'Vertical', file: 'logoC1.svg', use: 'Espaços quadrados ou estreitos: redes sociais, carimbos.' },
  { id: 'simbolo', label: 'Símbolo isolado', file: 'logo3final.svg', use: 'Favicon, marca d\u2019água, avatar, espaços muito reduzidos.' },
  { id: 'mono', label: 'Monocromática', file: 'logoG1.svg', use: 'Impressão em 1 cor, fax, gravação a laser, chancela.' }
]
const activeVariant = ref(logoVariants[0])

const bgPresets = [
  { id: 'sand', hex: '#F3EFE4', label: 'Areia' },
  { id: 'moss', hex: '#12190F', label: 'Moss' },
  { id: 'dark', hex: '#0A0A08', label: 'Escuro' },
  { id: 'gold', hex: '#D9B673', label: 'Dourado' }
]
const activeBgId = ref('moss')
const customColor = ref('#12190F')

const stageHex = computed(() => {
  if (activeBgId.value === 'custom') return customColor.value
  return bgPresets.find((p) => p.id === activeBgId.value)?.hex ?? '#12190F'
})

function selectPreset(id) {
  activeBgId.value = id
}
function onCustomColorInput(e) {
  customColor.value = e.target.value
  activeBgId.value = 'custom'
}
function useAsBackground(hex) {
  customColor.value = hex
  activeBgId.value = 'custom'
  scrollToSection('versoes')
}

/* WCAG relative luminance / contraste */
function hexToRgb(hex) {
  const clean = hex.replace('#', '')
  const full = clean.length === 3 ? clean.split('').map((c) => c + c).join('') : clean
  const num = parseInt(full, 16)
  return { r: (num >> 16) & 255, g: (num >> 8) & 255, b: num & 255 }
}
function relativeLuminance(hex) {
  const { r, g, b } = hexToRgb(hex)
  const chan = [r, g, b].map((v) => {
    const c = v / 255
    return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4)
  })
  return 0.2126 * chan[0] + 0.7152 * chan[1] + 0.0722 * chan[2]
}
function contrastRatio(hexA, hexB) {
  const lA = relativeLuminance(hexA) + 0.05
  const lB = relativeLuminance(hexB) + 0.05
  return lA > lB ? lA / lB : lB / lA
}

const logoIsLightOnStage = computed(() => relativeLuminance(stageHex.value) < 0.4)
const contrastAgainstLogo = computed(() => {
  const logoHex = logoIsLightOnStage.value ? '#F3EFE4' : '#1B2A1A'
  return contrastRatio(stageHex.value, logoHex)
})
const contrastLabel = computed(() => {
  const ratio = contrastAgainstLogo.value
  if (ratio >= 4.5) return { pass: true, text: `${ratio.toFixed(1)}:1 · AA` }
  if (ratio >= 3) return { pass: 'partial', text: `${ratio.toFixed(1)}:1 · só p/ texto grande` }
  return { pass: false, text: `${ratio.toFixed(1)}:1 · baixo contraste` }
})

/* ═════════════════════ 5/6 · proteção e tamanho mínimo ═════════════════════ */
const showGrid = ref(false)
const minSizePx = ref(96)
const minSizeWarning = computed(() => minSizePx.value < 32)

/* ═════════════════════ 7. paleta de cores ═════════════════════ */
const palette = [
  { name: 'Moss 950', hex: '#12190F', rgb: '18, 25, 15', use: 'Fundos escuros, overlays' },
  { name: 'Moss 600', hex: '#3D5A3E', rgb: '61, 90, 62', use: 'Superfícies moss médias' },
  { name: 'Gold 300', hex: '#D9B673', rgb: '217, 182, 115', use: 'Destaques, ícones, CTAs' },
  { name: 'Gold 700', hex: '#8A6A34', rgb: '138, 106, 52', use: 'Sombra do gradiente dourado' },
  { name: 'Sand 100', hex: '#F3EFE4', rgb: '243, 239, 228', use: 'Fundos claros, texto sobre moss' },
  { name: 'Ink 900', hex: '#1B2A1A', rgb: '27, 42, 26', use: 'Texto principal sobre fundo claro' }
]
const gradients = [
  { name: 'Aurora do campo', css: 'linear-gradient(135deg, #12190F 0%, #3D5A3E 55%, #D9B673 100%)' },
  { name: 'Amanhecer dourado', css: 'linear-gradient(135deg, #8A6A34 0%, #D9B673 60%, #F3EFE4 100%)' }
]
const copiedHex = ref(null)
async function copyHex(hex) {
  try {
    await navigator.clipboard.writeText(hex)
    copiedHex.value = hex
    setTimeout(() => { if (copiedHex.value === hex) copiedHex.value = null }, 1600)
  } catch {
    /* clipboard indisponível — falha silenciosa, o valor já está visível na tela */
  }
}

/* ═════════════════════ 8. tipografia ═════════════════════ */
const typeWeight = ref(600)

/* ═════════════════════ 9. usos incorretos ═════════════════════ */
const misuses = [
  { title: 'Não distorcer', reason: 'Esticar ou achatar quebra a proporção do círculo-guia e desalinha o monograma.' },
  { title: 'Não recolorir livremente', reason: 'Use apenas as cores da paleta oficial — combinações fora dela enfraquecem o reconhecimento da marca.' },
  { title: 'Não rotacionar', reason: 'O símbolo tem um eixo vertical intencional (o caule da espiga); girar quebra essa leitura.' },
  { title: 'Não aplicar sombra ou brilho', reason: 'Efeitos 3D contradizem o traço fino e plano que define a identidade.' },
  { title: 'Não usar sobre fundo de baixo contraste', reason: 'A versão dourada precisa de pelo menos 4.5:1 de contraste para permanecer legível.' },
  { title: 'Não alterar o espaçamento interno', reason: 'A distância entre símbolo e wordmark segue a grade de construção — não aproxime nem afaste manualmente.' }
]
const flippedCard = ref(null)

/* ═════════════════════ 11. arquivos ═════════════════════ */
const deliverables = [
  { name: 'logoname1.svg', type: 'SVG', desc: 'Vetor, lockup horizontal completo' },
  { name: 'logoC1.svg', type: 'SVG', desc: 'Vetor, versão empilhada' },
  { name: 'logo3final.svg', type: 'SVG', desc: 'Vetor, símbolo isolado' },
  { name: 'logoG1.svg', type: 'SVG', desc: 'Vetor, 1 cor (currentColor)' },
  { name: 'LogoBrancaOficial.svg', type: 'SVG', desc: 'Versão branca para fundos escuros' },
  { name: 'logo1final.svg / logo2final.svg', type: 'SVG', desc: 'Camadas de construção do símbolo' }
]
const copiedFile = ref(null)
async function copyFilename(name) {
  try {
    await navigator.clipboard.writeText(name)
    copiedFile.value = name
    setTimeout(() => { if (copiedFile.value === name) copiedFile.value = null }, 1600)
  } catch {
    /* falha silenciosa */
  }
}

/* ═════════════════════ lifecycle ═════════════════════ */
onMounted(() => {
  motionQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
  prefersReducedMotion.value = motionQuery.matches
  motionQuery.addEventListener?.('change', (e) => (prefersReducedMotion.value = e.matches))

  isFinePointer.value = window.matchMedia('(pointer: fine)').matches

  if (prefersReducedMotion.value) {
    constructionStep.value = constructionMax
    rootEl.value?.querySelectorAll('[data-reveal]').forEach((el) => revealed.add(el.dataset.reveal))
  } else {
    revealObserver = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            revealed.add(entry.target.dataset.reveal)
            revealObserver.unobserve(entry.target)
          }
        })
      },
      { threshold: 0.2, rootMargin: '0px 0px -10% 0px' }
    )
    rootEl.value?.querySelectorAll('[data-reveal]').forEach((el) => revealObserver.observe(el))
  }

  spyObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) activeSection.value = entry.target.id
      })
    },
    { rootMargin: '-35% 0px -55% 0px', threshold: 0 }
  )
  sections.forEach((s) => {
    const el = document.getElementById(s.id)
    if (el) spyObserver.observe(el)
  })

  window.addEventListener('scroll', updateScrollProgress, { passive: true })
  window.addEventListener('pointermove', onWindowPointerMove, { passive: true })
  window.addEventListener('deviceorientation', onHeroTiltDevice)
  updateScrollProgress()
})

onBeforeUnmount(() => {
  revealObserver?.disconnect()
  spyObserver?.disconnect()
  stopConstruction()
  window.removeEventListener('scroll', updateScrollProgress)
  window.removeEventListener('pointermove', onWindowPointerMove)
  window.removeEventListener('deviceorientation', onHeroTiltDevice)
  window.removeEventListener('pointermove', updateStepFromPointer)
  window.removeEventListener('pointerup', onTrackPointerUp)
})
</script>

<template>
  <div ref="rootEl" class="bb" :class="{ 'no-motion': prefersReducedMotion, 'has-cursor': isFinePointer && !prefersReducedMotion }">
    <!-- ══════════ cursor magnético custom ══════════ -->
    <div
      v-if="isFinePointer && !prefersReducedMotion"
      class="bb-cursor"
      :class="{ 'is-hover': cursor.hover, 'is-active': cursor.active }"
      :style="{ transform: `translate3d(${cursor.x}px, ${cursor.y}px, 0)` }"
      aria-hidden="true"
    />

    <!-- ══════════ barra de progresso de leitura ══════════ -->
    <div class="bb-progress" role="progressbar" :aria-valuenow="Math.round(scrollProgress)" aria-valuemin="0" aria-valuemax="100">
      <div class="bb-progress__fill" :style="{ width: scrollProgress + '%' }" />
    </div>

    <!-- ══════════ botão de menu (mobile) ══════════ -->
    <button type="button" class="bb-menu-btn" :class="{ open: mobileNavOpen }" @click="mobileNavOpen = !mobileNavOpen" :aria-expanded="mobileNavOpen" aria-controls="bb-drawer" aria-label="Abrir sumário">
      <span /><span /><span />
    </button>

    <!-- ══════════ gaveta de navegação (mobile) ══════════ -->
    <Transition name="bb-drawer-fade">
      <div v-if="mobileNavOpen" class="bb-drawer-backdrop" @click="mobileNavOpen = false" />
    </Transition>
    <Transition name="bb-drawer-slide">
      <nav v-if="mobileNavOpen" id="bb-drawer" class="bb-drawer" aria-label="Sumário do manual de marca">
        <p class="bb-drawer__brand">{{ brandName }}</p>
        <p class="bb-drawer__sub">{{ brandSuffix }}</p>
        <ul>
          <li v-for="(s, i) in sections" :key="s.id">
            <button type="button" :class="{ active: activeSection === s.id }" @click="scrollToSection(s.id)">
              <span class="bb-drawer__idx">{{ String(i + 1).padStart(2, '0') }}</span>
              {{ s.label }}
            </button>
          </li>
        </ul>
      </nav>
    </Transition>

    <!-- ══════════ nav lateral fixa (desktop) ══════════ -->
    <nav class="bb-nav" aria-label="Sumário do manual de marca">
      <p class="bb-nav__brand">{{ brandName }}</p>
      <p class="bb-nav__brand-sub">Manual de marca</p>
      <ul>
        <li v-for="s in sections" :key="s.id">
          <button type="button" class="magnet" :class="{ active: activeSection === s.id }" @click="scrollToSection(s.id)" @mousemove="onMagnetMove" @mouseleave="onMagnetLeave">
            {{ s.label }}
          </button>
        </li>
      </ul>
      <div class="bb-nav__dots" aria-hidden="true">
        <span v-for="(s, i) in sections" :key="'dot-' + s.id" class="bb-nav__dot" :class="{ active: i <= activeSectionIndex }" />
      </div>
    </nav>

    <main class="bb-main">
      <!-- ══════════ 1. capa ══════════ -->
      <section class="bb-cover" @mousemove="onHeroMove" @mouseleave="onHeroLeave">
        <div class="bb-cover__field" aria-hidden="true">
          <span
            v-for="p in heroParticles"
            :key="p.id"
            class="bb-cover__grain"
            :style="{ left: p.left, animationDelay: p.delay, animationDuration: p.duration, width: p.size + 'px', height: p.size + 'px', '--drift': p.drift }"
          />
        </div>

        <svg
          class="bb-cover__loop"
          viewBox="0 0 200 200"
          width="120"
          height="120"
          aria-hidden="true"
          :style="{ transform: `perspective(600px) rotateX(${-heroTilt.y}deg) rotateY(${heroTilt.x}deg)` }"
        >
          <circle class="bb-cover__ring" cx="100" cy="100" r="90" fill="none" stroke="currentColor" stroke-width="1" pathLength="1" />
          <image :href="asset('LogoBrancaOficial.svg')" x="20" y="20" width="160" height="160" />
        </svg>

        <p class="bb-cover__eyebrow">Manual de identidade visual</p>
        <h1 class="bb-cover__title">{{ brandName }}</h1>
        <p class="bb-cover__subtitle">{{ brandSuffix }} — versões, cores, tipografia e regras de uso da marca.</p>

        <div class="bb-cover__stats">
          <div v-for="s in heroStats" :key="s.label" class="bb-cover__stat">
            <strong>{{ s.value }}</strong>
            <span>{{ s.label }}</span>
          </div>
        </div>

        <button type="button" class="bb-cover__hint magnet" @click="scrollToSection('conceito')" @mousemove="onMagnetMove" @mouseleave="onMagnetLeave">
          <span>role para explorar</span>
          <svg width="14" height="20" viewBox="0 0 14 20" fill="none" aria-hidden="true">
            <rect x="1" y="1" width="12" height="18" rx="6" stroke="currentColor" stroke-width="1" />
            <circle class="bb-cover__hint-dot" cx="7" cy="6" r="2" fill="currentColor" />
          </svg>
        </button>
      </section>

      <!-- ══════════ 2. conceito ══════════ -->
      <section id="conceito" class="bb-section">
        <div class="bb-block" data-reveal="conceito" :class="{ 'is-in': isRevealed('conceito') }">
          <p class="bb-kicker">01 · Conceito</p>
          <h2 class="bb-h2">Duas raízes, um só compromisso</h2>
          <p class="bb-lead">
            O monograma entrelaça as iniciais <strong>R</strong> e <strong>B</strong> como dois traços que se apoiam —
            metáfora para uma advocacia que sustenta o cliente tanto no contencioso quanto na consultoria preventiva.
            A espiga de trigo, ancorada no eixo central, situa a marca fisicamente no campo: cada detalhe existe para
            lembrar que direito e agronegócio crescem da mesma terra.
          </p>
          <blockquote class="bb-quote">"Uma marca que protege limites — de terra e de direito."</blockquote>
        </div>
      </section>

      <!-- ══════════ 3. construção geométrica ══════════ -->
      <section id="construcao" class="bb-section">
        <div class="bb-block" data-reveal="construcao" :class="{ 'is-in': isRevealed('construcao') }">
          <p class="bb-kicker">02 · Construção</p>
          <h2 class="bb-h2">Como o símbolo é montado</h2>
          <p class="bb-lead">Arraste diretamente na régua abaixo ou use o play para ver as quatro camadas que formam o símbolo.</p>

          <div class="bb-construction">
            <svg viewBox="0 0 200 200" class="bb-construction__art" aria-hidden="true">
              <circle v-if="constructionStep >= 1" cx="100" cy="100" r="92" fill="none" stroke="var(--bb-gold)" stroke-width="0.75" stroke-dasharray="2 3" opacity="0.6" class="bb-construction__guide" />
              <g v-if="constructionStep >= 2" fill="none" stroke="var(--bb-ink)" stroke-width="3.2" stroke-linecap="round" stroke-linejoin="round" class="bb-construction__layer">
                <image :href="asset('logo1final.svg')" x="20" y="20" width="160" height="160" />
              </g>
              <g v-if="constructionStep >= 3" stroke="var(--bb-gold-dark)" stroke-width="2" fill="none" stroke-linecap="round" class="bb-construction__layer">
                <image :href="asset('logo2final.svg')" x="20" y="20" width="160" height="160" />
              </g>
              <g v-if="constructionStep >= 4" class="bb-construction__layer bb-construction__layer--final">
                <image :href="asset('logo3final.svg')" x="20" y="20" width="160" height="160" />
              </g>
            </svg>

            <div class="bb-construction__controls">
              <div
                ref="constructionTrackEl"
                class="bb-construction__track"
                role="slider"
                tabindex="0"
                :aria-valuemin="0"
                :aria-valuemax="constructionMax"
                :aria-valuenow="constructionStep"
                aria-label="Etapa de construção do símbolo"
                @pointerdown="onTrackPointerDown"
                @keydown.right="constructionStep = Math.min(constructionMax, constructionStep + 1)"
                @keydown.left="constructionStep = Math.max(0, constructionStep - 1)"
              >
                <div class="bb-construction__track-fill" :style="{ width: (constructionStep / constructionMax) * 100 + '%' }" />
                <button
                  v-for="n in constructionMax + 1"
                  :key="n"
                  type="button"
                  class="bb-construction__stop"
                  :class="{ active: constructionStep >= n - 1 }"
                  :style="{ left: ((n - 1) / constructionMax) * 100 + '%' }"
                  @click.stop="constructionStep = n - 1"
                  :aria-label="`Ir para etapa ${n - 1}`"
                />
                <div class="bb-construction__handle" :style="{ left: (constructionStep / constructionMax) * 100 + '%' }" />
              </div>

              <div class="bb-construction__row">
                <div class="bb-construction__buttons">
                  <button type="button" class="bb-btn bb-btn--icon magnet" @click="resetConstruction" @mousemove="onMagnetMove" @mouseleave="onMagnetLeave" aria-label="Reiniciar">↺</button>
                  <button type="button" class="bb-btn magnet" @click="playing ? stopConstruction() : playConstruction()" @mousemove="onMagnetMove" @mouseleave="onMagnetLeave">
                    {{ playing ? '❙❙ Pausar' : '▶ Reproduzir' }}
                  </button>
                  <button type="button" class="bb-btn bb-btn--icon magnet" @click="toggleDirection" @mousemove="onMagnetMove" @mouseleave="onMagnetLeave" :aria-label="direction === 1 ? 'Inverter para trás' : 'Inverter para frente'">
                    {{ direction === 1 ? '→' : '←' }}
                  </button>
                </div>
                <div class="bb-construction__speed" role="group" aria-label="Velocidade">
                  <button v-for="opt in speedOptions" :key="opt.id" type="button" :class="{ active: speed === opt.id }" @click="speed = opt.id; if (playing) playConstruction()">
                    {{ opt.label }}
                  </button>
                </div>
              </div>
              <p class="bb-construction__label">
                <span class="bb-construction__label-step">Etapa {{ constructionStep }}/{{ constructionMax }}</span>
                {{ constructionStep === 0 ? 'Estado inicial' : constructionLabels[constructionStep - 1] }}
              </p>
            </div>
          </div>
        </div>
      </section>

      <!-- ══════════ 4. laboratório de fundo / versões ══════════ -->
      <section id="versoes" class="bb-section">
        <div class="bb-block bb-block--wide" data-reveal="versoes" :class="{ 'is-in': isRevealed('versoes') }">
          <p class="bb-kicker">03 · Laboratório de fundo</p>
          <h2 class="bb-h2">Teste a marca em qualquer cor</h2>
          <p class="bb-lead">Escolha um preset, arraste na roda de cores ou cole um hex — o logotipo troca de claro para escuro sozinho, e o contraste é medido ao vivo.</p>

          <div class="bb-versions">
            <div class="bb-versions__stage" :style="{ background: stageHex }" :class="{ 'is-light-logo': logoIsLightOnStage }">
              <span class="bb-versions__contrast" :class="contrastLabel.pass === true ? 'pass' : contrastLabel.pass === 'partial' ? 'partial' : 'fail'">
                {{ contrastLabel.text }}
              </span>
              <img :src="asset(activeVariant.file)" :alt="activeVariant.label" class="bb-versions__logo" />

              <div class="bb-versions__bgpicker">
                <button
                  v-for="opt in bgPresets"
                  :key="opt.id"
                  type="button"
                  class="bb-versions__swatchbtn"
                  :class="{ active: activeBgId === opt.id }"
                  :style="{ background: opt.hex }"
                  @click="selectPreset(opt.id)"
                  :aria-label="opt.label"
                  :title="opt.label"
                />
                <label class="bb-versions__custom" title="Cor personalizada">
                  <input type="color" :value="customColor" @input="onCustomColorInput" aria-label="Escolher cor de fundo personalizada" />
                  <span :class="{ active: activeBgId === 'custom' }">+</span>
                </label>
              </div>
            </div>

            <ul class="bb-versions__list">
              <li v-for="v in logoVariants" :key="v.id">
                <button type="button" class="bb-versions__item" :class="{ active: activeVariant.id === v.id }" @click="activeVariant = v">
                  <img :src="asset(v.file)" :alt="v.label" />
                  <div>
                    <strong>{{ v.label }}</strong>
                    <p>{{ v.use }}</p>
                  </div>
                </button>
              </li>
            </ul>
          </div>
        </div>
      </section>

      <!-- ══════════ 5. área de proteção ══════════ -->
      <section id="protecao" class="bb-section">
        <div class="bb-block" data-reveal="protecao" :class="{ 'is-in': isRevealed('protecao') }">
          <p class="bb-kicker">04 · Área de proteção</p>
          <h2 class="bb-h2">Espaço mínimo ao redor</h2>
          <p class="bb-lead">
            A unidade de medida é <strong>x</strong> = a largura de uma folha da espiga. Nenhum elemento — texto, imagem
            ou borda — deve invadir esse espaço.
          </p>
          <label class="bb-toggle">
            <input type="checkbox" v-model="showGrid" />
            <span>Mostrar grade de unidades</span>
          </label>
          <div class="bb-clearspace" :class="{ 'is-in': isRevealed('protecao') }">
            <svg viewBox="0 0 260 260" width="220" height="220" aria-hidden="true">
              <rect x="10" y="10" width="240" height="240" fill="none" stroke="var(--bb-gold)" stroke-width="1" stroke-dasharray="3 4" class="bb-clearspace__box" />
              <g v-if="showGrid" stroke="var(--bb-gold)" stroke-width="0.4" opacity="0.35">
                <line v-for="n in 5" :key="'v' + n" :x1="10 + n * 40" y1="10" :x2="10 + n * 40" y2="250" />
                <line v-for="n in 5" :key="'h' + n" x1="10" :y1="10 + n * 40" x2="250" :y2="10 + n * 40" />
              </g>
              <image :href="asset('logo3final.svg')" x="55" y="55" width="150" height="150" />
              <line x1="10" y1="130" x2="55" y2="130" stroke="var(--bb-gold-dark)" stroke-width="1" />
              <text x="30" y="124" text-anchor="middle" font-size="11" fill="var(--bb-gold-dark)" font-family="Inter, sans-serif">x</text>
            </svg>
          </div>
        </div>
      </section>

      <!-- ══════════ 6. tamanho mínimo ══════════ -->
      <section id="tamanho" class="bb-section">
        <div class="bb-block" data-reveal="tamanho" :class="{ 'is-in': isRevealed('tamanho') }">
          <p class="bb-kicker">05 · Tamanho mínimo</p>
          <h2 class="bb-h2">Arraste até o ponto de quebra</h2>
          <p class="bb-lead">Reduza o símbolo e veja em que ponto ele deixa de ser legível.</p>

          <div class="bb-minsize">
            <div class="bb-minsize__preview" :class="{ warn: minSizeWarning }">
              <img :src="asset('logo3final.svg')" :style="{ width: minSizePx + 'px', height: minSizePx + 'px' }" alt="Pré-visualização de tamanho" />
            </div>
            <input type="range" min="16" max="200" step="2" v-model.number="minSizePx" aria-label="Tamanho do símbolo em pixels" />
            <p class="bb-minsize__readout">
              <strong>{{ minSizePx }}px</strong>
              <span v-if="minSizeWarning" class="bb-minsize__warning">⚠ abaixo de 32px o traço se perde — não use.</span>
              <span v-else>legível — acima do mínimo recomendado de 32px digital.</span>
            </p>
          </div>
        </div>
      </section>

      <!-- ══════════ 7. cores ══════════ -->
      <section id="cores" class="bb-section">
        <div class="bb-block" data-reveal="cores" :class="{ 'is-in': isRevealed('cores') }">
          <p class="bb-kicker">06 · Cores</p>
          <h2 class="bb-h2">Paleta oficial</h2>
          <div class="bb-palette">
            <div v-for="c in palette" :key="c.hex" class="bb-swatch">
              <button type="button" class="bb-swatch__color" :style="{ background: c.hex }" @click="copyHex(c.hex)" :aria-label="`Copiar ${c.hex}`" />
              <span class="bb-swatch__meta">
                <strong>{{ c.name }}</strong>
                <span>{{ copiedHex === c.hex ? 'copiado!' : c.hex }}</span>
                <small>{{ c.use }}</small>
                <button type="button" class="bb-swatch__use magnet" @click="useAsBackground(c.hex)" @mousemove="onMagnetMove" @mouseleave="onMagnetLeave">usar como fundo →</button>
              </span>
            </div>
          </div>

          <p class="bb-kicker" style="margin-top: 2.5rem">Combinações em gradiente</p>
          <div class="bb-gradients">
            <button v-for="g in gradients" :key="g.name" type="button" class="bb-gradient" :style="{ background: g.css }" @click="useAsBackground('#12190F')">
              <span>{{ g.name }}</span>
            </button>
          </div>
        </div>
      </section>

      <!-- ══════════ 8. tipografia ══════════ -->
      <section id="tipografia" class="bb-section">
        <div class="bb-block" data-reveal="tipografia" :class="{ 'is-in': isRevealed('tipografia') }">
          <p class="bb-kicker">07 · Tipografia</p>
          <h2 class="bb-h2">Duas vozes, um par</h2>
          <div class="bb-type">
            <div class="bb-type__card">
              <p class="bb-type__label">Display — Cormorant Garamond</p>
              <p class="bb-type__sample bb-type__sample--serif">Cultivando soluções</p>
              <p class="bb-type__use">Títulos, citações, headlines. Peso 600.</p>
            </div>
            <div class="bb-type__card">
              <p class="bb-type__label">Texto — Inter</p>
              <p class="bb-type__sample bb-type__sample--sans" :style="{ fontWeight: typeWeight }">Direito do agronegócio</p>
              <p class="bb-type__use">Corpo de texto, rótulos, botões.</p>
              <input type="range" min="400" max="700" step="100" v-model.number="typeWeight" aria-label="Peso da fonte" />
              <p class="bb-type__weight">Peso {{ typeWeight }}</p>
            </div>
          </div>

          <div class="bb-type__pairing">
            <p class="bb-kicker" style="margin-top: 0">Pareamento em contexto</p>
            <h3 class="bb-type__pairing-title">Consultoria que antecipa o litígio</h3>
            <p class="bb-type__pairing-body">Assessoria jurídica especializada em agronegócio, do contrato de arrendamento à demarcação de terras — texto corrido em Inter, sempre com respiro generoso entre linhas.</p>
          </div>
        </div>
      </section>

      <!-- ══════════ 9. usos incorretos ══════════ -->
      <section id="incorretos" class="bb-section">
        <div class="bb-block" data-reveal="incorretos" :class="{ 'is-in': isRevealed('incorretos') }">
          <p class="bb-kicker">08 · Usos incorretos</p>
          <h2 class="bb-h2">Toque ou passe o mouse para entender o motivo</h2>
          <div class="bb-misuse-grid">
            <div
              v-for="(m, i) in misuses"
              :key="m.title"
              class="bb-flip"
              :class="{ flipped: flippedCard === i }"
              @mouseenter="flippedCard = i"
              @mouseleave="flippedCard = null"
              @click="flippedCard = flippedCard === i ? null : i"
              tabindex="0"
              @keydown.enter="flippedCard = flippedCard === i ? null : i"
            >
              <div class="bb-flip__inner">
                <div class="bb-flip__front">
                  <span class="bb-flip__x">×</span>
                  <p>{{ m.title }}</p>
                  <span class="bb-flip__tap">toque para ver o motivo</span>
                </div>
                <div class="bb-flip__back">
                  <p>{{ m.reason }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- ══════════ 10. aplicações ══════════ -->
      <section id="aplicacoes" class="bb-section">
        <div class="bb-block" data-reveal="aplicacoes" :class="{ 'is-in': isRevealed('aplicacoes') }">
          <p class="bb-kicker">09 · Aplicações</p>
          <h2 class="bb-h2">A marca em uso</h2>
          <p class="bb-lead">Os mockups abaixo usam a mesma cor escolhida no laboratório de fundo.</p>
          <div class="bb-mockups">
            <div class="bb-mock bb-mock--card" :style="{ background: stageHex }" :class="{ 'is-light-logo': logoIsLightOnStage }">
              <img :src="asset('LogoBrancaOficial.svg')" alt="" />
              <div>
                <strong>{{ brandName }}</strong>
                <small>{{ brandSuffix }}</small>
              </div>
            </div>
            <div class="bb-mock bb-mock--avatar">
              <img :src="asset('LogoBrancaOficial.svg')" alt="" />
            </div>
            <div class="bb-mock bb-mock--doc">
              <img :src="asset('logo3final.svg')" alt="" />
              <div class="bb-mock__lines">
                <span /><span /><span style="width: 60%" />
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- ══════════ 11. arquivos ══════════ -->
      <section id="arquivos" class="bb-section bb-section--last">
        <div class="bb-block" data-reveal="arquivos" :class="{ 'is-in': isRevealed('arquivos') }">
          <p class="bb-kicker">10 · Arquivos</p>
          <h2 class="bb-h2">O que compõe este pacote</h2>
          <p class="bb-lead">Todos os arquivos desta entrega. Clique no nome para copiar.</p>
          <div class="bb-files-wrap">
            <table class="bb-files">
              <thead>
                <tr><th>Arquivo</th><th>Tipo</th><th>Uso</th></tr>
              </thead>
              <tbody>
                <tr v-for="f in deliverables" :key="f.name">
                  <td><button type="button" class="bb-files__name" @click="copyFilename(f.name)"><code>{{ copiedFile === f.name ? 'copiado!' : f.name }}</code></button></td>
                  <td><span class="bb-tag">{{ f.type }}</span></td>
                  <td>{{ f.desc }}</td>
                </tr>
              </tbody>
            </table>
          </div>
          <p class="bb-footer">{{ brandName }} · {{ brandSuffix }} — manual de marca</p>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.bb {
  --bb-moss: #12190f;
  --bb-gold: #d9b673;
  --bb-gold-dark: #8a6a34;
  --bb-sand: #f3efe4;
  --bb-ink: #1b2a1a;
  --bb-ink-soft: #445042;

  position: relative;
  display: grid;
  grid-template-columns: 1fr;
  background: var(--bb-sand);
  color: var(--bb-ink);
  font-family: 'Inter', system-ui, sans-serif;
  overflow-x: clip;
}
@media (min-width: 960px) {
  .bb { grid-template-columns: 15rem 1fr; }
}
.bb.has-cursor, .bb.has-cursor * { cursor: none; }

/* ---------- cursor custom ---------- */
.bb-cursor {
  position: fixed;
  top: 0; left: 0;
  width: 14px; height: 14px;
  margin: -7px 0 0 -7px;
  border-radius: 50%;
  background: var(--bb-gold);
  pointer-events: none;
  z-index: 999;
  mix-blend-mode: difference;
  transition: width 0.25s ease, height 0.25s ease, margin 0.25s ease, background 0.25s ease;
  will-change: transform;
}
.bb-cursor.is-hover { width: 40px; height: 40px; margin: -20px 0 0 -20px; background: var(--bb-gold-dark); }

/* ---------- barra de progresso ---------- */
.bb-progress {
  position: fixed;
  top: 0; left: 0; right: 0;
  height: 3px;
  z-index: 60;
  background: rgba(27, 42, 26, 0.08);
}
.bb-progress__fill {
  height: 100%;
  background: linear-gradient(90deg, var(--bb-gold-dark), var(--bb-gold));
  transition: width 0.1s linear;
}

/* ---------- botão de menu mobile ---------- */
.bb-menu-btn {
  position: fixed;
  top: 1rem; right: 1rem;
  z-index: 70;
  width: 44px; height: 44px;
  border-radius: 50%;
  border: none;
  background: var(--bb-moss);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 4px;
  cursor: pointer;
  box-shadow: 0 4px 14px rgba(18, 25, 15, 0.35);
}
@media (min-width: 960px) { .bb-menu-btn { display: none; } }
.bb-menu-btn span { width: 18px; height: 2px; background: var(--bb-gold); transition: transform 0.3s ease, opacity 0.3s ease; }
.bb-menu-btn.open span:nth-child(1) { transform: translateY(6px) rotate(45deg); }
.bb-menu-btn.open span:nth-child(2) { opacity: 0; }
.bb-menu-btn.open span:nth-child(3) { transform: translateY(-6px) rotate(-45deg); }

/* ---------- gaveta mobile ---------- */
.bb-drawer-backdrop { position: fixed; inset: 0; z-index: 65; background: rgba(10, 10, 8, 0.55); backdrop-filter: blur(2px); }
.bb-drawer-fade-enter-active, .bb-drawer-fade-leave-active { transition: opacity 0.25s ease; }
.bb-drawer-fade-enter-from, .bb-drawer-fade-leave-to { opacity: 0; }

.bb-drawer {
  position: fixed;
  top: 0; right: 0;
  z-index: 68;
  width: min(80vw, 20rem);
  height: 100%;
  background: var(--bb-sand);
  padding: 4.5rem 1.5rem 2rem;
  overflow-y: auto;
  box-shadow: -8px 0 30px rgba(18, 25, 15, 0.25);
}
.bb-drawer-slide-enter-active, .bb-drawer-slide-leave-active { transition: transform 0.32s cubic-bezier(0.22, 1, 0.36, 1); }
.bb-drawer-slide-enter-from, .bb-drawer-slide-leave-to { transform: translateX(100%); }
.bb-drawer__brand { font-family: 'Cormorant Garamond', serif; font-size: 1.3rem; font-weight: 600; margin: 0; }
.bb-drawer__sub { font-size: 0.75rem; color: var(--bb-gold-dark); text-transform: uppercase; letter-spacing: 0.1em; margin: 0.2rem 0 1.6rem; }
.bb-drawer ul { list-style: none; margin: 0; padding: 0; display: flex; flex-direction: column; gap: 0.2rem; }
.bb-drawer button {
  all: unset; cursor: pointer; display: flex; align-items: center; gap: 0.7rem;
  width: 100%; padding: 0.7rem 0.5rem; border-radius: 8px; font-size: 0.95rem; color: var(--bb-ink-soft);
}
.bb-drawer button.active { background: rgba(217, 182, 115, 0.22); color: var(--bb-ink); font-weight: 600; }
.bb-drawer__idx { font-size: 0.7rem; color: var(--bb-gold-dark); font-family: monospace; }

/* ---------- nav desktop ---------- */
.bb-nav {
  display: none;
  position: sticky;
  top: 0;
  align-self: start;
  height: 100svh;
  padding: 2.5rem 1.5rem;
  border-right: 0.5px solid rgba(27, 42, 26, 0.12);
  overflow-y: auto;
}
@media (min-width: 960px) { .bb-nav { display: flex; flex-direction: column; } }
.bb-nav__brand { font-family: 'Cormorant Garamond', serif; font-size: 1.3rem; font-weight: 600; margin: 0; }
.bb-nav__brand-sub { font-size: 0.72rem; letter-spacing: 0.12em; text-transform: uppercase; color: var(--bb-gold-dark); margin: 0.2rem 0 1.8rem; }
.bb-nav ul { list-style: none; margin: 0; padding: 0; display: flex; flex-direction: column; gap: 0.1rem; }
.bb-nav button {
  all: unset; cursor: pointer; display: block; width: 100%; padding: 0.45rem 0.6rem;
  font-size: 0.85rem; border-radius: 6px; color: var(--bb-ink-soft);
  transition: background 0.2s ease, color 0.2s ease, transform 0.15s ease;
  transform: translate(var(--mx, 0), var(--my, 0));
}
.bb-nav button:hover { background: rgba(217, 182, 115, 0.15); }
.bb-nav button.active { background: rgba(217, 182, 115, 0.25); color: var(--bb-ink); font-weight: 600; }
.bb-nav__dots { margin-top: auto; display: flex; flex-direction: column; gap: 0.4rem; padding-top: 1rem; }
.bb-nav__dot { width: 6px; height: 6px; border-radius: 50%; background: rgba(27, 42, 26, 0.15); transition: background 0.3s ease; }
.bb-nav__dot.active { background: var(--bb-gold-dark); }

/* ---------- capa ---------- */
.bb-cover {
  position: relative;
  min-height: 100svh;
  display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center;
  padding: 3rem 1.5rem;
  background: radial-gradient(circle at 50% 30%, #1b2a1a 0%, var(--bb-moss) 70%);
  color: var(--bb-sand);
  overflow: hidden;
}
.bb-cover__field { position: absolute; inset: 0; pointer-events: none; }
.bb-cover__grain {
  position: absolute;
  bottom: -5%;
  border-radius: 2px 2px 6px 6px;
  background: linear-gradient(180deg, var(--bb-gold) 0%, var(--bb-gold-dark) 100%);
  opacity: 0.35;
  animation: bb-drift linear infinite;
}
@keyframes bb-drift {
  0% { transform: translateY(0) translateX(0) rotate(0deg); opacity: 0; }
  10% { opacity: 0.35; }
  90% { opacity: 0.25; }
  100% { transform: translateY(-115svh) translateX(var(--drift)) rotate(25deg); opacity: 0; }
}
.bb-cover__loop { color: var(--bb-gold); margin-bottom: 1.5rem; transition: transform 0.15s ease-out; transform-style: preserve-3d; }
.bb-cover__ring { stroke-dasharray: 1; stroke-dashoffset: 1; animation: bb-draw 3s ease-in-out infinite alternate; }
.bb-cover__eyebrow { font-size: 0.72rem; letter-spacing: 0.24em; text-transform: uppercase; color: var(--bb-gold); margin: 0 0 0.6rem; position: relative; }
.bb-cover__title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(2.4rem, 6vw, 4rem);
  font-weight: 600;
  margin: 0 0 0.6rem;
  position: relative;
  background: linear-gradient(100deg, var(--bb-sand) 30%, var(--bb-gold) 50%, var(--bb-sand) 70%);
  background-size: 220% 100%;
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  animation: bb-sheen 7s ease-in-out infinite;
}
@keyframes bb-sheen { 0%, 100% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } }
.bb-cover__subtitle { max-width: 30rem; opacity: 0.85; margin: 0 0 2rem; position: relative; }
.bb-cover__stats { display: flex; gap: clamp(1.2rem, 4vw, 2.5rem); margin-bottom: 2.4rem; position: relative; }
.bb-cover__stat { display: flex; flex-direction: column; align-items: center; }
.bb-cover__stat strong { font-family: 'Cormorant Garamond', serif; font-size: 1.8rem; color: var(--bb-gold); line-height: 1; }
.bb-cover__stat span { font-size: 0.7rem; text-transform: uppercase; letter-spacing: 0.08em; opacity: 0.7; margin-top: 0.3rem; }
.bb-cover__hint {
  all: unset; cursor: pointer; display: flex; flex-direction: column; align-items: center; gap: 0.5rem;
  font-size: 0.8rem; opacity: 0.7; transform: translate(var(--mx, 0), var(--my, 0)); transition: transform 0.15s ease, opacity 0.2s ease;
}
.bb-cover__hint:hover { opacity: 1; }
.bb-cover__hint-dot { animation: bb-scroll-dot 1.8s ease-in-out infinite; }
@keyframes bb-scroll-dot { 0%, 100% { transform: translateY(0); opacity: 1; } 50% { transform: translateY(6px); opacity: 0.3; } }

/* ---------- estrutura geral de seção ---------- */
.bb-main { min-width: 0; }
.bb-section { padding: 5rem clamp(1.25rem, 5vw, 4rem); border-bottom: 0.5px solid rgba(27, 42, 26, 0.1); }
.bb-section--last { border-bottom: none; }
.bb-block {
  max-width: 46rem;
  opacity: 0;
  transform: translateY(18px);
  transition: opacity 0.7s ease, transform 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}
.bb-block--wide { max-width: 56rem; }
.bb-block.is-in { opacity: 1; transform: none; }
.bb-kicker { font-size: 0.72rem; font-weight: 600; letter-spacing: 0.2em; text-transform: uppercase; color: var(--bb-gold-dark); margin: 0 0 0.6rem; }
.bb-h2 { font-family: 'Cormorant Garamond', serif; font-size: clamp(1.7rem, 3vw, 2.3rem); font-weight: 600; margin: 0 0 1rem; }
.bb-lead { line-height: 1.65; color: var(--bb-ink-soft); margin: 0 0 1.5rem; }
.bb-quote { font-family: 'Cormorant Garamond', serif; font-size: 1.4rem; font-style: italic; border-left: 2px solid var(--bb-gold); padding-left: 1rem; margin: 0; color: var(--bb-ink); }

/* ---------- toggle genérico ---------- */
.bb-toggle { display: inline-flex; align-items: center; gap: 0.6rem; font-size: 0.85rem; color: var(--bb-ink-soft); margin-bottom: 1.5rem; cursor: pointer; }
.bb-toggle input { accent-color: var(--bb-gold-dark); }

/* ---------- construção ---------- */
.bb-construction { display: flex; flex-direction: column; align-items: center; gap: 1.4rem; }
.bb-construction__art { width: 220px; height: 220px; }
.bb-construction__guide { animation: bb-draw 1.4s ease-out; }
.bb-construction__layer { animation: bb-layer-in 0.5s cubic-bezier(0.22, 1, 0.36, 1); }
@keyframes bb-layer-in { from { opacity: 0; transform: scale(0.92); } to { opacity: 1; transform: scale(1); } }
.bb-construction__controls { width: 100%; max-width: 380px; display: flex; flex-direction: column; gap: 0.9rem; }

.bb-construction__track {
  position: relative; height: 28px; display: flex; align-items: center; touch-action: none; cursor: pointer;
}
.bb-construction__track::before {
  content: ''; position: absolute; left: 0; right: 0; top: 50%; height: 3px; transform: translateY(-50%);
  background: rgba(27, 42, 26, 0.12); border-radius: 999px;
}
.bb-construction__track-fill {
  position: absolute; left: 0; top: 50%; height: 3px; transform: translateY(-50%);
  background: linear-gradient(90deg, var(--bb-gold-dark), var(--bb-gold)); border-radius: 999px; transition: width 0.2s ease;
}
.bb-construction__stop {
  all: unset; position: absolute; top: 50%; width: 10px; height: 10px; border-radius: 50%;
  transform: translate(-50%, -50%); background: var(--bb-sand); border: 2px solid rgba(27, 42, 26, 0.25); cursor: pointer;
  transition: border-color 0.2s ease, background 0.2s ease;
}
.bb-construction__stop.active { border-color: var(--bb-gold-dark); background: var(--bb-gold); }
.bb-construction__handle {
  position: absolute; top: 50%; width: 16px; height: 16px; border-radius: 50%;
  transform: translate(-50%, -50%); background: var(--bb-gold-dark); box-shadow: 0 2px 8px rgba(138, 106, 52, 0.5);
  transition: left 0.15s ease; pointer-events: none;
}
.bb-construction__row { display: flex; align-items: center; justify-content: space-between; gap: 0.8rem; flex-wrap: wrap; }
.bb-construction__buttons { display: flex; align-items: center; gap: 0.5rem; }
.bb-construction__label { font-size: 0.85rem; color: var(--bb-ink-soft); margin: 0; display: flex; gap: 0.6rem; align-items: baseline; flex-wrap: wrap; }
.bb-construction__label-step { font-family: monospace; font-size: 0.72rem; color: var(--bb-gold-dark); }
.bb-construction__speed { display: flex; gap: 0.3rem; border: 0.5px solid rgba(27, 42, 26, 0.15); border-radius: 999px; padding: 0.15rem; }
.bb-construction__speed button {
  all: unset; cursor: pointer; font-size: 0.72rem; padding: 0.25rem 0.55rem; border-radius: 999px; color: var(--bb-ink-soft);
}
.bb-construction__speed button.active { background: var(--bb-moss); color: var(--bb-sand); }

.bb-btn {
  all: unset; border: 1px solid var(--bb-gold-dark); background: transparent; color: var(--bb-gold-dark);
  padding: 0.5rem 0.9rem; border-radius: 999px; font-size: 0.8rem; font-weight: 600; cursor: pointer;
  transition: background 0.2s ease, color 0.2s ease, transform 0.15s ease;
  transform: translate(var(--mx, 0), var(--my, 0));
}
.bb-btn:hover { background: var(--bb-gold-dark); color: var(--bb-sand); }
.bb-btn--icon { padding: 0.5rem 0.7rem; }

/* ---------- versões / laboratório de fundo ---------- */
.bb-versions { display: grid; gap: 1.5rem; }
@media (min-width: 720px) { .bb-versions { grid-template-columns: 1.2fr 1fr; align-items: start; } }
.bb-versions__stage {
  position: relative; border-radius: 16px; padding: 2.8rem 1.5rem 1.4rem;
  display: flex; flex-direction: column; align-items: center; gap: 1.2rem;
  transition: background 0.35s ease; overflow: hidden;
}
.bb-versions__contrast {
  position: absolute; top: 0.9rem; right: 0.9rem; font-size: 0.68rem; font-weight: 600;
  padding: 0.3rem 0.6rem; border-radius: 999px; letter-spacing: 0.02em;
}
.bb-versions__contrast.pass { background: rgba(61, 90, 62, 0.9); color: #f3efe4; }
.bb-versions__contrast.partial { background: rgba(217, 182, 115, 0.9); color: #1b2a1a; }
.bb-versions__contrast.fail { background: rgba(179, 69, 42, 0.9); color: #f3efe4; }
.bb-versions__logo { max-width: 220px; width: 60%; height: auto; transition: filter 0.3s ease; filter: brightness(0) saturate(100%) invert(9%) sepia(20%) saturate(900%) hue-rotate(60deg); }
.bb-versions__stage.is-light-logo .bb-versions__logo { filter: brightness(0) saturate(100%) invert(96%) sepia(6%) saturate(300%); }
.bb-versions__bgpicker { display: flex; align-items: center; gap: 0.5rem; flex-wrap: wrap; justify-content: center; }
.bb-versions__swatchbtn {
  all: unset; width: 26px; height: 26px; border-radius: 50%; cursor: pointer;
  border: 2px solid rgba(255, 255, 255, 0.4); box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.1);
  transition: transform 0.15s ease;
}
.bb-versions__swatchbtn:hover { transform: scale(1.12); }
.bb-versions__swatchbtn.active { box-shadow: 0 0 0 2px var(--bb-gold); transform: scale(1.15); }
.bb-versions__custom { position: relative; width: 26px; height: 26px; cursor: pointer; }
.bb-versions__custom input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }
.bb-versions__custom span {
  display: flex; align-items: center; justify-content: center; width: 100%; height: 100%; border-radius: 50%;
  background: conic-gradient(from 0deg, #d9b673, #3d5a3e, #8a6a34, #12190f, #f3efe4, #d9b673);
  color: #fff; font-size: 0.9rem; font-weight: 700; border: 2px solid rgba(255, 255, 255, 0.5);
}
.bb-versions__custom span.active { box-shadow: 0 0 0 2px var(--bb-gold); }
.bb-versions__list { list-style: none; margin: 0; padding: 0; display: flex; flex-direction: column; gap: 0.6rem; }
.bb-versions__item {
  all: unset; cursor: pointer; display: flex; align-items: center; gap: 0.9rem; padding: 0.7rem 0.8rem;
  border: 0.5px solid rgba(27, 42, 26, 0.15); border-radius: 10px; transition: border-color 0.2s ease, background 0.2s ease;
}
.bb-versions__item img { width: 40px; height: 40px; object-fit: contain; flex: none; }
.bb-versions__item strong { display: block; font-size: 0.9rem; }
.bb-versions__item p { margin: 0.15rem 0 0; font-size: 0.78rem; color: var(--bb-ink-soft); }
.bb-versions__item:hover { border-color: var(--bb-gold); }
.bb-versions__item.active { border-color: var(--bb-gold-dark); background: rgba(217, 182, 115, 0.1); }

/* ---------- área de proteção ---------- */
.bb-clearspace { display: flex; justify-content: center; color: var(--bb-ink); }
.bb-clearspace__box { stroke-dasharray: 1; stroke-dashoffset: 1; }
.bb-clearspace.is-in .bb-clearspace__box { animation: bb-draw 1.1s ease-out forwards; }

/* ---------- tamanho mínimo ---------- */
.bb-minsize { display: flex; flex-direction: column; align-items: center; gap: 1rem; max-width: 22rem; margin: 0 auto; }
.bb-minsize__preview {
  display: flex; align-items: center; justify-content: center; width: 100%; min-height: 220px;
  background: var(--bb-sand); border-radius: 12px; border: 0.5px solid rgba(27, 42, 26, 0.12); transition: box-shadow 0.3s ease;
}
.bb-minsize__preview.warn { box-shadow: inset 0 0 0 2px rgba(179, 69, 42, 0.4); }
.bb-minsize__preview img { transition: width 0.05s linear, height 0.05s linear; }
.bb-minsize input[type='range'] { width: 100%; accent-color: var(--bb-gold-dark); }
.bb-minsize__readout { display: flex; flex-direction: column; align-items: center; gap: 0.2rem; font-size: 0.85rem; color: var(--bb-ink-soft); text-align: center; }
.bb-minsize__warning { color: #b3452a; font-weight: 600; }

/* ---------- paleta ---------- */
.bb-palette { display: grid; grid-template-columns: repeat(auto-fit, minmax(190px, 1fr)); gap: 1rem; }
.bb-swatch { display: flex; flex-direction: column; border-radius: 12px; overflow: hidden; border: 0.5px solid rgba(27, 42, 26, 0.15); transition: transform 0.15s ease; }
.bb-swatch:hover { transform: translateY(-2px); }
.bb-swatch__color { all: unset; cursor: pointer; height: 64px; display: block; }
.bb-swatch__meta { padding: 0.6rem 0.8rem 0.8rem; display: flex; flex-direction: column; gap: 0.2rem; background: #fffdf8; }
.bb-swatch__meta strong { font-size: 0.85rem; }
.bb-swatch__meta > span { font-size: 0.78rem; color: var(--bb-gold-dark); font-family: monospace; }
.bb-swatch__meta small { font-size: 0.72rem; color: var(--bb-ink-soft); }
.bb-swatch__use { all: unset; cursor: pointer; font-size: 0.72rem; color: var(--bb-ink-soft); margin-top: 0.4rem; text-decoration: underline; text-underline-offset: 2px; }
.bb-swatch__use:hover { color: var(--bb-gold-dark); }

.bb-gradients { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; }
.bb-gradient {
  all: unset; cursor: pointer; height: 96px; border-radius: 12px; display: flex; align-items: flex-end; padding: 0.8rem 1rem;
  font-size: 0.85rem; font-weight: 600; color: #f3efe4; transition: transform 0.15s ease;
}
.bb-gradient:hover { transform: translateY(-2px); }

/* ---------- tipografia ---------- */
.bb-type { display: grid; gap: 1.2rem; }
@media (min-width: 640px) { .bb-type { grid-template-columns: 1fr 1fr; } }
.bb-type__card { border: 0.5px solid rgba(27, 42, 26, 0.15); border-radius: 12px; padding: 1.5rem; }
.bb-type__label { font-size: 0.75rem; text-transform: uppercase; letter-spacing: 0.1em; color: var(--bb-gold-dark); margin: 0 0 0.8rem; }
.bb-type__sample { margin: 0 0 0.8rem; font-size: 1.9rem; transition: font-weight 0.15s ease; }
.bb-type__sample--serif { font-family: 'Cormorant Garamond', serif; font-weight: 600; }
.bb-type__sample--sans { font-family: 'Inter', sans-serif; }
.bb-type__use { font-size: 0.8rem; color: var(--bb-ink-soft); margin: 0 0 0.8rem; }
.bb-type__card input[type='range'] { width: 100%; accent-color: var(--bb-gold-dark); }
.bb-type__weight { font-size: 0.72rem; color: var(--bb-ink-soft); margin: 0.3rem 0 0; font-family: monospace; }
.bb-type__pairing { margin-top: 1.5rem; padding: 1.8rem; border-radius: 12px; background: var(--bb-moss); color: var(--bb-sand); }
.bb-type__pairing .bb-kicker { color: var(--bb-gold); }
.bb-type__pairing-title { font-family: 'Cormorant Garamond', serif; font-size: 1.5rem; font-weight: 600; margin: 0 0 0.6rem; }
.bb-type__pairing-body { font-size: 0.9rem; line-height: 1.6; opacity: 0.85; margin: 0; }

/* ---------- usos incorretos (flip cards) ---------- */
.bb-misuse-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 1rem; }
.bb-flip { perspective: 800px; cursor: pointer; height: 140px; }
.bb-flip__inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s cubic-bezier(0.22, 1, 0.36, 1); transform-style: preserve-3d; }
.bb-flip.flipped .bb-flip__inner { transform: rotateY(180deg); }
.bb-flip__front, .bb-flip__back {
  position: absolute; inset: 0; backface-visibility: hidden; border-radius: 10px; padding: 1rem;
  display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; font-size: 0.85rem;
}
.bb-flip__front { background: #fffdf8; border: 0.5px solid rgba(196, 70, 40, 0.35); color: var(--bb-ink); gap: 0.3rem; }
.bb-flip__x { font-size: 1.6rem; color: #b3452a; line-height: 1; margin-bottom: 0.2rem; transition: transform 0.3s ease; }
.bb-flip:hover .bb-flip__x { transform: rotate(90deg); }
.bb-flip__tap { font-size: 0.65rem; color: var(--bb-ink-soft); opacity: 0.6; }
.bb-flip__back { background: var(--bb-moss); color: var(--bb-sand); transform: rotateY(180deg); }

/* ---------- aplicações (mockups) ---------- */
.bb-mockups { display: flex; flex-wrap: wrap; gap: 1.5rem; }
.bb-mock { border-radius: 10px; overflow: hidden; transition: background 0.3s ease; }
.bb-mock--card {
  width: 220px; height: 130px; color: var(--bb-sand);
  display: flex; flex-direction: column; align-items: flex-start; justify-content: flex-end; padding: 1rem; gap: 0.4rem;
}
.bb-mock--card img { width: 34px; height: 34px; }
.bb-mock--card strong { display: block; font-family: 'Cormorant Garamond', serif; font-size: 1rem; }
.bb-mock--card small { font-size: 0.65rem; letter-spacing: 0.08em; text-transform: uppercase; opacity: 0.75; }
.bb-mock--avatar { width: 90px; height: 90px; border-radius: 50%; background: var(--bb-gold); display: flex; align-items: center; justify-content: center; }
.bb-mock--avatar img { width: 64px; height: 64px; }
.bb-mock--doc { width: 220px; padding: 1rem; background: #fffdf8; border: 0.5px solid rgba(27, 42, 26, 0.15); }
.bb-mock--doc img { width: 130px; color: var(--bb-ink); margin-bottom: 0.8rem; }
.bb-mock__lines { display: flex; flex-direction: column; gap: 0.4rem; }
.bb-mock__lines span { height: 6px; border-radius: 3px; background: rgba(27, 42, 26, 0.12); display: block; }

/* ---------- arquivos ---------- */
.bb-files-wrap { overflow-x: auto; }
.bb-files { width: 100%; border-collapse: collapse; font-size: 0.85rem; min-width: 480px; }
.bb-files th { text-align: left; font-size: 0.72rem; text-transform: uppercase; letter-spacing: 0.08em; color: var(--bb-ink-soft); padding-bottom: 0.6rem; border-bottom: 1px solid rgba(27, 42, 26, 0.15); }
.bb-files td { padding: 0.6rem 0.5rem 0.6rem 0; border-bottom: 0.5px solid rgba(27, 42, 26, 0.1); }
.bb-files__name { all: unset; cursor: pointer; }
.bb-files code { font-size: 0.8rem; background: rgba(27, 42, 26, 0.06); padding: 0.15rem 0.4rem; border-radius: 4px; transition: background 0.2s ease; }
.bb-files__name:hover code { background: rgba(217, 182, 115, 0.25); }
.bb-tag { font-size: 0.68rem; font-weight: 600; padding: 0.15rem 0.5rem; border-radius: 999px; background: rgba(217, 182, 115, 0.25); color: var(--bb-gold-dark); }
.bb-footer { margin-top: 3rem; font-size: 0.75rem; color: var(--bb-ink-soft); text-align: center; }

@keyframes bb-draw { to { stroke-dashoffset: 0; } }

/* ---------- reduced motion ---------- */
.bb.no-motion .bb-block,
.bb.no-motion .bb-clearspace__box,
.bb.no-motion .bb-cover__ring,
.bb.no-motion .bb-cover__title,
.bb.no-motion .bb-cover__grain,
.bb.no-motion .bb-cover__hint-dot,
.bb.no-motion .bb-construction__layer,
.bb.no-motion .bb-construction__guide {
  opacity: 1 !important;
  transform: none !important;
  animation: none !important;
  stroke-dashoffset: 0 !important;
  transition: none !important;
  background-position: 0 0 !important;
}
</style>