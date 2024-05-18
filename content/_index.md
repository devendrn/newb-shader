+++
title = "Newb Shader"
template = "index.html"

[extra]
hero_title = "Newb Shader <i class='hero__title-hat fa-solid fa-wand-magic-sparkles'></i>"
hero_caption = "An enhanced vanilla shader for Minecraft BE"
hero_btns = [
    { name = "<i class='fa fa-download'></i> Download", url = "download" }
]
+++

<div style="width: 100%; display: flex; flex-wrap: wrap; gap: 10px;">

{{ explorecard(
    title = "Newb X Legacy"
    description = "Newb Shader for RenderDragon."
    image = "https://raw.githubusercontent.com/devendrn/newb-x-mcbe/main/docs/screenshots.jpg"
    url = "download"
    size = 2
) }}

{{ explorecard(
    title = "Newb Variants"
    description = "Modified versions of Newb X Legacy."
    image = "https://i.ibb.co/pKP5j02/newb-variants-collage.jpg"
    url = "variants"
    size = 1
) }}

</div>


<div style="text-align: center;">

# #aesthetic

**Newb Shader** makes your Minecraft world more pleasing!

<br>

---

## Features

<div style="display: flex; flex-wrap: wrap;">

| Lighting |
| - |
| Overworld, underwater, nether, and end lighting. |

| Wave effects |
| - |
| Green plants/leaves wave, farm plants wave, lantern wave, and water wave. |

| Rain effects |
| - |
| Wet ground with sky reflection, mist blow. |

| Water|
| - |
| Simple sky reflection, smoother water texture.  |

| Sky |
| - |
| Three color gradient.  |

| Clouds |
| - |
| Soft clouds with rain transition and aurora. |

| Entities |
| - |
| Soft highlight around edges of entities. |

</div>
<br>

---

## Discord Server

Join our Discord server if you are interested!

**[<i class='fa-brands fa-discord'></i> Newb Community](https://discord.gg/newb-community-844591537430069279)**

</div>

<style>

.hero__title {
  background: linear-gradient(80deg, hsl(10,100%,60%) 0%, hsl(220,100%,60%) 100%);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.hero__title-hat {
  font-size: 56px;
  animation-name: rotating;
  animation-duration: 10s;
  animation-iteration-count: infinite;
  animation-timing-function: ease-in-out;
}
 
@keyframes rotating {
  0% { transform: rotate(0deg); opacity: 1.0; }
  92% { transform: rotate(0deg); opacity: 0.5; }
  100% { transform: rotate(360deg); opacity: 1.0; }
}

</style>

