<script lang="ts">
	import { onMount, onDestroy, createEventDispatcher } from 'svelte';
	import * as THREE from 'three';

	const dispatch = createEventDispatcher();

	// Props
	export let scene: THREE.Scene | null = null;
	export let message: string = ""; 
	export let position = { x: 0.2, y: 1.0, z: -3.3 };
	export let width = 2200;  
	export let height = 1000;  
	export let viewType: 'front-view' | 'side-view' = 'front-view'; 
	export let camera: THREE.PerspectiveCamera | null = null;  

	// Export du mesh pour la détection du clic par le parent
	export let boardMesh: THREE.Mesh | null = null;

	let animationFrame: number | null = null;
	let currentText = '';
	let targetText = '';
	let lastUpdateTime = 0;
	const CHAR_DELAY = 40;

	// Coordonnées du bouton sur le canvas (utilisées pour le Raycaster)
	export const buttonRect = { x: 850, y: 780, w: 500, h: 140 };

	const boardDimensions = {
		'front-view': { width: 3.6, height: 1.8 },
		'side-view': { width: 3.2, height: 1.6 }
	};

	function parseMessageContent(content: string): string {
		if (!content || content.trim() === "") {
			return "BIENVENUE DANS VOTRE CLASSE !\n\nL'aventure commence ici.\nPrêt à explorer de nouveaux horizons ?\n\nCliquez sur le bouton ci-dessous pour rejoindre.";
		}
		let text = content.replace(/```json/g, '').replace(/```/g, '');
		const match = text.match(/"response"\s*:\s*"([^"]+)"/);
		return match ? match[1] : text;
	}

	function createBoardText(displayText?: string) {
		if (!scene) return;
		const dims = boardDimensions[viewType];
		const canvas = document.createElement('canvas');
		canvas.width = width;
		canvas.height = height;
		const context = canvas.getContext('2d');
		if (!context) return;

		context.clearRect(0, 0, canvas.width, canvas.height);

		// --- STYLE DU TEXTE (EFFET CRAIE) ---
		context.shadowColor = 'rgba(255,255,255,0.6)';
		context.shadowBlur = 4;
		context.fillStyle = 'rgba(255, 255, 255, 0.98)';
		context.font = 'bold 80px "Segoe UI", Arial, sans-serif';
		context.textAlign = 'center';

		const textToDisplay = displayText || parseMessageContent(message);
		const paragraphs = textToDisplay.split('\n');
		
		let y = 180;
		const lineHeight = 110;
		const maxTextWidth = width - 400;

		for (const paragraph of paragraphs) {
			const words = paragraph.split(' ');
			let line = '';
			for(let n = 0; n < words.length; n++) {
				let testLine = line + words[n] + ' ';
				if (context.measureText(testLine).width > maxTextWidth && n > 0) {
					context.fillText(line, width/2, y);
					line = words[n] + ' ';
					y += lineHeight;
				} else { line = testLine; }
			}
			context.fillText(line, width/2, y);
			y += lineHeight;
		}

		// --- DESSIN DU BOUTON INTERACTIF ---
		// Ombre pour le relief (bien visible en capture N&B)
		context.shadowBlur = 20;
		context.shadowColor = 'rgba(0, 0, 0, 0.5)';
		context.fillStyle = '#4f46e5'; // Indigo
		
		roundRect(context, buttonRect.x, buttonRect.y, buttonRect.w, buttonRect.h, 70, true);
		
		// Texte du bouton
		context.shadowBlur = 0;
		context.fillStyle = 'white';
		context.font = 'bold 55px Arial';
		context.textBaseline = 'middle';
		context.fillText("REJOINDRE LA CLASSE", width/2, buttonRect.y + (buttonRect.h / 2));

		const texture = new THREE.CanvasTexture(canvas);
		const material = new THREE.MeshBasicMaterial({ 
			map: texture, 
			transparent: true, 
			side: THREE.DoubleSide, 
			alphaTest: 0.05 
		});
		
		const geometry = new THREE.PlaneGeometry(dims.width, dims.height);
		
		if (boardMesh) scene.remove(boardMesh);
		
		boardMesh = new THREE.Mesh(geometry, material);
		boardMesh.position.set(position.x, position.y, position.z + 0.02);
		boardMesh.rotation.set(0, viewType === 'front-view' ? 0 : -Math.PI/2, 0);
		
		// Identifiant pour le Raycaster
		boardMesh.userData = { isBoard: true };
		
		scene.add(boardMesh);
	}

	function roundRect(ctx: CanvasRenderingContext2D, x: number, y: number, w: number, h: number, r: number, fill: boolean) {
		ctx.beginPath();
		ctx.moveTo(x + r, y);
		ctx.lineTo(x + w - r, y);
		ctx.quadraticCurveTo(x + w, y, x + w, y + r);
		ctx.lineTo(x + w, y + h - r);
		ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h);
		ctx.lineTo(x + r, y + h);
		ctx.quadraticCurveTo(x, y + h, x, y + h - r);
		ctx.lineTo(x, y + r);
		ctx.quadraticCurveTo(x, y, x + r, y);
		ctx.closePath();
		if (fill) ctx.fill();
	}

	function animateText(timestamp: number) {
		if (!lastUpdateTime) lastUpdateTime = timestamp;
		if (timestamp - lastUpdateTime >= CHAR_DELAY && currentText !== targetText) {
			currentText = targetText.slice(0, currentText.length + 1);
			lastUpdateTime = timestamp;
			createBoardText(currentText);
		}
		if (currentText !== targetText) animationFrame = requestAnimationFrame(animateText);
	}

	function startTextAnimation(newMessage: string) {
		if (animationFrame !== null) cancelAnimationFrame(animationFrame);
		targetText = parseMessageContent(newMessage);
		currentText = '';
		lastUpdateTime = 0;
		animationFrame = requestAnimationFrame(animateText);
	}

	onMount(() => {
		if (scene) startTextAnimation(message);
		return () => {
			if (animationFrame) cancelAnimationFrame(animationFrame);
			if (boardMesh) scene?.remove(boardMesh);
		};
	});

	$: if (message && scene) startTextAnimation(message);
</script>