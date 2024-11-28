<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import * as THREE from 'three';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
	import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
	import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';
	import spline from './spline';

	let canvasContainer: HTMLDivElement;
	let animationFrameId: number;

	onMount(() => {
		const w = window.innerWidth;
		const h = window.innerHeight;

		const scene = new THREE.Scene();
		scene.fog = new THREE.FogExp2(0x000000, 0.3);

		const camera = new THREE.PerspectiveCamera(75, w / h, 0.1, 1000);
		camera.position.z = 5;

		const renderer = new THREE.WebGLRenderer();
		renderer.setSize(w, h);
		renderer.toneMapping = THREE.ACESFilmicToneMapping;
		renderer.outputColorSpace = THREE.SRGBColorSpace;
		canvasContainer.appendChild(renderer.domElement);

		const controls = new OrbitControls(camera, renderer.domElement);
		controls.enableDamping = true;
		controls.dampingFactor = 0.03;

		const renderScene = new RenderPass(scene, camera);
		const bloomPass = new UnrealBloomPass(new THREE.Vector2(w, h), 1.5, 0.4, 100);
		bloomPass.threshold = 0.002;
		bloomPass.strength = 3.5;
		bloomPass.radius = 0;
		const composer = new EffectComposer(renderer);
		composer.addPass(renderScene);
		composer.addPass(bloomPass);

		const tubeGeo = new THREE.TubeGeometry(spline, 222, 0.65, 16, true);

		const edges = new THREE.EdgesGeometry(tubeGeo, 0.2);
		const lineMat = new THREE.LineBasicMaterial({ color: 0xffc0cb });
		const tubeLines = new THREE.LineSegments(edges, lineMat);
		scene.add(tubeLines);

		const numBoxes = 55;
		const size = 0.075;
		const boxGeo = new THREE.BoxGeometry(size, size, size);
		for (let i = 0; i < numBoxes; i += 1) {
			const boxMat = new THREE.MeshBasicMaterial({
				color: 0xffffff,
				wireframe: true
			});
			const p = (i / numBoxes + Math.random() * 0.1) % 1;
			const pos = tubeGeo.parameters.path.getPointAt(p);
			pos.x += Math.random() - 0.4;
			pos.z += Math.random() - 0.4;

			const rote = new THREE.Vector3(
				Math.random() * Math.PI,
				Math.random() * Math.PI,
				Math.random() * Math.PI
			);

			const edges = new THREE.EdgesGeometry(boxGeo, 0.2);
			const lineMat = new THREE.LineBasicMaterial({ color: 0x5c2fff });
			const boxLines = new THREE.LineSegments(edges, lineMat);
			boxLines.position.copy(pos);
			boxLines.rotation.set(rote.x, rote.y, rote.z);
			scene.add(boxLines);
		}

		function updateCamera(t: number) {
			const time = t * 0.1;
			const looptime = 4 * 1000;
			const p = (time % looptime) / looptime;
			const pos = tubeGeo.parameters.path.getPointAt(p);
			const lookAt = tubeGeo.parameters.path.getPointAt((p + 0.03) % 1);
			camera.position.copy(pos);
			camera.lookAt(lookAt);
		}

		function animate(t = 0) {
			animationFrameId = requestAnimationFrame(animate);
			updateCamera(t);
			composer.render(scene, camera);
			controls.update();
		}
		animate();

		function handleWindowResize() {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		}
		window.addEventListener('resize', handleWindowResize, false);

		return () => {
			window.removeEventListener('resize', handleWindowResize);
			cancelAnimationFrame(animationFrameId);
			composer.dispose();
			renderer.dispose();
			scene.dispose();
		};
	});
</script>

<div bind:this={canvasContainer} class="fixed inset-0 z-0"></div>

<style>
	:global(body) {
		margin: 0;
		overflow: hidden;
	}
</style>
