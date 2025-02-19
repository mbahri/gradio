<script lang="ts">
	import type { FileData } from "@gradio/upload";
	import { BlockLabel, IconButton } from "@gradio/atoms";
	import { File, Download } from "@gradio/icons";
	import { _ } from "svelte-i18n";
	import { onMount } from "svelte";
	import * as BABYLON from "babylonjs";
	import * as BABYLON_LOADERS from "babylonjs-loaders";

	export let value: FileData | null;
	export let clearColor: [number, number, number, number] = [0, 0, 0, 0];
	export let label = "";
	export let show_label: boolean;

	BABYLON_LOADERS.OBJFileLoader.IMPORT_VERTEX_COLORS = true;

	let canvas: HTMLCanvasElement;
	let scene: BABYLON.Scene;
	let engine: BABYLON.Engine | null;

	let mounted = false;

	onMount(() => {
		engine = new BABYLON.Engine(canvas, true);
		window.addEventListener("resize", () => {
			engine?.resize();
		});
		mounted = true;
	});

	$: ({ data, name } = value || {
		data: undefined,
		name: undefined
	});

	$: canvas && mounted && data != null && name && dispose();

	function dispose(): void {
		if (scene && !scene.isDisposed) {
			scene.dispose();
			engine?.stopRenderLoop();
			engine?.dispose();
			engine = null;
			engine = new BABYLON.Engine(canvas, true);
			window.addEventListener("resize", () => {
				engine?.resize();
			});
		}
		addNewModel();
	}

	function addNewModel(): void {
		scene = new BABYLON.Scene(engine!);
		scene.createDefaultCameraOrLight();

		scene.clearColor = new BABYLON.Color4(...clearColor);

		engine?.runRenderLoop(() => {
			scene.render();
		});

		if (!value) return;

		let url: string;
		if (value.is_file) {
			url = value.data;
		} else {
			let base64_model_content = value.data;
			let raw_content = BABYLON.Tools.DecodeBase64(base64_model_content);
			let blob = new Blob([raw_content]);
			url = URL.createObjectURL(blob);
		}

		BABYLON.SceneLoader.ShowLoadingScreen = false;

		BABYLON.SceneLoader.Append(
			"",
			url,
			scene,
			() => {
				scene.createDefaultCamera(true, true, true);
			},
			undefined,
			undefined,
			"." + value["name"].split(".")[1]
		);
	}
</script>

<BlockLabel {show_label} Icon={File} label={label || $_("3D_model.3d_model")} />
{#if value}
	<div class="model3D">
		<div class="download">
			<a
				href={value.data}
				target={window.__is_colab__ ? "_blank" : null}
				download={window.__is_colab__ ? null : value.orig_name || value.name}
			>
				<IconButton Icon={Download} label={$_("common.download")} />
			</a>
		</div>

		<canvas bind:this={canvas} />
	</div>
{/if}

<style>
	.model3D {
		display: flex;
		position: relative;
		width: var(--size-full);
		height: var(--size-full);
	}
	canvas {
		width: var(--size-full);
		height: var(--size-full);
		object-fit: contain;
		overflow: hidden;
	}
	.download {
		position: absolute;
		top: 6px;
		right: 6px;
	}
</style>
