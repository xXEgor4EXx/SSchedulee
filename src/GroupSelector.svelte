<script>
	export let formEduData;
	export let selectedGroup;
	export let loadingSchedule;
	export let onGroupSelect;
	export let expandedForms = {};
	export let expandedCurs = {};
	export let toggleForm;
	export let toggleCurs;
</script>

<div class="tree-container">
	{#each formEduData as form}
		<div class="tree-node">
			<div class="folder" on:click={() => toggleForm(form.FormEdu_ID)}>
				<span class="arrow">{expandedForms[form.FormEdu_ID] ? '▼' : '▶'}</span>
				<span class="folder-name">{form.FormEduName}</span>
			</div>
			
			{#if expandedForms[form.FormEdu_ID]}
				<div class="nested">
					{#each form.arr as cursItem}
						<div class="tree-node">
							<div class="folder" on:click={() => toggleCurs(form.FormEdu_ID, cursItem.Curs)}>
								<span class="arrow">{expandedCurs[`${form.FormEdu_ID}_${cursItem.Curs}`] ? '▼' : '▶'}</span>
								<span class="folder-name">Курс {cursItem.Curs}</span>
							</div>
							
							{#if expandedCurs[`${form.FormEdu_ID}_${cursItem.Curs}`]}
								<div class="nested">
									{#each cursItem.arr as group}
										<div 
											class="group-item {selectedGroup?.GS_ID === group.GS_ID ? 'selected' : ''}"
											on:click={() => onGroupSelect(group)}
										>
											{group.GSName}
											{#if loadingSchedule && selectedGroup?.GS_ID === group.GS_ID}
												<span class="loading">Загрузка...</span>
											{/if}
										</div>
									{/each}
								</div>
							{/if}
						</div>
					{/each}
				</div>
			{/if}
		</div>
	{/each}
</div>

<style>
	.tree-container {
		border: 1px solid #ddd;
		border-radius: 8px;
		padding: 15px;
		background: white;
		box-shadow: 0 2px 4px rgba(0,0,0,0.1);
	}

	.tree-node {
		margin: 2px 0;
	}

	.folder {
		cursor: pointer;
		padding: 8px 5px;
		border-radius: 4px;
		display: flex;
		align-items: center;
		gap: 5px;
		transition: background-color 0.2s;
	}

	.folder:hover {
		background-color: #e9ecef;
	}

	.arrow {
		font-size: 12px;
		width: 15px;
		text-align: center;
		color: #666;
	}

	.folder-name {
		font-weight: 600;
		color: #333;
	}

	.nested {
		margin-left: 20px;
		border-left: 1px solid #e0e0e0;
		padding-left: 10px;
	}

	.group-item {
		cursor: pointer;
		padding: 6px 10px;
		border-radius: 4px;
		margin: 2px 0;
		transition: all 0.2s;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.group-item:hover {
		background-color: #e3f2fd;
	}

	.group-item.selected {
		background-color: #2196f3;
		color: white;
		font-weight: bold;
	}

	.loading {
		font-size: 12px;
		color: #666;
		font-style: italic;
	}

	/* Мобильные стили */
	@media (max-width: 500px) {
		.tree-container {
			padding: 10px;
			border: none;
			box-shadow: none;
		}

		.folder {
			padding: 12px 8px;
			font-size: 15px;
		}

		.folder-name {
			font-size: 15px;
		}

		.group-item {
			padding: 10px 12px;
			font-size: 14px;
			margin: 3px 0;
		}

		.arrow {
			font-size: 14px;
			width: 20px;
		}

		.nested {
			margin-left: 15px;
			padding-left: 8px;
		}
	}
</style>