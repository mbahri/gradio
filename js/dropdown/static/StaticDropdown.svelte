<script lang="ts">
	import type { Gradio, SelectData } from "@gradio/utils";
	import Dropdown from "../shared";
	import { Block } from "@gradio/atoms";
	import { StatusTracker } from "@gradio/statustracker";
	import type { LoadingStatus } from "@gradio/statustracker";

	export let label = "Dropdown";
	export let info: string | undefined = undefined;
	export let elem_id = "";
	export let elem_classes: string[] = [];
	export let visible = true;
	export let value: string | string[];
	export let value_is_output = false;
	export let multiselect = false;
	export let max_choices: number;
	export let choices: string[];
	export let show_label: boolean;
	export let container = true;
	export let scale: number | null = null;
	export let min_width: number | undefined = undefined;
	export let loading_status: LoadingStatus;
	export let allow_custom_value = false;
	export let gradio: Gradio<{
		change: never;
		input: never;
		select: SelectData;
		blur: never;
		focus: never;
	}>;

	if (multiselect && !value) {
		value = [];
	} else if (!value) {
		value = "";
	}
</script>

<Block
	{visible}
	{elem_id}
	{elem_classes}
	padding={container}
	allow_overflow={false}
	{scale}
	{min_width}
>
	<StatusTracker {...loading_status} />

	<Dropdown
		bind:value
		bind:value_is_output
		{choices}
		{multiselect}
		{max_choices}
		{label}
		{info}
		{show_label}
		{allow_custom_value}
		{container}
		on:change={() => gradio.dispatch("change")}
		on:input={() => gradio.dispatch("input")}
		on:select={(e) => gradio.dispatch("select", e.detail)}
		on:blur={() => gradio.dispatch("blur")}
		on:focus={() => gradio.dispatch("focus")}
		disabled
	/>
</Block>
