<script lang="ts">
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import {
		AmbientLight,
		DirectionalLight,
		WebGLRenderer,
		MeshStandardMaterial,
		Mesh,
		BoxGeometry,
		PerspectiveCamera,
		Scene
	} from 'three';

	const windowSize = writable({ width: window.innerWidth, height: window.innerHeight });

	let canvas: HTMLCanvasElement;

	function onResize() {
		windowSize.set({ width: window.innerWidth, height: window.innerHeight });
	}

	onMount(() => {
		const scene = new Scene();
		const camera = new PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
		camera.position.z = 5;

		const renderer = new WebGLRenderer({ canvas });

		const unsub = windowSize.subscribe(({ width, height }) => {
			camera.aspect = width / height;
			camera.updateProjectionMatrix();
			renderer.setSize(width, height);
			renderer.render(scene, camera);
		});

		const geometry = new BoxGeometry(1, 1, 1);
		const material = new MeshStandardMaterial({ color: 0xff2200 });
		const cube = new Mesh(geometry, material);
		scene.add(cube);

		const ambient = new AmbientLight(0xffffff, 0.5);
		scene.add(ambient);

		const light = new DirectionalLight(0xffffff, 0.5);
		light.position.set(5, 5, 3);
		light.target.position.set(0, 0, 0);
		scene.add(light);

		let running = true;
		function animate() {
			cube.rotation.x += 0.01;
			cube.rotation.y += 0.01;

			if (running) requestAnimationFrame(animate);
			renderer.render(scene, camera);
		}
		animate();

		return () => {
			running = false;
			scene.clear();
			renderer.dispose();
			unsub();
		};
	});
</script>

<canvas bind:this={canvas} />

<svelte:window on:resize={onResize} />
