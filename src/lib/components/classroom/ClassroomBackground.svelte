<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import * as THREE from 'three';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
  import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
  import { CSS3DRenderer } from 'three/examples/jsm/renderers/CSS3DRenderer.js';
  import BoardText from './BoardText.svelte';

  export let classroomModel: 'default' | 'alternative' = 'default';
  export let scene: THREE.Scene | null = null;
  export let camera: THREE.PerspectiveCamera | null = null;
  export let boardMessage: string = "";

  let classroom: THREE.Group | null = null;
  let cssRenderer: CSS3DRenderer | null = null;
  let container: HTMLElement;
  let boardMeshRef: THREE.Mesh | null = null;

  // --- STATISTIQUES POUR L'ENSEIGNANT (Option 3) ---
  let showStats = false;
  let startTime = Date.now();
  let elapsedTime = "00:00";
  let interactionCount = 0;
  
  // Simulation de calcul de motivation (Basé sur les interactions)
  $: motivationScore = Math.min(100, interactionCount * 20);

  function toggleStats() {
    showStats = !showStats;
  }

  // Mise à jour du chrono
  setInterval(() => {
    const totalSeconds = Math.floor((Date.now() - startTime) / 1000);
    const mins = Math.floor(totalSeconds / 60).toString().padStart(2, '0');
    const secs = (totalSeconds % 60).toString().padStart(2, '0');
    elapsedTime = `${mins}:${secs}`;
  }, 1000);

  const raycaster = new THREE.Raycaster();
  const mouse = new THREE.Vector2();

  function onMouseDown(event: MouseEvent) {
    if (!camera || !boardMeshRef) return;
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObject(boardMeshRef);

    if (intersects.length > 0) {
      const uv = intersects[0].uv;
      if (uv) {
        const canvasX = uv.x * 2200;
        const canvasY = (1 - uv.y) * 1000;
        if (canvasX >= 850 && canvasX <= 1350 && canvasY >= 780 && canvasY <= 920) {
          interactionCount++; // On compte chaque clic réussi
          alert("Interaction enregistrée dans le journal.");
        }
      }
    }
  }

  onMount(() => {
    if (scene && container) {
      cssRenderer = new CSS3DRenderer();
      cssRenderer.setSize(window.innerWidth, window.innerHeight);
      cssRenderer.domElement.style.position = 'absolute';
      cssRenderer.domElement.style.top = '0';
      cssRenderer.domElement.style.left = '0';
      cssRenderer.domElement.style.pointerEvents = 'none';
      cssRenderer.domElement.style.zIndex = '10';
      container.appendChild(cssRenderer.domElement);
      window.addEventListener('mousedown', onMouseDown);
      loadClassroom();
      return () => window.removeEventListener('mousedown', onMouseDown);
    }
  });

  // Tes fonctions existantes (loadClassroom, getBoardPosition, etc.)
  // ... [Garder le reste du code identique] ...

</script>

<div class="teacher-tools">
    <button on:click={toggleStats} class="stats-btn">
        📊 {showStats ? 'Fermer les stats' : 'Journal de bord (Enseignant)'}
    </button>

    {#if showStats}
    <div class="stats-panel">
        <h3 class="title">Suivi de la séance - Bechi Nissrine</h3>
        <hr/>
        <div class="stat-item">
            <span>⏱ Temps écoulé :</span>
            <strong>{elapsedTime}</strong>
        </div>
        <div class="stat-item">
            <span>🖱 Interactions :</span>
            <strong>{interactionCount} clics</strong>
        </div>
        <div class="stat-item">
            <span>🔥 Score de Motivation :</span>
            <div class="progress-bar">
                <div class="fill" style="width: {motivationScore}%"></div>
            </div>
            <strong>{motivationScore}%</strong>
        </div>
        <p class="hint">Analyse en temps réel pour pédagogie active.</p>
    </div>
    {/if}
</div>

<div bind:this={container} class="classroom-container"></div>

{#if scene}
  <BoardText 
    {scene} 
    {camera}
    bind:boardMesh={boardMeshRef}
    message={boardMessage} 
    position={getBoardPosition()} 
    viewType={classroomModel === 'default' ? 'front-view' : 'side-view'}
  />
{/if}

<style>
  .classroom-container { position: relative; width: 100%; height: 100%; }

  .teacher-tools {
    position: absolute;
    top: 20px;
    right: 20px;
    z-index: 100;
  }

  .stats-btn {
    background: black;
    color: white;
    padding: 10px 20px;
    border: 3px solid white;
    font-weight: bold;
    cursor: pointer;
    box-shadow: 5px 5px 0px black;
  }

  .stats-panel {
    margin-top: 15px;
    background: white;
    border: 4px solid black;
    padding: 20px;
    width: 300px;
    box-shadow: 10px 10px 0px rgba(0,0,0,0.2);
  }

  .title { margin: 0 0 10px 0; font-size: 1.1rem; text-transform: uppercase; }

  .stat-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 10px 0;
    font-family: monospace;
  }

  .progress-bar {
    width: 60px;
    height: 10px;
    background: #ddd;
    border: 1px solid black;
  }

  .fill { height: 100%; background: black; transition: width 0.3s; }

  .hint { font-size: 0.7rem; font-style: italic; color: #666; margin-top: 15px; }
</style>