+++
title = "Glow preview"
description = ""

[extra]
use_twgl = true
+++

<style>

* {
  text-align: center
}

button {
  color: hsla(0, 0%, var(--fg-b), 1);
  background-color: hsla(0, 0%, var(--bg-b), 1);
  border: 1px solid hsla(0, 0%, var(--bg-c), 1);
  border-radius: 8px;
  padding: 10px;
}

button:hover {
  background-color: hsla(0, 0%, var(--bg-c), 1);
  border: 1px solid hsla(0, 0%, var(--bg-d), 1);
}

#container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-content: stretch;
  gap: 10px;
}

#settings {
  flex: 1;
  border: 1px solid hsla(0, 0%, var(--bg-c), 1);
  padding: 0 20px;
}

#settings > p {
  font-size: 18px;
  margin-bottom: 20px;
}

canvas {
  aspect-ratio: 1 / 1;
  min-width: 100px;
  max-width: 360px;
  background: hsla(0, 0%, var(--bg-c), 1);
  border: 1px solid hsla(0, 0%, var(--bg-c), 1);
}

input {
  display: inline;
}

.slider {
  -webkit-appearance: none;
  appearance: none;
  width: 100%; 
  height: 15px; 
  background-color: hsla(0, 0%, var(--bg-b), 1);
  border: 1px solid hsla(0, 0%, var(--bg-c), 1);
  border-radius: 8px;
  outline: none;
  opacity: 0.7;
  -webkit-transition: .2s;
  transition: opacity .2s;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 25px;
  height: 25px;
  background: hsla(100, 00%, var(--bg-c), 1);
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  background: hsla(100, 00%, var(--bg-d), 1);
  border: 1px solid hsla(100, 0%, var(--fg-d), 1);
  cursor: pointer;
}

tbody, table {
  width: 100%;
}

td:first-child {
  max-width: 80px;
}

td {
  text-align: right;
  background: none;
}

</style>

<button id="load-button">Load texture</button>

<div id="container"> 
  <canvas id="glow-canvas" width="360" height="360"></canvas>
  <div id="settings">
    <p>Glow preview settings</p>
    <table>
      <tr>
        <td>Ambient</td>
        <td><input type="range" min="0.0" max="1.5" step="0.02" value="0.4" class="slider" id="s-ambient"></td>
      </tr>
      <tr>
        <td>Glow intensity</td>
        <td><input type="range" min="0.0" max="5.0" step="0.02" value="2.2" class="slider" id="s-glow"></td>
      </tr>
      <tr>
        <td>Shimmer intensity</td>
        <td><input type="range" min="0.0" max="1.0" step="0.02" value="1.0" class="slider" id="s-shimmer"></td>
      </tr>
      <tr>
        <td>Glow leak intensity</td>
        <td><input type="range" min="0.0" max="1.0" step="0.02" value="0.4" class="slider" id="s-leak"></td>
      </tr>
    </table>
  </div>
</div>

> For pixel opacity use 252/255 for 100% glow and 253/255 for 40% glow.

<script>
const canvas = document.getElementById("glow-canvas");
const loadButton = document.getElementById('load-button');
const settingsGlow = document.getElementById('s-glow');
const settingsLeak = document.getElementById('s-leak');
const settingsShimmer = document.getElementById('s-shimmer');
const settingsAmbient = document.getElementById('s-ambient');

var image = new Image();
image.src = "favicon_glow.png";

const gl = canvas.getContext("webgl");

const fs = `
precision mediump float;

uniform sampler2D u_texture;
uniform vec2 u_tex_res;
uniform vec2 u_resolution;
uniform vec4 u_settings;
uniform float u_time;

varying float v_shimmer;

#define NL_GLOW_TEX u_settings.x
#define NL_GLOW_LEAK u_settings.y
#define NL_GLOW_SHIMMER u_settings.z
#define NL_AMBIENT u_settings.w

vec3 glowDetect(vec4 diffuse) {
  if (diffuse.a > 0.988 && diffuse.a < 0.993) {
    if (diffuse.a > 0.989) { return 0.4 * diffuse.rgb; }
    return diffuse.rgb;
  }
  return vec3(0.0,0.0,0.0);
}

vec3 glowDetectC(sampler2D tex, vec2 uv) {
  return glowDetect(texture2D(tex, uv, 0.0));
}

vec3 nlGlow(sampler2D tex, vec2 uv, float shimmer) {
  vec3 glow = glowDetectC(tex, uv);
  vec2 texSize = u_tex_res;
  vec2 offset = (1.0 / texSize);
  vec3 c1 = glowDetectC(tex, uv - offset);
  vec3 c2 = glowDetectC(tex, uv + offset*vec2(-1, 0));
  vec3 c3 = glowDetectC(tex, uv + offset*vec2(-1, 1));
  vec3 c4 = glowDetectC(tex, uv + offset*vec2( 0, 1));
  vec3 c5 = glowDetectC(tex, uv + offset);
  vec3 c6 = glowDetectC(tex, uv + offset*vec2( 1, 0));
  vec3 c7 = glowDetectC(tex, uv + offset*vec2( 1,-1));
  vec3 c8 = glowDetectC(tex, uv + offset*vec2( 0,-1));
  vec2 p = uv * texSize;
  vec2 u = fract(p);
  vec2 v = 1.0 - u;
  vec3 g = mix(mix(c1, c3, u.y), mix(c7, c5, u.y), u.x);
  g = max(g, max(max(c2*v.x, c4*u.y), max(c6*u.x, c8*v.y)));
  g = ((g*0.7 + 0.2)*g + 0.1)*g;
  glow = max(glow, g*NL_GLOW_LEAK);
  glow *= mix(1.0, (0.3 + 0.9*shimmer), NL_GLOW_SHIMMER);
  return glow * NL_GLOW_TEX;
}

void main() {
  float t = u_time*0.4;
  vec2 uv1 = gl_FragCoord.xy/u_resolution;
  uv1.y = 1.0-uv1.y;
  float count = (u_tex_res.y/u_tex_res.x);
  vec2 uv2 = uv1;
  uv1.y += floor(fract(t)*count);
  uv1.y /= count;
  float uvtmpy = ceil(fract(t)*count);
  if (uvtmpy<count) {
    uv2.y += uvtmpy;
  }
  uv2.y /= count;

  vec4 col1 = texture2D(u_texture, uv1);
  col1.rgb *= NL_AMBIENT;
  col1.rgb += nlGlow(u_texture, uv1, v_shimmer);

  vec4 col2 = texture2D(u_texture, uv2);
  col2.rgb *= NL_AMBIENT;
  col2.rgb += nlGlow(u_texture, uv2, v_shimmer);

  vec4 col = mix(col1, col2, fract(t*count));
  if (col.a < 0.5) { discard; }

  col.rgb /= (1.0 + col.rgb);
  col.rgb = pow(col.rgb, vec3(1.2));

  gl_FragColor = col;
}
`;
const vs = `
precision mediump float;

attribute vec4 a_position;

uniform float u_time;

varying float v_shimmer;

float nlGlowShimmer(vec3 cPos, float t) {
  float d = dot(cPos,vec3(1.0,1.0,1.0));
  float shimmer = sin(1.57*d + 0.7854*sin(d + 0.1*t) + 0.8*t);
  return shimmer * shimmer;
}

void main() {
  v_shimmer = nlGlowShimmer(a_position.xyz, u_time);
  gl_Position = a_position;
}
`;
const programInfo = twgl.createProgramInfo(gl, [vs, fs]);

const arrays = {
  a_position: [-1, -1, 0, 1, -1, 0, -1, 1, 0, -1, 1, 0, 1, -1, 0, 1, 1, 0]
};
const bufferInfo = twgl.createBufferInfoFromArrays(gl, arrays);
var texture = twgl.createTexture(gl, {src: "favicon_glow.png", mag: gl.NEAREST});

image.onload = function() {
  texture = twgl.createTexture(gl, {src: image, mag: gl.NEAREST, wrap: gl.CLAMP_TO_EDGE});
};

function render(time) {
  twgl.resizeCanvasToDisplaySize(gl.canvas);
  gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);

  const uniforms = {
    u_time: time * 0.001,
    u_resolution: [gl.canvas.width, gl.canvas.height],
    u_tex_res: [image.naturalWidth, image.naturalHeight],
    u_settings: [settingsGlow.value, settingsLeak.value, settingsShimmer.value, settingsAmbient.value],
    u_texture: texture
  };

  gl.useProgram(programInfo.program);
  twgl.setBuffersAndAttributes(gl, programInfo, bufferInfo);
  twgl.setUniforms(programInfo, uniforms);
  twgl.drawBufferInfo(gl, bufferInfo);

  requestAnimationFrame(render);
}
requestAnimationFrame(render);

loadButton.addEventListener('click', () => {
  const fileInput = document.createElement('input');
  fileInput.type = 'file';
  fileInput.accept = 'image/png';
  fileInput.onchange = (e) => {
    const uploadedImage = URL.createObjectURL(e.target.files[0]);
    image.src = uploadedImage;
  };
  fileInput.click();
});
</script>

