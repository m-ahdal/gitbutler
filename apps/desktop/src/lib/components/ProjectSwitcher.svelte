<script lang="ts">
	import { ProjectsService, Project } from '$lib/backend/projects';
	import OptionsGroup from '$lib/select/OptionsGroup.svelte';
	import Select from '$lib/select/Select.svelte';
	import SelectItem from '$lib/select/SelectItem.svelte';
	import { getContext, maybeGetContext } from '@gitbutler/shared/context';
	import Button from '@gitbutler/ui/Button.svelte';
	import { goto } from '$app/navigation';

	const projectsService = getContext(ProjectsService);
	const project = maybeGetContext(Project);

	const projects = $derived(projectsService.projects);

	const mappedProjects = $derived(
		$projects?.map((project) => ({
			value: project.id,
			label: project.title
		})) || []
	);

	let selectedProjectId: string | undefined = $state(project ? project.id : undefined);

	let newProjectLoading = $state(false);
	let cloneProjectLoading = $state(false);
</script>

<div class="project-switcher">
	<Select
		value={selectedProjectId}
		options={mappedProjects}
		label="Switch to another project"
		wide
		onselect={(value) => {
			selectedProjectId = value;
		}}
		searchable
	>
		{#snippet itemSnippet({ item, highlighted })}
			<SelectItem selected={item.value === selectedProjectId} {highlighted}>
				{item.label}
			</SelectItem>
		{/snippet}

		<OptionsGroup>
			<SelectItem
				icon="plus"
				loading={newProjectLoading}
				on:click={async () => {
					newProjectLoading = true;
					try {
						await projectsService.addProject();
					} finally {
						newProjectLoading = false;
					}
				}}
			>
				Add local repository
			</SelectItem>
			<SelectItem
				icon="clone"
				loading={cloneProjectLoading}
				on:click={async () => {
					cloneProjectLoading = true;
					try {
						goto('/onboarding/clone');
					} finally {
						cloneProjectLoading = false;
					}
				}}
			>
				Clone repository
			</SelectItem>
		</OptionsGroup>
	</Select>

	<Button
		style="pop"
		kind="solid"
		icon="chevron-right-small"
		disabled={selectedProjectId === project?.id}
		onmousedown={() => {
			if (selectedProjectId) goto(`/${selectedProjectId}/`);
		}}
	>
		Open project
	</Button>
</div>

<style lang="postcss">
	.project-switcher {
		display: flex;
		flex-direction: column;
		gap: 10px;
		align-items: flex-end;
	}
</style>
