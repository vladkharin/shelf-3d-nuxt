<template>
  <div ref="mount" class="scene" role="img" aria-label="Трёхмерная модель стеллажа. Перетаскивайте для вращения, прокручивайте для приближения.">
    <div v-if="failed" class="scene-fallback"><strong>3D-просмотр недоступен</strong><p>Попробуйте браузер с поддержкой WebGL. Настройки и скачивание конфигурации остаются доступны.</p></div>
  </div>
</template>
<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
const props = defineProps<{width:number;height:number;depth:number;rows:number;cols:number;color:string}>();
const mount = ref<HTMLDivElement|null>(null);
const failed = ref(false);
let scene:THREE.Scene, camera:THREE.PerspectiveCamera, renderer:THREE.WebGLRenderer, controls:OrbitControls;
let shelf:THREE.Group|undefined, resize:ResizeObserver|undefined;
let ready=false;
const render=()=>{if(ready) renderer.render(scene,camera);};
function disposeGroup(group:THREE.Object3D){
  const materials=new Set<THREE.Material>();
  group.traverse(object=>{if(object instanceof THREE.Mesh){object.geometry.dispose();const list=Array.isArray(object.material)?object.material:[object.material];list.forEach(m=>materials.add(m));}});
  materials.forEach(m=>m.dispose());
}
function buildShelf(){
  if(!ready)return;
  if(shelf){scene.remove(shelf);disposeGroup(shelf);}
  shelf=new THREE.Group();
  const {width:w,height:h,depth:d,rows,cols}=props;
  const t=.018;
  const mat=new THREE.MeshStandardMaterial({color:props.color,roughness:.65,metalness:0});
  const panel=(width:number,height:number,depth:number,x:number,y:number,z:number)=>{
    const mesh=new THREE.Mesh(new THREE.BoxGeometry(width,height,depth),mat);
    mesh.position.set(x,y,z);mesh.castShadow=true;mesh.receiveShadow=true;shelf!.add(mesh);
  };
  panel(t,h,d,-w/2+t/2,h/2,0);panel(t,h,d,w/2-t/2,h/2,0);
  panel(w-2*t,t,d,0,t/2,0);panel(w-2*t,t,d,0,h-t/2,0);
  for(let i=1;i<rows;i++)panel(w-2*t,t,d,0,i*h/rows,0);
  for(let j=1;j<cols;j++)panel(t,h-2*t,d,-w/2+j*w/cols,h/2,0);
  panel(w-2*t,h-2*t,.006,0,h/2,-d/2+.003);
  scene.add(shelf);render();
}
function fit(){
  if(!ready||!mount.value)return;
  const w=mount.value.clientWidth,h=mount.value.clientHeight;if(!w||!h)return;
  camera.aspect=w/h;camera.updateProjectionMatrix();renderer.setSize(w,h,false);
  const target=new THREE.Vector3(0,props.height/2,0);
  const verticalFov=THREE.MathUtils.degToRad(camera.fov);
  const horizontalFov=2*Math.atan(Math.tan(verticalFov/2)*camera.aspect);
  const distance=Math.max(props.height/2/Math.tan(verticalFov/2),props.width/2/Math.tan(horizontalFov/2))*1.65+props.depth;
  camera.position.copy(target).add(new THREE.Vector3(.9,.38,1.6).normalize().multiplyScalar(distance));
  controls.target.copy(target);controls.minDistance=distance*.45;controls.maxDistance=distance*2.5;controls.update();render();
}
function snapshot(){
  if(!ready)return false;render();
  const a=document.createElement('a');a.download='form-shelf.png';a.href=renderer.domElement.toDataURL('image/png');a.click();return true;
}
defineExpose({snapshot,resetView:fit});
onMounted(()=>{
  if(!mount.value)return;
  try{
    scene=new THREE.Scene();scene.background=new THREE.Color('#eae7e0');
    camera=new THREE.PerspectiveCamera(35,1,.05,100);
    renderer=new THREE.WebGLRenderer({antialias:true,preserveDrawingBuffer:true});
    renderer.setPixelRatio(Math.min(devicePixelRatio,2));renderer.shadowMap.enabled=true;renderer.shadowMap.type=THREE.PCFSoftShadowMap;renderer.outputColorSpace=THREE.SRGBColorSpace;renderer.toneMapping=THREE.ACESFilmicToneMapping;renderer.toneMappingExposure=1.2;
    mount.value.append(renderer.domElement);
    scene.add(new THREE.HemisphereLight('#ffffff','#d6cfc0',2.5));
    const light=new THREE.DirectionalLight('#fff5e8',4);light.position.set(-3,5,4);light.castShadow=true;light.shadow.mapSize.set(2048,2048);light.shadow.camera.left=-4;light.shadow.camera.right=4;light.shadow.camera.top=5;light.shadow.camera.bottom=-3;light.shadow.normalBias=.025;light.shadow.bias=-.0003;scene.add(light);
    const fill=new THREE.DirectionalLight('#f3f6ff',1);fill.position.set(4,2,-2);scene.add(fill);
    const floor=new THREE.Mesh(new THREE.PlaneGeometry(200,200),new THREE.MeshStandardMaterial({color:'#eae7e0',roughness:1}));floor.rotation.x=-Math.PI/2;floor.position.y=-.005;floor.receiveShadow=true;scene.add(floor);
    scene.fog=new THREE.Fog('#eae7e0',15,40);
    controls=new OrbitControls(camera,renderer.domElement);controls.enablePan=false;controls.enableDamping=false;controls.maxPolarAngle=Math.PI/2-.03;controls.minPolarAngle=.1;controls.addEventListener('change',render);
    ready=true;buildShelf();resize=new ResizeObserver(fit);resize.observe(mount.value);fit();
  }catch{failed.value=true;renderer?.dispose();}
});
watch(()=>[props.width,props.height,props.depth,props.rows,props.cols,props.color],()=>{buildShelf();});
watch(()=>[props.width,props.height,props.depth],()=>{fit();});
onUnmounted(()=>{ready=false;resize?.disconnect();controls?.dispose();if(scene)disposeGroup(scene);renderer?.dispose();renderer?.domElement.remove();});
</script>
<style scoped>
.scene{height:100%;width:100%;position:relative;min-height:400px}.scene :deep(canvas){width:100%;height:100%;display:block;touch-action:none}.scene-fallback{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;text-align:center;padding:45px;gap:15px}.scene-fallback p{max-width:350px;color:#77766a;font-size:13px;line-height:1.8}@media(max-width:700px){.scene{min-height:420px}}
</style>
