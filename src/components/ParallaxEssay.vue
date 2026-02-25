<script setup>
import { ref, computed, onMounted, onUnmounted, reactive } from 'vue'

const scrollY = ref(0)
const windowHeight = ref(0)

function onScroll() {
  scrollY.value = window.scrollY
}

function onResize() {
  windowHeight.value = window.innerHeight
}

onMounted(() => {
  windowHeight.value = window.innerHeight
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('resize', onResize, { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('resize', onResize)
})

// ── Scroll phases ──
const titleScrollRange = 3
const fadeRange = 1

const textOffset = computed(() => {
  if (windowHeight.value === 0) return 100
  const progress = Math.min(Math.max(scrollY.value / (windowHeight.value * titleScrollRange), 0), 1)
  // Changed from 150 to 250 to ensure the text moves far enough to clear the screen on the left
  return 100 - (progress * 250)
})

const subtitleOffset = computed(() => {
  if (windowHeight.value === 0) return -100
  const progress = Math.min(Math.max(scrollY.value / (windowHeight.value * titleScrollRange), 0), 1)
  // Subtitle: Starts at -100vw (left), scrolls to 150vw (off-screen right)
  return -100 + (progress * 250)
})

const titleOpacity = computed(() => {
  // Changed to return 1 consistently so the title no longer fades away
  return 1
})

const bgOpacity = computed(() => {
  if (windowHeight.value === 0) return 1
  const fadeStart = windowHeight.value * titleScrollRange
  const fadeEnd = fadeStart + windowHeight.value * fadeRange
  if (scrollY.value <= fadeStart) return 1
  if (scrollY.value >= fadeEnd) return 0.5
  return 1 - 0.5 * ((scrollY.value - fadeStart) / (fadeEnd - fadeStart))
})

const essayOpacity = computed(() => {
  if (windowHeight.value === 0) return 0
  const fadeStart = windowHeight.value * titleScrollRange
  const fadeEnd = fadeStart + windowHeight.value * fadeRange
  if (scrollY.value <= fadeStart) return 0
  if (scrollY.value >= fadeEnd) return 1
  return (scrollY.value - fadeStart) / (fadeEnd - fadeStart)
})

// ── Tooltip state ──
const tooltip = reactive({
  visible: false,
  x: 0,
  y: 0,
  note: '',
  image: ''
})

let hideTimeout = null

function showTooltip(event, annotation) {
  if (hideTimeout) clearTimeout(hideTimeout)
  const rect = event.target.getBoundingClientRect()
  tooltip.x = rect.left + rect.width / 2
  tooltip.y = rect.top
  tooltip.note = annotation.note || ''
  tooltip.image = annotation.image || ''
  tooltip.visible = true
}

function hideTooltip() {
  hideTimeout = setTimeout(() => {
    tooltip.visible = false
  }, 200)
}

function keepTooltip() {
  if (hideTimeout) clearTimeout(hideTimeout)
}

// ══════════════════════════════════════════════════════
// ESSAY DATA WITH ANNOTATIONS
// ══════════════════════════════════════════════════════
//
// Each paragraph is a string. To add a tooltip annotation,
// wrap the phrase in double curly braces with a pipe-separated
// annotation config:
//
//   {{phrase|note:Your note text here}}
//   {{phrase|image:/images/my-photo.jpg}}
//   {{phrase|note:Some note|image:/images/photo.jpg}}
//
// Examples:
//   "I visited {{Gwangju|note:A city in South Jeolla Province, known for the 1980 uprising.}} last year."
//   "We ate {{shin ramen|image:/images/ramen.jpg}} on the mountain."
//   "The {{Baekdudaegan|note:Korea's main mountain ridge|image:/images/map.jpg}} stretches 1500km."
//
// ══════════════════════════════════════════════════════

const rawParagraphs = [
  `by Anne Lee Steele`,

  `I first heard about the Baekdusan in a book my umma would read to me as a child. Each page was illustrated with myths, stories, and songs that she remembered from her own childhood in a mountain village near {{Gwangju|note:A city in South Jeolla Province, South Korea, known for the pro-democracy uprising of May 1980.}}, a city in the south of Korea made famous by the bloody protests that took place there in 1980. She had bought the book after the 1988 Olympics, the first time an international event had taken place in her homeland, a glimmer of what eventually became the {{hallyu wave|note:The Korean Wave — the global spread of South Korean culture including K-pop, K-drama, film, and cuisine.}}. Appropriate for the type of patriotism it was intended to inspire, the book was titled "I Love Korea!", exclamation point included.`,

  `Of course, I was too young to understand the shifts in selfhood that migration creates as a child born and raised in the American empire. Instead, I loved the stories themselves, and the illustrations that showed me another way of seeing the world outside of high-rise Chicago skyscrapers or midwestern pay-to-pick farms. The world of the book (and therefore the world of my mother's homeland) was inhabited by talking tigers and sea spirits, by brothers and sisters that became the sun and the moon, of gods descending to earth through the mountains. One of my favorites was about the tiger and the bear.`,

  `In the beginning, {{Hwanung|note:A figure in Korean mythology, the son of Hwanin (Lord of Heaven), who descended to earth to found a city on Mount Paekdu.}}, the son of Hwanin (the Sky god) so wanted to come to earth that he was sent to the peak of Mount Paekdu. As he ruled the world from a city he created at the volcano's center, a bear and tiger came to ask if they could become humans. He gave the pair a stalk of mugwort and twenty pieces of garlic, and told them that if they ate both and avoided the sunlight for one hundred days, they would become human.`,

  `While the tiger only lasted a few days in the cave, the bear stayed and became a woman. She eventually wed Hwanung, and their son, {{Tan'gun|note:The legendary founder of Gojoseon, the first Korean kingdom, said to have been established in 2333 BC.}} became the ancestor of the Korean people. He ruled for 1,500 years before eventually becoming a mountain god ({{sanshin|note:산신 — Mountain spirit. Sanshin worship is one of the oldest continuous spiritual practices in Korea, predating Buddhism and Confucianism.}}).`,

  `This means that the birthplace of the Korean people lies within the volcano of Baekdusan, and that their ancestors are the gods themselves. Nation-building is a constant process, sometimes a loud one, sometimes a quiet one, and it begs us to suspend the disbelief that would tell us to question otherwise.`,

  `For me, these stories were magical illustrations of the world that once was, and it imbued nature with a magic I only started to understand when I started spending more time in mountains myself: first as a hiker in the Adirondacks and the Cascades, then as a climber in the Andes, a researcher in the Himalayas, later a skier in the Alps.`,

  `The mountains have always drawn me in for one reason or another: their hulking mass a means of escaping my mind to be fully present in my body, one capable (by some miracle of my DNA) of walking, running, jumping, leaping, skiing, and climbing through these wild landscapes that make me feel appropriately small and insignificant. That's not to say that I'm particularly good at any of the sports I practice, nor am I particularly fast or strong.`,

  `There's no real meaningful explanation as to why I like mountains other than that I simply feel more alive when I'm outside. My face is more sensitive to the wind and the rain on my cheeks. My food, however disgustedly prepared, tastes more delicious at altitude. My sense of mortality seems more profound and lending of direction when I can see the world from a bird's eye view.`,

  `That's not to say that I've always been like this. My suburban childhood led to an urban adulthood. I came of age in global cities: New York City for university, Los Angeles and Madrid for two different summers in college, Washington DC for my first job, Geneva for graduate school, London for another job. Most of my friends would likely categorize me as a "city mouse", even a "townie", especially in comparison to my more traditionally mountain goat-like partner.`,

  `But even in cities, I always found myself gravitating towards the spaces where I could find breathing room: the roof of my apartment in {{Inwood|note:A neighbourhood at the northern tip of Manhattan, known for its proximity to Fort Tryon Park and the Cloisters.}}, the ponds of Hampstead Heath, the rambling hills of the lower Jura that were a run away from my apartment.`,

  `Going to the mountains has always felt more like a spiritual practice than an adventure, something I could never quite explain even to my sportiest of friends. I've always needed my dosage on a regular basis, and I could never really understand why until recently.`,

  `I came back to these myths of my childhood again when I started researching how to walk across the {{Baekdudaegan|note:백두대간 — The main mountain ridge of the Korean peninsula, stretching roughly 1,500km from Baekdusan in the north to Hallasan in the south.}}.`,

  `The Baekdudaegan is a mountain range that is as much a product of mythology as it is of geology, roughly 1,500 kilometers across the Korean peninsula. It starts at Baekdusan, an active volcano that sits on the border of what is now known as the Democratic Republic of Korea (North Korea) and China. After curving down the peninsula, the range ends at {{Hallasan|note:한라산 — A shield volcano and the highest peak in South Korea (1,950m), located on Jeju Island.}} on Jeju Island, a dormant volcano that sits on what's known as the "Hawaii of Korea", the home of the legendary {{hanyeo|note:해녀 — Female free-divers of Jeju Island, recognized by UNESCO as Intangible Cultural Heritage. They dive without oxygen tanks to harvest seafood.}} freediving women. These two mountains sit at the heart of Korean mythology, with Baekdusan the father and Hallasan the mother cradling both sides of the peninsula in some kind of protective embrace.`,

  `When I first explored the possibility of a thru-hike through the Baekdudaegan a few years ago, I approached it in the same way an American thru-hiker might plan their route across the Pacific Coast or Appalachian Trails, long-distance paths made by mountaineers and foresters through landscapes carved over century-long cycles of violence, settlement, and conservation.`,

  `Like most keen thru-hikers, I started with a guide book. Mine was written by {{Roger Shepard|note:Author of "Korea's Mountain Ridgeline Trail: The Baekdu Daegan", one of the few English-language guides to the trail.}}, a kiwi tour guide who has hiked in both North and South Korea, a privilege granted by the non-threatening otherness of his passport. While I obviously couldn't walk through the North Korean half of the range, I could walk through the Southern part that he had mapped: 687km distributed over a little more than 45 days according to the book's somewhat relaxed pace.`,

  `I started to trace the route on online maps and scour the web for travel blogs that contained experiences akin to the one I was hoping to find. My heritage? My love of the mountains? Myself?`,

  `In any case, my research soon revealed that this is not how Koreans "do" the Baekdudaegan at all. They do it over many months, years, even decades, stringing together long weekends sprinkled in between the few holidays that Koreans are able to take off in an infamously punishing working culture.`,

  `Life starts, or rather flows down from the rivers that begin at the Baekdudaegan, a belief descendent from the combination of Taoism and Buddhism that made its way to Korea from the joint imperial influences of China and Japan, combining to make something different: a deep, instinctual respect for mountains, and a worship of nature despite whatever wider religion may be dominant at the time. I see this reverence play out in the way all of my family best appreciate their kimchi and rice outside, and up high. I took a road trip up the California coast a few years ago with my uncle and aunt, and our best meals always consisted of {{shin ramen|note:신라면 — A popular brand of instant spicy noodles, a staple of Korean outdoor eating.}}, eggs, kimchi and rice, cooked under the shade of my uncle's van.`,

  `I learned about the sanshin – which roughly translates to mountain spirit – from {{David Mason|note:An American scholar based in South Korea who has spent decades documenting sanshin shrines and mountain worship traditions across the peninsula.}}, a scholar who's been tracking their worship across Korea for decades now. He is, in many ways, the kind of brash yet benevolent Orientalist that taught me in college, a product of 60s counterculture and Alan Watts that spawned many an East Asian Studies scholar. The sanshin encapsulates the living relationship that Koreans have with their mountains, interspersed with discussions about where energy flows, and how it strengthens in mountain air.`,

  `Maybe my tether to Korea lies in the mountains, as I've always been searching for an entry-point into a culture that never felt quite like mine to claim. My own relationship to my motherland is tenuous at best. Although my parents ironically now live in one of the largest Korean enclaves in the US, a town called {{Annandale, Virginia|note:A census-designated place in Fairfax County, Virginia, known as "Koreatown" for its large Korean-American community and concentration of Korean businesses.}}, for years we moved around whitewashed suburbs near Chicago, Washington DC, Atlanta, and Houston. Growing up, we oscillated between visiting our German-American side on the beaches of New Jersey and the Korean-American side on the beaches of California. The Stop Asian Hate movement put a thermometer to the melting pot that shaped my childhood.`,

  `The problem is that the Korea my mother knows and remembers from her own childhood is not the one that exists now. It was only when I was a researcher in Bhutan that I started to understand this. In the mountains there, I found a landscape so laden with myth that I was afraid to misstep: because there was a rock that the {{Zhabdrung|note:Zhabdrung Ngawang Namgyal — the Tibetan Buddhist lama who unified Bhutan in the 17th century. Sacred sites associated with him are found throughout the country.}} had meditated on, or because beyond that hill lay a forest full of ghosts, or a {{chorten|note:A Buddhist shrine or stupa, common throughout the Himalayas, often marking sacred sites or serving as points for circumambulation.}} (stupa) that had been a pilgrimage site for centuries.`,

  `There, children played games not unlike the ones my mom told me about growing up, variations of pick-up sticks and marbles that are as much a function of poverty as they are of culture. When she returned to Korea for the first time in 30 years back in 2018, she was overwhelmed by the hyper-urbanism of Seoul, and felt rejected by the relatives that expected her to have American money she never earned. I felt more connected to my mother's Korea in the Himalayas than I did in Seoul myself, when I visited for the first time in 2013.`,

  `Throughout her life, caught between a childhood where she was taught how to play dead for roaming tigers and an adulthood spent raising four American children, she gravitated towards what she perceived to be a universal tether between the two. Christianity was the thread that connected all life, and more importantly it tethered her own story to the heavens above.`,

  `Foreign authors like Herman Hesse and Leo Tolstoy wrote intergenerational stories that helped her to understand her own complicated family, in the same way that epics like {{Min Jin Lee's Pachinko|note:A bestselling novel (2017) following four generations of a Korean family through the Japanese occupation and diaspora. It was adapted into an Apple TV+ series.}} followed in their footsteps centuries later. We were taught to be fiercely proud of our Korean heritage without understanding necessarily why. But more than that, we were told that people ultimately all want the same things: to survive, to belong, to love and be loved.`,

  `Her four children have carried this forward with a kind of view from nowhere, expected to embrace these universal truths but without the tools, let alone the language, to know where we came from. Maybe that's what drew me to mountains in the first place. In their lack of specificity, but their universal creation through the collision of tectonic plates the world over, I found a way of connecting to a past that I will never quite experience, and a connection to a sort of universal truth. I learned about the sanshin, yes, but I loved the fact that many different mountain spirits exist in many different places. I followed my bliss to the tops of many mountains, the world over.`,

  `Each sub-range within the Baekdudaegan has its own legends and myths. {{Jirisan|note:지리산 — The highest peak on the Korean mainland (1,915m) and one of the most sacred mountains in Korean culture, said to give wisdom to those who climb it.}} is known as the range that can give ordinary people wisdom. {{Soraksan|note:설악산 — A dramatic granite mountain range in Gangwon Province, home to ancient Buddhist temples and known for its autumn foliage.}} contains the cave where a Buddhist monk once meditated for more than 1000 years. {{Geumgangsan|note:금강산 — The Diamond Mountains, located in North Korea and considered among the most beautiful landscapes on the peninsula. South Korean tourists briefly had access in the early 2000s.}} are known to be some of the most beautiful of the range, called the 'diamonds' of the peninsula, but they too lie beyond the borders of the North. Even the Baekdusan has been a product of modern mythology, with the Kim family claiming that Kim Il Sung was born in the volcano's center. A few years ago, his grandson, Kim Jung Un rode a white horse through the same landscape to reinvigorate his claim to the "Mount Paekdu bloodline".`,

  `I spoke to a scientist who coordinates a cross-national research project on Mount Paekdu, coordinating North and South Korean, British, and American scientists to investigate the tectonic movements of the holy volcano. After it showed signs of erupting in the early 2010s, international calls for disaster prevention enabled research to be done on a scale previously thought impossible. {{Professor James Hammond|note:A British geophysicist at Birkbeck, University of London, who led collaborative research with North Korean scientists studying the volcanic activity of Mount Paekdu.}} is a British geophysicist with a background in working in geopolitically tense environments. He told me that until recently, glaciology, oceanography, and volcanology were seen as universal sciences in the sense that they knew no borders. When he became a scientist, research was seen as a form of diplomacy. Now, he says, it is increasingly seen as a tool for real politik.`,

  `From my London flat, I use Google maps to trace my mouse over Mount Paekdu, the bird's eye view afforded by satellite imagery somehow flattening rather than elevating the landscape's majesty. As I zoom into the volcano, a blinding white surrounds a deep blue pool, the center of the volcano that birthed both legend and modern politics. As I scroll further down, there is a sea of deep green trees, interspersed with squares of snow, a quilt of data captured at different seasons stitched together for the semblance of continuity. Scrolling further, deep in North Korea now, I see labels for working camps, nondescript buildings that do little to capture the unimaginable acts that take place there. Geopolitical analysts read this satellite imagery like tea leaves, trying to understand a regime that remains closed to the world more than 70 years after the Korean War.`,

  `The Baekdudaegan has become a symbol, for obvious reasons, of Korean reunification. The white and blue flags that flew at the {{2018 Pyeongchang Olympics|note:The XXIII Winter Olympic Games, held in Pyeongchang, South Korea. North and South Korean athletes marched together under a unified Korean flag during the opening ceremony.}} brought tears to the eyes of my whole family, as the blue rendering of the peninsula was reminiscent of a body made whole, with its mountainous spine intact. The dream of a cross-national trail remains a pipedream, especially after the breakdown of diplomatic relations in Hanoi in 2019.`,

  `As I click around the landscape, across the 38th parallel now, I think I'm looking for something else. For the past year, I've been 'walking' the landscape on my computer with every season, waiting for a chance when my screen might match the summits of the psychogeography I've been mapping for myself. But there's always something that gets in the way: a new job, an injury, a visa switch, a wedding. My digital walk is a means for me to be there, maybe not dissimilar from the researchers that follow North Korea's every move. Conversations with people closer to the Baekdudaegan bring me closer to the landscapes I have yet only traced on screens.`,

  `On my 32nd birthday, during the same week I was meant to start my hike last year, I scheduled a call with {{Mudang Jenn|note:A Korean-American mudang (shaman) based in New York and initiated in Korea. She practices traditional Korean shamanism and saju (four pillars) divination.}}, a Korean-American shaman born in Queens and initiated on {{Gyeryongsan|note:계룡산 — The Phoenix Dragon Mountain in South Chungcheong Province, considered one of Korea's most spiritually powerful mountains and a centre for shamanic practice.}}, the Phoenix Dragon Mountain. She told me that when border restrictions loosened after leaders of the two Koreas met at Mount Baekdusan in 2018, thousands of mudangs took the opportunity to go on pilgrimage to the mountain. She gave me a {{saju reading|note:사주 — A Korean divination practice based on the four pillars of one's birth (year, month, day, hour), used to understand personality, fortune, and life path.}}, a folkshamanistic practice increasingly reclaimed as a spiritual one after years of being relegated to cheap thrill. I'm told that I am 80% metal element, a lightning-rod for my ancestors. She tells me that it's important for my ancestral energy to visit Jirisan. I want to believe her.`,

  `The last time I was in a mountain range was back in September. I was with my partner and a friend in the {{Picos de Europa|note:A mountain range in northern Spain spanning Asturias, Cantabria, and León. The name refers to these being the first European peaks sighted by sailors returning from the Americas.}}, a small mountain range in the North of Spain named for the fact that they were often the first peaks that sailors saw when they returned to the European continent from the Americas. We hiked for 8 days around three massifs, sleeping in tents that whipped violently in the perennial autumn wind.`,

  `The funny thing about actually walking in the mountains is that all of these questions of belonging and heritage tend to fade away. The landscape of my screen and myself distills into something else. The questions are simple: How long have I been walking? When was the last time I ate? When did I last drink water? Am I sunburned? Am I warm? How many kilometers do I have to go? Where do I sleep tonight? What's for dinner? Maybe, in that blissful moment between these thoughts of survival, you actually think nothing at all. In another, you realise that maybe the biggest lie we have ever been told is that life is complex. Maybe if I eat 20 garlics and a mugwort and stay in a cave for 100 days, I too will emerge a new being. Maybe if I eat 200 ramens and walk across the Baekdudaegan, I will be a better person.`,

  `My flights are booked for this year, to finally join the trail. I'll start with Hallasan, or perhaps Jirisan. This year, and maybe the year after that, and the one after that and so on, I will walk to Baekdusan. I told my umma, and she wants to come with me. So does my cousin, and my uncle, maybe one of my brothers. I won't bring my computer.`
]

// ── Parse annotations from raw text ──
// Converts "text {{phrase|note:...|image:...}} more text"
// into [ {text: "text "}, {text: "phrase", annotation: {note, image}}, {text: " more text"} ]
function parseParagraph(raw) {
  const segments = []
  const regex = /\{\{(.+?)\}\}/g
  let lastIndex = 0
  let match

  while ((match = regex.exec(raw)) !== null) {
    // Text before the annotation
    if (match.index > lastIndex) {
      segments.push({ text: raw.slice(lastIndex, match.index) })
    }

    // Parse the annotation: "phrase|note:...|image:..."
    const inner = match[1]
    const parts = inner.split('|')
    const phrase = parts[0]
    const annotation = {}

    for (let i = 1; i < parts.length; i++) {
      const colonIdx = parts[i].indexOf(':')
      if (colonIdx > -1) {
        const key = parts[i].slice(0, colonIdx).trim()
        const val = parts[i].slice(colonIdx + 1).trim()
        annotation[key] = val
      }
    }

    segments.push({ text: phrase, annotation })
    lastIndex = match.index + match[0].length
  }

  // Remaining text after last annotation
  if (lastIndex < raw.length) {
    segments.push({ text: raw.slice(lastIndex) })
  }

  return segments
}

const essayParagraphs = computed(() => rawParagraphs.map(parseParagraph))
</script>

<template>
  <div class="page">
    <!-- ═══ FIXED MOUNTAIN LAYERS (always visible) ═══ -->

    <!-- Background mountains (z-index 1) -->
    <div class="fixed-layer fixed-layer--bg" :style="{ opacity: bgOpacity }">
      <img src="/images/background.png" alt="" class="fixed-layer__img" />
    </div>

    <!-- Foreground mountains (z-index 3, on top of text) -->
    <div class="fixed-layer fixed-layer--fg-mountains">
      <img src="/images/foreground-mountains.png" alt="" class="fixed-layer__img fixed-layer__img--bottom" />
    </div>

    <!-- Foreground clouds (z-index 4, drifting left to right) -->
    <div class="fixed-layer fixed-layer--fg-clouds">
      <img src="/images/foreground-clouds.png" alt="" class="fixed-layer__img fixed-layer__img--clouds" />
    </div>

    <!-- ═══ SCROLLING CONTENT (z-index 2, between the mountain layers) ═══ -->
    <div class="scroll-content">

      <!-- Phase 1: Title scroll spacer -->
        <div class="title-spacer">
        <div class="title-group">
          <div class="title-track" :style="{ opacity: titleOpacity }">
            <h1
              class="title-text"
              :style="{ transform: 'translateX(' + textOffset + 'vw)' }"
            >Tell umma I'm walking to Baekdusan</h1>
            <!-- <p 
                class="subtitle-text"
                :style="{ transform: 'translateX(' + subtitleOffset + 'vw)' }"
              >by Anne Lee Steele</p> -->
          </div>
        </div>
      </div>

      <!-- Phase 2: Transparent crossfade spacer -->
      <div class="fade-spacer"></div>

      <!-- Phase 3: Essay with annotated text -->
      <article class="essay" :style="{ opacity: essayOpacity }">
        <div class="essay__body">
          <p
            v-for="(segments, pIndex) in essayParagraphs"
            :key="pIndex"
            class="essay__paragraph"
          >
            <template v-for="(seg, sIndex) in segments" :key="sIndex">
              <span
                v-if="seg.annotation"
                class="annotated"
                @mouseenter="showTooltip($event, seg.annotation)"
                @mouseleave="hideTooltip"
                @focus="showTooltip($event, seg.annotation)"
                @blur="hideTooltip"
                tabindex="0"
              >{{ seg.text }}</span>
              <template v-else>{{ seg.text }}</template>
            </template>
          </p>
        </div>
      </article>
    </div>

    <!-- ═══ TOOLTIP (z-index 100, above everything) ═══ -->
    <Transition name="tooltip-fade">
      <div
        v-if="tooltip.visible"
        class="tooltip"
        :style="{
          left: tooltip.x + 'px',
          top: tooltip.y + 'px'
        }"
        @mouseenter="keepTooltip"
        @mouseleave="hideTooltip"
      >
        <img
          v-if="tooltip.image"
          :src="tooltip.image"
          alt=""
          class="tooltip__image"
        />
        <p v-if="tooltip.note" class="tooltip__note">{{ tooltip.note }}</p>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.page {
  background: #d5c9a6;
  min-height: 100vh;
}

/* ─── Fixed Mountain Layers ─── */
.fixed-layer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  pointer-events: none;
}

.fixed-layer--bg {
  z-index: 1;
}

.fixed-layer--fg-mountains {
  z-index: 3;
  top: auto;
  bottom: 0;
  height: auto;
}

.fixed-layer--fg-clouds {
  z-index: 4;
}

.fixed-layer__img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center center;
  display: block;
}

/* Mountains: natural size, pinned to bottom */
.fixed-layer__img--bottom {
  width: 100%;
  height: auto;
  display: block;
}

/* Clouds drift slowly left to right */
.fixed-layer__img--clouds {
  object-fit: contain;
  object-position: center top;
  animation: cloud-drift 50s ease-in-out infinite alternate;
}

@keyframes cloud-drift {
  0% { transform: translateX(-3%); }
  100% { transform: translateX(3%); }
}

/* ─── Scrolling Content ─── */
.scroll-content {
  position: relative;
  z-index: 2;
}

.title-spacer {
  height: 300vh;
  position: relative;
}

.title-track {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  z-index: 2;
  pointer-events: none;
  display: flex;
  align-items: center;
  overflow: visible;
}

.title-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem; /* Vertical space between title and subtitle */
  width: 100%;
}

.title-text {
  font-family: 'Noto Serif KR', sans-serif;
  font-size: clamp(3rem, 7vw, 6rem);
  font-weight: 700;
  color: #1a1a1a;
  white-space: nowrap;
  margin: 0;
  will-change: transform;
  line-height: 1.1;
}

.subtitle-text {
  font-family: 'Noto Serif KR', sans-serif;
  font-size: clamp(1.2rem, 2.5vw, 2.5rem);
  color: #1a1a1a;
  margin-top: 1rem;
  white-space: nowrap;
  will-change: transform;
}

.fade-spacer {
  height: 100vh;
}

/* ─── Essay ─── */
.essay {
  max-width: 1000px;
  margin: 0 auto;
  padding: 5vh 2rem 100vh;
  will-change: opacity;
}

.essay__paragraph {
  font-family: 'Noto Serif KR', sans-serif;                                              
  font-size: clamp(1rem, 1.4vw, 1.05rem);
  line-height: 1.75;
  color: #1a1a1a;
  font-weight: 500;
  margin-bottom: 1.8em;
  text-align: left;
}

/* ─── Annotated phrases ─── */
.annotated {
  text-decoration: underline;
  text-decoration-style: dotted;
  text-decoration-color: #6b8f71;
  text-underline-offset: 3px;
  cursor: help;
  transition: text-decoration-color 0.2s ease;
}

.annotated:hover,
.annotated:focus {
  text-decoration-color: #1a1a1a;
  text-decoration-style: solid;
  outline: none;
}

/* ─── Tooltip ─── */
.tooltip {
  position: fixed;
  z-index: 100;
  transform: translate(-50%, -100%) translateY(-12px);
  max-width: 320px;
  background: #f5f0e1;
  border: 1.5px solid #1a1a1a;
  padding: 12px;
  pointer-events: auto;
  box-shadow: 3px 3px 0px #1a1a1a;
}

.tooltip__image {
  width: 100%;
  max-height: 200px;
  object-fit: cover;
  display: block;
  margin-bottom: 8px;
}

.tooltip__note {
  font-family: 'Cascadia Code', sans-serif;
  font-size: 0.8rem;
  line-height: 1.5;
  color: #1a1a1a;
  margin: 0;
}

/* Tooltip transition */
.tooltip-fade-enter-active,
.tooltip-fade-leave-active {
  transition: opacity 0.15s ease;
}

.tooltip-fade-enter-from,
.tooltip-fade-leave-to {
  opacity: 0;
}

/* ─── Responsive ─── */
@media (max-width: 768px) {
  .essay {
    padding: 2vh 1.5rem 30vh;
  }

  .title-group { 
    gap: 1rem; 
  }

  .title-text {
    font-size: clamp(3rem, 6vw, 4rem);
    object-position: center center;
  }

  .tooltip {
    max-width: 260px;
  }
}

@media (max-width: 480px) {
  .essay {
    padding: 2vh 1rem 20vh;
  }

  .title-text {
    font-size: clamp(1.8rem, 5vw, 3rem);
  }

  .tooltip {
    max-width: 220px;
  }
}
</style>
