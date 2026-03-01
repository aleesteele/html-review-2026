<script setup>
import { ref, computed, onMounted, onUnmounted, reactive } from 'vue'

const scrollY = ref(0)
const windowHeight = ref(0)

function onScroll() {
  scrollY.value = window.scrollY
}

function onResize() {
  windowHeight.value = window.innerHeight
  measureTitle()
}

onMounted(() => {
  windowHeight.value = window.innerHeight
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('resize', onResize, { passive: true })
  document.addEventListener('touchstart', onTouchOutside, { passive: true })
  window.addEventListener('scroll', onScrollHideTooltip, { passive: true })
  setTimeout(measureTitle, 100)
})

onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('resize', onResize)
  document.removeEventListener('touchstart', onTouchOutside)
  window.removeEventListener('scroll', onScrollHideTooltip)
})

// ── Scroll phases ──
const titleScrollRange = 5
const fadeRange = 1

const titleEl = ref(null)
const titleWidthVw = ref(300) // fallback

function measureTitle() {
  if (titleEl.value) {
    const w = titleEl.value.offsetWidth
    const vw = window.innerWidth
    titleWidthVw.value = (w / vw) * 100
  }
}

const textOffset = computed(() => {
  if (windowHeight.value === 0) return 100
  const totalRange = titleScrollRange + fadeRange
  const progress = Math.min(Math.max(scrollY.value / (windowHeight.value * totalRange), 0), 1)
  // Must travel: 100vw (start off-screen right) + title width (to fully exit left)
  const totalTravel = 100 + titleWidthVw.value + 20 // 20vw buffer
  return 100 - (progress * totalTravel)
})

const titleOpacity = computed(() => {
  return 1
})

const bgOpacity = computed(() => {
  if (windowHeight.value === 0) return 1
  const fadeStart = windowHeight.value * titleScrollRange
  const fadeEnd = fadeStart + windowHeight.value * fadeRange
  // Fade back in: starts 2 viewports before the end of the page
  const docHeight = document.documentElement.scrollHeight
  const fadeBackStart = docHeight - windowHeight.value * 1
  const fadeBackEnd = docHeight - windowHeight.value

  // // Fade back in at the end
  // if (scrollY.value >= fadeBackStart) {
  //   const progress = Math.min((scrollY.value - fadeBackStart) / (fadeBackEnd - fadeBackStart), 1)
  //   return 0.30 + 0.70 * progress
  // }

  // Normal fade down during title→essay transition
  if (scrollY.value <= fadeStart) return 1
  if (scrollY.value >= fadeEnd) return 0.30
  return 1 - 0.70 * ((scrollY.value - fadeStart) / (fadeEnd - fadeStart))
})


const essayOpacity = computed(() => {
  return 1
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
  const tooltipWidth = 260 // max-width on mobile
  const padding = 12
  let x = rect.left + rect.width / 2
  // Clamp so tooltip doesn't bleed off edges
  x = Math.max(tooltipWidth / 2 + padding, x)
  x = Math.min(window.innerWidth - tooltipWidth / 2 - padding, x)
  tooltip.x = x
  tooltip.y = rect.top
  tooltip.note = annotation.note || ''
  tooltip.image = annotation.image || ''
  tooltip.link = annotation.link || ''
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

// Close tooltip on tap outside or scroll (mobile)
function onTouchOutside(event) {
  if (!tooltip.visible) return
  if (event.target.closest('.tooltip') || event.target.closest('.annotated')) return
  tooltip.visible = false
}

function onScrollHideTooltip() {
  if (tooltip.visible) tooltip.visible = false
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

  `I first heard about the Baekdusan in a book my {{umma|image:/images/umma.jpg|note:Me and my mom. 1994, Evanston, Illinois, USA.}} would read to me as a child. Each page was illustrated with myths, stories, and songs that she remembered from her own childhood in a {{mountain village|image:/images/yeogogeumgye-gil.png|note:Yeogogeumgye-gil, South Korea. (Google Maps, 2026)}} near Gwangju, a city in the south of Korea made famous by the bloody protests that took place there in 1980. She had bought the book after the 1988 Olympics, the first time an international event had taken place in her homeland, a glimmer of what eventually became the {{hallyu wave|note: The Korean Wave — the global spread of South Korean culture including K-pop, K-drama, film, and cuisine. The term was coined in the late 90s.}}. Appropriate for the type of patriotism it was intended to inspire, the book was titled {{"I Love Korea!"|image:/images/ilovekorea.jpg|note: Written By Andrew C. Nahm, B.J. Jones, and Gi-eun Lee (1991).}}, exclamation point included.`,

  `Of course, I was too young to understand the shifts in selfhood that migration creates as a child born and raised in the American empire. Instead, I loved the stories themselves, and the illustrations that showed me another way of seeing the world outside of {{high-rise Chicago skyscrapers|image:/images/chicago.jpg|note:Navy Pier with my cousins in Chicago, Illinois, USA (2003).}} or {{pay-to-pick farms|image:/images/larriland.jpg|note:Larriland Farm with my siblings in Woodbine, Maryland, USA (2000).}}. The world of the book (and therefore the world of my mother's homeland) was inhabited by talking tigers and sea spirits, by brothers and sisters that became the sun and the moon, of gods descending to earth through the mountains. One of my favorites was about the {{tiger and the bear|image:/images/bear+tiger.png|note:Illustration from "I love Korea!".}}.`,

  `In the beginning, {{Hwanung|image:/images/hwanung.png|note:Illustration from "I love Korea!".}}, the son of Hwanin (the Sky god) so wanted to come to earth that he was sent to the peak of {{Mount Baekdu|image:/images/mt-paekdu.png|note:백두산 — Baekdusan is an active stratovolcano on the border of North Korea and China and the highest one on the penninsula (2,744m). (Google Maps, 2026).}}. As he ruled the world from a city he created at the volcano's center, a bear and tiger came to ask if they could become humans. He gave the pair a stalk of mugwort and twenty pieces of garlic, and told them that if they ate both and avoided the sunlight for one hundred days, they would become human.`,

  `While the tiger only lasted a few days in the cave, the bear stayed and became a woman. She eventually wed Hwanung, and their son, {{Tan'gun|note:The founder of Gojoseon, the first Korean kingdom, said to have been established in 2333 BC.}} became the ancestor of the Korean people. He ruled for 1,500 years before eventually becoming a {{sanshin|note:산신 — Mountain spirit. Sanshin worship is one of the oldest continuous spiritual practices in Korea, predating Buddhism and Confucianism.}}.`,

  `This means that the birthplace of the Korean people lies within the volcano of Baekdusan, and that their ancestors are the gods themselves. Nation-building is a constant process, sometimes a loud one, sometimes a quiet one, and it begs us to suspend the disbelief that would tell us to question otherwise.`,

  `For me, these stories were magical illustrations of the world that once was, and it imbued nature with a magic I only started to understand when I started spending more time in mountains myself: first as a {{hiker in the Cascades|image:/images/enchantments.jpg|note:Me on the Enchantments trail in Washington, USA (2015). I broke my ankle later that day on the descent of this hike.}}, then as a {{researcher and mountaineer in the Himalayas|image:/images/uwicer.jpg|note:The Ugyen Wangchuck Institute for Forestry Research and Training is a government research and training institute in Jakar, Bhutan. Photo by Emma Greenberg (2017).}}, later a {{skier in the Alps|image:/images/chamonix.jpg|note:On my second ski tour in front of the Refuge du Requin, on the Vallée Blanche in Chamonix, France (2020). The first COVID-19 lockdown started the day after this photo was taken. Photo by David Littlejohn-Carrillo.}}.`,

  `The mountains have always drawn me in for one reason or another: their hulking mass a means of escaping my mind to be fully present in my body, one capable (by some miracle of my DNA) of walking, running, jumping, leaping, skiing, and climbing through these wild landscapes that make me feel appropriately small and insignificant. That's not to say that I'm particularly good at any of the sports I practice, nor am I particularly fast or strong.`,

  `I simply feel more alive when I'm outside. My face is more sensitive to the wind and the rain on my cheeks. My food, {{however disgustedly prepared|image:/images/camp-food.jpg|note:Canned chilli and an improvised crumble made with clotted cream and biscoff biscuits (2024).}}, tastes more delicious at altitude. My sense of mortality seems more profound and lending of direction when I can see the world from a bird's eye view.`,

  `That's not to say that I've always been like this. My suburban childhood predictably led to an urban adulthood. I came of age in {{global cities|image:/images/global-cities.png|note:In her 1991 book, sociologist Saskia Sassen coined the term 'Global City' to describe cities with outsized influence on the global economy during (what was at the time) the emerging phenomena of globalisation. I worked for her for a year during undergrad.}}: New York City for university, Los Angeles and Madrid for two different summers in college, Washington DC for my first job, Geneva for graduate school, London for another job. Most of my friends would call me a {{"city mouse"|note:"The Town Mouse and the Country Mouse" is one of Aesop's Fables, and has been adopted for many children's books.}}, especially in comparison to my more traditionally mountain goat-like partner.`,

  `But even in cities, I always found myself gravitating towards the spaces where I could find breathing room: {{the roof of my apartment in Inwood|image:/images/sam.jpg|note:Sam on my rooftop in at the northern tip of Manhattan, filtered through early-era Instagram (2016).}}, {{the ponds of Hampstead Heath|image:/images/hampstead-heath.jpg|note:Swimming in the mixed pond (2023). The heath is one of the oldest public spaces in London.}}, {{the rambling hills of Lac Léman|image:/images/jura.jpg|note:My desk in Geneva (2019).}} that were a run away from my apartment.`,

  `Going to the mountains has always felt more like a spiritual practice than an adventure, something I could never quite explain even to my sportiest of friends. I've always needed my dosage on a regular basis, and I could never really understand why until recently.`,

  `I came back to these myths of my childhood again when I started researching how to walk across the {{Baekdudaegan|image:/images/baekdudaegan-2.png|note:백두대간 — The main mountain ridge of the Korean peninsula, stretching roughly 1,500km from Baekdusan in the north to Jirisan then Hallasan in the south. Image by Yong-Chan Cho, et al (2024)}}.`,

  `The Baekdudaegan is a mountain range that is as much a product of mythology as it is of geology, roughly 1,500 kilometers across the Korean peninsula. It starts at Baekdusan, an active volcano that sits on the border of what is now known as the Democratic Republic of Korea (North Korea) and China. After curving down the peninsula, the range ends at {{Hallasan|image:/images/hallasan.png|note:한라산 — A shield volcano and the highest peak in South Korea (1,950m), located on Jeju Island.}} on Jeju Island, a dormant volcano that sits on what's known as the "Hawaii of Korea", the home of the legendary {{hanyeo|note:해녀 — Female free-divers of Jeju Island, who were recognized by UNESCO as Intangible Cultural Heritage. They dive without oxygen tanks to harvest seafood, primarily abalone, conch, sea urchins, octopus, sea cucumbers, and types of seaweed.}}. These two mountains sit at the heart of Korean mythology, with Baekdusan the father and Hallasan the mother cradling both sides of the peninsula in some kind of protective embrace.`,

  `When I first explored the possibility of a thru-hike through the Baekdudaegan a few years ago, I approached it in the same way an American thru-hiker might plan their route across the Pacific Coast or Appalachian Trails, long-distance paths made by mountaineers and foresters through landscapes carved over century-long cycles of violence, settlement, and conservation.`,

  `Like most keen thru-hikers, I started with a guide book. {{Mine was written by Roger Shepard|image:/images/guidebook.jpg|note:"The Baekdudaegan Trail: Hiking Korea's Mountain Spine" was first published in 2010, and it remains one of the few English-language guides to the trail.}}, a kiwi tour guide who has hiked in both North and South Korea, a privilege granted by the non-threatening otherness of his passport. While I obviously couldn't walk through the North Korean half of the range, I could walk through the Southern part that he had mapped: 687km distributed over a little more than 45 days according to the book's somewhat relaxed pace. I started to trace the route on online maps and scour the web for travel blogs that contained experiences akin to the one I was hoping to find. My heritage? My love of the mountains? Myself?`,

  `In any case, my research soon revealed that this is not how Koreans "do" the Baekdudaegan at all. They do it over many months, years, even decades, stringing together long weekends sprinkled in between the few holidays that Koreans are able to take off in an infamously punishing working culture.`,

  `Life starts, or rather flows down from the rivers that begin at the Baekdudaegan, a belief descendent from the combination of Taoism and Buddhism that made its way to Korea from the joint imperial influences of China and Japan, combining to make something different: a deep, instinctual respect for mountains, and a worship of nature despite whatever wider religion may be dominant at the time. I see this reverence play out in the way all of my family {{best appreciate their kimchi and rice outside|image:/images/camping.jpg|note:Camping with my grandparents in Great Falls, Virginia, USA (1999).}}, and up high. I took a road trip up the California coast a few years ago with my {{uncle and aunt|image:/images/roadtrip.jpg|note:Uncle Sung and Aunt Unmi, parked for lunch (2024).}}, and our best meals always consisted of {{shin ramyeon|note:신라면 — The most popular brand of instant spicy noodles in Korea. My favorite is actually the Neoguri (너구리) brand.}}, eggs, kimchi and rice, cooked under the shade of my uncle's van.`,

  `I learned about the sanshin – which roughly translates to mountain spirit – from David Mason, a scholar who's been tracking their worship across Korea for decades now. He is, in many ways, the kind of brash yet benevolent Orientalist that taught me in college, a product of 1960s counterculture and Alan Watts that spawned many an East Asian Studies scholar. He tells me that the sanshin encapsulates the living relationship that Koreans have with their mountains, interspersed with discussions about where energy flows, and how it strengthens in mountain air.`,

  `Maybe my tether to Korea lies in the mountains, as I've always been searching for an entry-point into a culture that never felt quite like mine to claim. My own relationship to my motherland is tenuous at best. Although my parents ironically now live in {{one of the largest Korean enclaves in the US|image:/images/annandale.png|note:Annandale, Virginia is an almost-town in Fairfax County, Virginia, widely known as "Koreatown" due to its large Korean-American community and concentration of Korean businesses. (Google Maps, 2026).}}, for years we moved around whitewashed suburbs near Chicago, Washington DC, Atlanta, and Houston. Growing up, we oscillated between visiting our German-American side on the {{beaches of New Jersey|image:/images/stone-harbor.png|note:Stone Harbor, New Jersey, USA. (Google Maps, 2026).}} and the Korean-American side on the {{beaches of California|image:/images/los-angeles.png|note:Los Angeles, California, USA. (Google Maps, 2026).}}. The Stop Asian Hate movement put a thermometer to the melting pot of my childhood.`,

  `The problem is that the Korea my mother knows and remembers from her own childhood is not the one that exists now. It was only when I was a researcher in Bhutan that I started to understand this. In the mountains there, I found a landscape so laden with myth that I was afraid to misstep: because there was a rock that the {{Zhabdrung|note:Zhabdrung Ngawang Namgyal — the Tibetan Buddhist lama who unified Bhutan in the 17th century. Sacred sites associated with him are found throughout Bhutan.}} had meditated on, or because beyond that hill lay a forest full of ghosts, or a {{chorten|image:/images/chorten.jpg|note:Chortens are Buddhist shrines, also called stupas. Chendebji Chorten, pictured above, was built to subdue a local demon (2017).}} that had been a pilgrimage site for centuries.`,

  `There, children played games not unlike the ones my mom told me about growing up, variations of pick-up sticks and marbles that are as much a function of poverty as they are of culture. When she returned to Korea for the first time in 30 years back in 2018, she was overwhelmed by the hyper-urbanism of Seoul, and felt rejected by the relatives that expected her to have American money she never earned. I felt more connected to my mother's Korea in the Himalayas than I did in Seoul myself, {{when I visited for the first time in 2013|image:/images/seoul.jpg|note:Somewhere in Seoul, South Korea with my great-aunt and second cousin (2013).}}.`,

  `Throughout her life, caught between a childhood where she was taught how to play dead for roaming tigers and an adulthood spent raising four American children, my mom gravitated towards what she perceived to be a universal tether between the two. Christianity was the thread that connected all life, and more importantly it tethered her own story to the heavens above.`,

  `Foreign authors like Herman Hesse and Leo Tolstoy wrote intergenerational stories that helped her to understand her own complicated family, in the same way that epics like {{Min Jin Lee's Pachinko|note:Her bestselling novel, which was published in 2017, followed four generations of a Korean family through the Japanese occupation and diaspora. It was adapted into an Apple TV+ series in 2022.}} followed in their footsteps centuries later. We were taught to be fiercely proud of our Korean heritage without understanding necessarily why. But more than that, we were told that people ultimately all want the same things: to survive, to belong, to love and be loved.`,

  `Her four children have carried this forward with a kind of view from nowhere, expected to embrace these universal truths but without the tools, let alone the language, to know where we came from. Maybe that's what drew me to mountains in the first place. In their lack of specificity, but their universal creation through the collision of tectonic plates the world over, I found a way of connecting to a past that I will never quite experience, and a connection to a sort of universal truth. I learned about the sanshin, yes, but I loved the fact that many different mountain spirits exist in many different places. I followed my bliss to the tops of many mountains, the world over.`,

  `Each sub-range within the Baekdudaegan has its own legends and myths. {{Jirisan|image:/images/jirisan.png|note:지리산 — The Mountain of Wisdom, the highest peak on the South Korean mainland (1,915m).}} is known as the range that can give ordinary people wisdom. {{Soraksan|image:/images/seoraksan.png|note:설악산 — Snowy Peaks Mountains, which contain Daecheongbong Peak (1,708m) and many Buddhist sites.}} contains the cave where a Buddhist monk once meditated for more than 1000 years. {{Geumgangsan|image:/images/geumgangsan.png|note:금강산 — The Diamond Mountains. South Korean tourists briefly had access in the early 2000s.}} are known to be some of the most beautiful of the range, called the 'diamonds' of the peninsula, but they too lie beyond the borders of the North.`,

  `Even the Baekdusan has been a product of modern mythology, with the Kim family claiming that Kim Il Sung was born in the volcano's center. A few years ago, his grandson, {{Kim Jung Un rode a white horse|image:/images/kim-horse.jpg|note:Photograph from the BBC (2019).}} through the same landscape to reinvigorate his claim to the "Mount Baekdu bloodline".`,

  `I spoke to a scientist who coordinates a {{cross-national research project on Mount Baekdu|note:The Mount Paektu Research Centre (MPRC) is an international partnership that promotes collaborative geoscience and environmental research in the Democratic People's Republic of Korea.}}, coordinating a team of North and South Korean, British, and American scientists to investigate the tectonic movements of the holy volcano. After it showed signs of erupting in the early 2010s, international calls for disaster prevention enabled research to be done on a scale previously thought impossible. Professor James Hammond told me that until recently, glaciology, oceanography, and volcanology were seen as universal sciences in the sense that they knew no borders. When he became a scientist, research was seen as a form of diplomacy. Now, he says, it is increasingly seen as a {{tool for real politik|note:In 2010, the Royal Society released a document on science policy called "New frontiers of science diplomacy" that documented "the long history of scientists supporting international cooperation". In 2025, they released a follow-up called "Science diplomacy in an era of disruption" that acknowledges that "science diplomacy is a tool that is used to achieve a nation or organisation’s diplomatic objectives".}}.`,

  `From my London flat, I use Google maps to trace my mouse over Mount Baekdu, the bird's eye view afforded by satellite imagery somehow flattening rather than elevating the landscape's majesty. As I {{zoom into the volcano|image:/images/baekdusan-gif.gif|note:The crater lake at the summit of Baekdusan, called Heaven Lake (천지), is one of the highest crater lakes in the world.}}, a blinding white surrounds a deep blue pool, the center of the volcano that birthed both legend and modern politics.`,

  `As I scroll further down, there is a sea of deep green trees, interspersed with squares of snow, a quilt of data captured at different seasons stitched together for the semblance of continuity. Scrolling further, deep in North Korea now, {{I see labels for working camps|image:/images/yodok.png|note:관리소 – North Korean political prison camps (kwanliso) have been extensively documented by human rights organisations. Pictured is Yodok Camp, also known as Kwanliso 15 (Google Maps, 2026).}}, nondescript buildings that do little to capture the unimaginable acts that take place there. Geopolitical analysts read this satellite imagery like tea leaves, trying to understand a regime that remains closed to the world more than 70 years after the Korean War.`,

  `The Baekdudaegan has become a symbol, for obvious reasons, of Korean reunification. The white and blue flags that flew at the {{2018 Pyeongchang Olympics|image:/images/reunification-flag.png|note:North and South Korean athletes marched together under a unified Korean flag during the opening ceremony of the XXIII Winter Olympic Games, held in Pyeongchang, South Korea.}} brought tears to the eyes of my whole family, as the blue rendering of the peninsula was reminiscent of a body made whole, with its mountainous spine intact. The dream of a cross-national trail remains a pipedream, especially after the breakdown of diplomatic relations in Hanoi in 2019.`,

  `As I click around the landscape, across the 38th parallel now, I think I'm looking for something else. For the past year, I've been 'walking' the landscape on my computer with every season, waiting for a chance when my screen might match the summits of the psychogeography I've been mapping for myself.`,

  `But there's always something that gets in the way: a new job, an injury, a visa switch, a wedding. My digital walk is a means for me to be there, maybe not dissimilar from the researchers that follow North Korea's every move. Conversations with people closer to the Baekdudaegan bring me closer to the landscapes I have yet only traced on screens.`,

  `On my last birthday, during the same week I was meant to start my hike last year, I scheduled a call with Mudang Jenn, a Korean-American shaman born in Queens and initiated on {{Gyeryongsan|note:계룡산 — The Phoenix Dragon Mountain in South Chungcheong Province, considered one of Korea's most spiritually powerful mountains and a centre for shamanic practice.}}. She told me that when border restrictions loosened after {{leaders of the two Koreas met|image:/images/summit.jpg|note:South Korean President Moon Jae-in and North Korean leader Kim Jong Un at the summit of Mounta Baekdu (BBC, 2018).}} at Mount Baekdusan in 2018, thousands of mudangs took the opportunity to go on pilgrimage to the mountain. She gave me a {{saju reading|note:사주 — A Korean divination practice based on the four pillars of one's birth (year, month, day, hour), used to understand personality, fortune, and life path.}}, a folk practice increasingly reclaimed as a spiritual one after years of being relegated to superstition. I'm told that I am 80% metal element, a lightning-rod for my ancestors. She tells me that it's important for my ancestral energy to visit Jirisan. I want to believe her.`,

  `The last time I was in a mountain range was back in September. I was with my partner and a friend in the {{Picos de Europa|note:A mountain range in northern Spain spanning Asturias, Cantabria, and León. The name refers to these being the first European peaks sighted by sailors returning from the Americas.}}. We hiked for 8 days around three massifs, sleeping in {{tents that whipped violently|image:/images/picos.jpg|note:Day 1 of the Anillo de Picos trail (2025).|}} in the perennial autumn wind. The funny thing about actually walking in the mountains is that all of these questions of belonging and heritage tend to fade away. The landscape of my screen and myself distills into something else.`,

  `The questions are simple: How long have I been walking? When was the last time I ate? When did I last drink water? Am I sunburned? Am I warm? How many kilometers do I have to go? Where do I sleep tonight? What's for dinner? Maybe, in that blissful moment between these thoughts of survival, I actually think nothing at all. In another, I realise that maybe the biggest lie we have ever been told is that life is complex.`,

  `Maybe if I eat 20 garlics and a mugwort and stay in a cave for 100 days, I too will emerge a new being. Maybe if I eat 200 ramyeons and walk across the Baekdudaegan, I will be a better person.`,

  `My flights are booked for this year, to finally join the trail. I'll start with Hallasan, or perhaps Jirisan. This year, and maybe the year after that, and the one after that and so on, I will walk to Baekdusan. I told my umma, and she wants to come. So does my cousin, and my uncle, maybe one of my brothers. I won't bring my computer.`
]

// ── Parse annotations from raw text ──
function parseParagraph(raw) {
  const segments = []
  const regex = /\{\{(.+?)\}\}/g
  let lastIndex = 0
  let match

  while ((match = regex.exec(raw)) !== null) {
    if (match.index > lastIndex) {
      segments.push({ text: raw.slice(lastIndex, match.index) })
    }

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

  if (lastIndex < raw.length) {
    segments.push({ text: raw.slice(lastIndex) })
  }

  return segments
}

const essayParagraphs = computed(() => rawParagraphs.map(parseParagraph))
</script>

<template>
  <div class="page">
    <!-- Background mountains (z-index 1) -->
    <div class="fixed-layer fixed-layer--bg" :style="{ opacity: bgOpacity }">
      <img src="/images/background.png" alt="" class="fixed-layer__img" />
    </div>

    <!-- Foreground mountains (z-index 3) -->
    <div class="fixed-layer fixed-layer--fg-mountains">
      <img src="/images/foreground-mountains.png" alt="" class="fixed-layer__img fixed-layer__img--bottom" />
    </div>

    <!-- Foreground clouds (z-index 4, drifting) -->
    <div class="fixed-layer fixed-layer--fg-clouds">
      <img src="/images/foreground-clouds.png" alt="" class="fixed-layer__img fixed-layer__img--clouds" />
    </div>

    <!-- Scrolling content (z-index 2) -->
    <div class="scroll-content">

      <!-- Phase 1: Title horizontal scroll -->
      <div class="title-spacer">
        <div class="title-track">
          <h1
            ref="titleEl"
            class="title-text"
            :style="{ transform: 'translateX(' + textOffset + 'vw)' }"
          >Tell umma I'm walking to Baekdusan
          <span class="title-subtitle">By <a href="https://aleesteele.com" target="_blank" rel="noopener noreferrer" class="title-link">Anne Lee Steele</a></span>
          </h1>
        </div>
      </div>

      <!-- Phase 2: Crossfade spacer -->
      <div class="fade-spacer"></div>

      <!-- Phase 3: Essay -->
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

    <!-- Tooltip (z-index 100) -->
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
        <a
          v-if="tooltip.link"
          :href="tooltip.link"
          target="_blank"
          rel="noopener noreferrer"
          class="tooltip__link"
        >Read more →</a>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.page {
  background: #d5c9a6;
  min-height: 100vh;
  overflow-x: hidden;
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

.fixed-layer__img--bottom {
  width: 100%;
  height: auto;
  display: block;
}

.fixed-layer__img--clouds {
  object-fit: contain;
  object-position: center top;
  animation: cloud-drift 60s ease-in-out infinite alternate;
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
  height: 600vh;
  position: relative;
}

/* Title: fixed, vertically centered, no horizontal clipping */

.title-track {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 2;
  pointer-events: none;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.title-text {
  font-family: 'Noto Serif KR', sans-serif;
  font-size: clamp(3.5rem, 7.5vw, 6.5rem);
  font-weight: 800;
  color: #1a1a1a;
  background-color: #e4dbc1;
  border:#1a1a1a;
  border-style: solid;
  white-space: nowrap;
  padding: 2rem;
  margin: 0;
  will-change: transform;
  line-height: 1.1;
  flex-shrink: 0;
}

.title-subtitle {
  display: block;
  font-family: 'Noto Serif KR', sans-serif;
  font-style: oblique;
  font-size: 0.2em;
  font-weight: 400;
  margin-top: 0.8rem;
  letter-spacing: 0.05em;
  text-align: center;
}

.title-link {
  color: #1a1a1a;
  text-decoration: underline;
  text-decoration-style: dotted;
  text-underline-offset: 3px;
  pointer-events: auto;
}

.fade-spacer {
  height: 100vh;
}

/* ─── Essay ─── */
.essay {
  max-width: 900px;
  margin: 0 auto;
  padding: 5vh 2rem 100vh;
  will-change: opacity;
}

.essay__paragraph {
  font-family: 'Noto Serif KR', sans-serif;
  font-size: clamp(1rem, 1.4vw, 1.05rem);
  line-height: 1.75;
  color: #1a1a1a;
  font-weight: 700;
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
    padding: 5vh 2rem 100vh;
  }

  .title-text {
    font-size: clamp(2.5rem, 6.5vw, 4.5rem);
  }

  .tooltip {
    max-width: 260px;
  }

  .fixed-layer__img--clouds {
  animation: cloud-drift 40s ease-in-out infinite alternate;
}
}

@media (max-width: 480px) {
  .essay {
    padding: 5vh 2rem 100vh;
  }

  .title-text {
    font-size: clamp(2.3rem, 5.5vw, 3.5rem);
  }

  .tooltip {
    max-width: 220px;
  }

  .fixed-layer__img--clouds {
  animation: cloud-drift 30s ease-in-out infinite alternate;
}
}
</style>