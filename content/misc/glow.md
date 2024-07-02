+++
title = "Glow preview"
description = ""
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
var gl = canvas.getContext("webgl");

var vertexShader;
var fragmentShader;
var program;

var fragmentShaderSrc = `
precision mediump float;

uniform sampler2D u_image;
uniform vec2 u_tex_res;
uniform vec4 u_settings;
uniform float u_time;

varying vec2 v_texCoord;
varying float v_shimmer;

#define NL_GLOW_TEX u_settings.x
#define NL_GLOW_LEAK u_settings.y
#define NL_GLOW_SHIMMER u_settings.z
#define NL_AMBIENT u_settings.w

vec3 glowDetect(vec4 diffuse) {
  // Texture alpha: diffuse.a
  // 252/255 = max glow
  // 253/255 = partial glow
  if (diffuse.a > 0.988 && diffuse.a < 0.993) {
    if (diffuse.a > 0.989) {
      return 0.4 * diffuse.rgb;
    }
    return diffuse.rgb;
  }
  return vec3(0.0,0.0,0.0);
}

vec3 glowDetectC(sampler2D tex, vec2 uv) {
  return glowDetect(texture2D(tex, uv, 0.0));
}

vec3 nlGlow(sampler2D tex, vec2 uv, float shimmer) {
  vec3 glow = glowDetectC(tex, uv);

  #ifdef NL_GLOW_LEAK
  // glow leak is done by interpolating 8 surrounding pixels
  // c3 c4 c5
  // c2    c6
  // c1 c8 c7
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
  //u *= u*(3.0 - 2.0*u);
  vec2 v = 1.0 - u;

  // corners
  vec3 g = mix(mix(c1, c3, u.y), mix(c7, c5, u.y), u.x);

  /*
  g = max(
    max(c1*min(v.x,v.y), c3*min(v.x,u.y)),
    max(c5*min(u.x,u.y), c7*min(u.x,v.y))
  );*/

  // sides
  g = max(g, max(max(c2*v.x, c4*u.y), max(c6*u.x, c8*v.y)));

  // apply attuenation and add to glow
  g = ((g*0.7 + 0.2)*g + 0.1)*g;
  glow = max(glow, g*NL_GLOW_LEAK);
  #endif

  #ifdef NL_GLOW_SHIMMER
  glow *= mix(1.0, (0.3 + 0.9*shimmer), NL_GLOW_SHIMMER);
  #endif

  return glow * NL_GLOW_TEX;
}

void main() {
  vec2 uv = v_texCoord;
  float count = (u_tex_res.y / u_tex_res.x);
  uv.y += floor(fract(u_time)*count);
  uv.y /= count;

  vec4 col = texture2D(u_image, uv);
  if (col.a < 0.5) {
    discard;
  }

  col.rgb *= NL_AMBIENT;
  col.rgb += nlGlow(u_image, uv, v_shimmer);
  col.rgb /= (1.0 + col.rgb);
  col.rgb = pow(col.rgb, vec3(1.2));

  // col.rgb *= sin(u_time * 0.1);
  gl_FragColor = col;
}
`; 

var vertexShaderSrc = `
precision mediump float;

attribute vec2 a_position;
attribute vec2 a_texCoord;

uniform float u_time;

varying vec2 v_texCoord;
varying float v_shimmer;

float nlGlowShimmer(vec3 cPos, float t) {
  float d = dot(cPos,vec3(1.0,1.0,1.0));
  float shimmer = sin(1.57*d + 0.7854*sin(d + 0.1*t) + 0.8*t);
  return shimmer * shimmer;
}

void main() {
  vec2 pos = (a_position * 2.0) - 1.0;
  v_texCoord = a_texCoord;
  v_shimmer = nlGlowShimmer(vec3(a_position, 1.0), u_time);
  gl_Position = vec4(pos * vec2(1, -1), 0, 1);
}
`;

var positionLocation;
var positionBuffer;

var texcoordLocation;
var texcoordBuffer;

var resolutionLocation;
var texResLocation;
var timeLocation;
var settingsLocation;

var texture;

main();

function main() {
  if (!gl) {
    alert("Your browser does not support WebGL.");
    return;
  }

  vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSrc);
  fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSrc);
  program = createProgram(gl, vertexShader, fragmentShader);

  positionLocation = gl.getAttribLocation(program, "a_position");
  texcoordLocation = gl.getAttribLocation(program, "a_texCoord");

  positionBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
  var x1 = 0;
  var y1 = 0;
  var x2 = 1.0;
  var y2 = 1.0;
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
     x1, y1,
     x2, y1,
     x1, y2,
     x1, y2,
     x2, y1,
     x2, y2,
  ]), gl.STATIC_DRAW);

  texcoordBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, texcoordBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
      0.0,  0.0,
      1.0,  0.0,
      0.0,  1.0,
      0.0,  1.0,
      1.0,  0.0,
      1.0,  1.0,
  ]), gl.STATIC_DRAW);

  // Create a texture.
  texture = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, texture);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);

  resizeCanvasToDisplaySize(canvas);

  image.src = "favicon_glow.png";
  image.onload = function() {
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);
    render(gl, image);
  };
}

function createShader(gl, type, source) {
  var shader = gl.createShader(type);
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  var success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
  if (success) {
    return shader;
  }
 
  console.log(gl.getShaderInfoLog(shader));
  gl.deleteShader(shader);
}

function createProgram(gl, vertexShader, fragmentShader) {
  var program = gl.createProgram();

  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  gl.linkProgram(program);
  var success = gl.getProgramParameter(program, gl.LINK_STATUS);
  if (success) {
    return program;
  }
 
  console.log(gl.getProgramInfoLog(program));
  gl.deleteProgram(program);
}

loadButton.addEventListener('click', () => {
  const fileInput = document.createElement('input');
  fileInput.type = 'file';
  fileInput.accept = 'image/png';

  fileInput.onchange = (e) => {
    const uploadedImage = URL.createObjectURL(e.target.files[0]);
    image.id = 'image';
    image.src = uploadedImage;
    image.onload = function() {
        gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);
    }
  };

  fileInput.click();
});

function render(time) {
  resolutionLocation = gl.getUniformLocation(program, "u_resolution");
  texResLocation = gl.getUniformLocation(program, "u_tex_res");
  timeLocation = gl.getUniformLocation(program, "u_time");
  settingsLocation = gl.getUniformLocation(program, "u_settings");

  gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
  gl.clearColor(0, 0, 0, 0);
  gl.clear(gl.COLOR_BUFFER_BIT);

  gl.useProgram(program);

  gl.enableVertexAttribArray(positionLocation);
  gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
  var size = 2;
  var type = gl.FLOAT;
  var normalize = false;
  var stride = 0;
  var offset = 0;
  gl.vertexAttribPointer(positionLocation, size, type, normalize, stride, offset);

  gl.enableVertexAttribArray(texcoordLocation);
  gl.bindBuffer(gl.ARRAY_BUFFER, texcoordBuffer);
  var size = 2;
  var type = gl.FLOAT;
  var normalize = false;
  var stride = 0;
  var offset = 0;
  gl.vertexAttribPointer(texcoordLocation, size, type, normalize, stride, offset);

  gl.uniform2f(resolutionLocation, gl.canvas.width, gl.canvas.height);
  gl.uniform2f(texResLocation, image.naturalWidth, image.naturalHeight);
  gl.uniform1f(timeLocation, time * 0.001);
  gl.uniform4f(settingsLocation, settingsGlow.value, settingsLeak.value, settingsShimmer.value, settingsAmbient.value);

  var primitiveType = gl.TRIANGLES;
  var offset = 0;
  var count = 6;
  gl.drawArrays(primitiveType, offset, count);
  requestAnimationFrame(render);
}
// requestAnimationFrame(render);

function resizeCanvasToDisplaySize(canvas) {
  const displayWidth  = canvas.clientWidth;
  const displayHeight = canvas.clientHeight;
 
  const needResize = canvas.width  !== displayWidth || canvas.height !== displayHeight;
 
  if (needResize) {
    canvas.width  = displayWidth;
    canvas.height = displayHeight;
  }

  return needResize;
}

</script>

