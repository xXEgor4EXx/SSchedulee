<!-- app.svelte -->
<script lang="ts">
	import { onMount, tick } from 'svelte';
	import { GetSchedule, GetGroupSchedule } from './Resource.js';
	import GroupSelector from './GroupSelector.svelte';
	
	let formEduData = $state([]);
	let selectedGroup = $state(null);
	let groupSchedule = $state(null);
	let loading = $state(true);
	let loadingSchedule = $state(false);
	let error = $state(null);

	let todayDate = $state('');
	let scheduleContainer = $state(null);
	let shouldScrollToToday = $state(false);
	
	let expandedForms = $state({});
	let expandedCurs = $state({});
	
	let showGroupSelector = $state(true);
	let drawerOpen = $state(false);

	onMount(async () => {
		try {
			const data = await GetSchedule();
			formEduData = data.FormEdu;
			loading = false;
		} catch (err) {
			error = err.message;
			loading = false;
		}
		const today = new Date();
		const day = String(today.getDate()).padStart(2, '0');
		const month = String(today.getMonth() + 1).padStart(2, '0');
		const year = today.getFullYear();
		todayDate = `${day}.${month}.${year}`;
	});

	function toggleForm(formId) {
		expandedForms[formId] = !expandedForms[formId];
	}

	function toggleCurs(formId, curs) {
		const key = `${formId}_${curs}`;
		expandedCurs[key] = !expandedCurs[key];
	}

	function openDrawer() {
		drawerOpen = true;
	}

	function closeDrawer() {
		drawerOpen = false;
	}

	async function onGroupSelect(group) {
		selectedGroup = group;
		loadingSchedule = true;
		groupSchedule = null;
		shouldScrollToToday = true;
		
		try {
			const schedule = await GetGroupSchedule(group.GS_ID);
			groupSchedule = schedule;
			console.log('Расписание группы:', schedule);
			
			// Закрываем Drawer после успешного выбора
			showGroupSelector = false;
			drawerOpen = false;
		} catch (err) {
			error = err.message;
			groupSchedule = null;
		} finally {
			loadingSchedule = false;
			if (shouldScrollToToday) {
				await tick();
				scrollToToday();
			}
		}
	}

	function changeGroup() {
		// Функция для смены группы
		showGroupSelector = true;
		drawerOpen = true;
	}

	// Функция для прокрутки к сегодняшнему дню
	function scrollToToday() {
		if (!scheduleContainer) return;
		
		const todayElement = scheduleContainer.querySelector('.day-section.today');
		if (todayElement) {
			todayElement.scrollIntoView({ 
				behavior: 'smooth', 
				block: 'center' 
			});
		}
	}
</script>

<div class="app-container">
	{#if showGroupSelector}
		<!-- Welcome Screen -->
		<div class="welcome-screen">
			<div class="welcome-content">
				<h1>Расписание студенческих групп</h1>
				<button class="select-group-btn" on:click={openDrawer}>
					Выберите группу
				</button>
			</div>
		</div>
	{:else}
		<!-- Schedule Screen -->
		<div class="schedule-screen">
			<div class="schedule-header">
				<h2>Расписание: {selectedGroup?.GSName}</h2>
				<button class="change-group-btn" on:click={changeGroup}>
					Сменить группу
				</button>
			</div>
			
			{#if loadingSchedule}
    <p>Загрузка расписания...</p>
{:else if groupSchedule}
    <div class='schedule-info' bind:this={scheduleContainer}>
        {#each groupSchedule.Month as month}
            <div class='month-section'>
                <h4 class='month-title'>{month.Name} {month.Numb}</h4>
                
                <!-- СТАТИСТИКА МЕСЯЦА -->
                <div class='month-stats'>
                    <span class='stats-item'>Дней: {month.Sched.length}</span>
                    <span class='stats-item'>Пар: {month.Sched.reduce((total, day) => total + (day.mainSchedule ? day.mainSchedule.length : 0), 0)}</span>
                </div>
                
                {#each month.Sched as day}
                    <div class='day-section {day.datePair === todayDate ? "today" : ""}'>
                        <div class='day-header'>
                            <span class='date'>{day.datePair}</span>
                            <span class='day-week'>{day.dayWeek}</span>
                            {#if day.datePair === todayDate}
                                <span class="today-badge">Сегодня</span>
                            {/if}
                        </div>
                        
                        {#if day.mainSchedule && day.mainSchedule.length > 0}
                            <div class='lessons'>
                                {#each day.mainSchedule as lesson, index}
                                    <div class='lesson'>
                                        <div class='lesson-time'>
                                            {lesson.TimeStart}
                                            {#if lesson.TimeEnd}
                                                - {lesson.TimeEnd}
                                            {/if}
                                        </div>
                                        <div class='lesson-content'>
                                            <div class='subject'>{lesson.SubjName}</div>
                                            <div class='lesson-details'>
                                                <span class='type'>{lesson.LoadKindSN}</span>
                                                <span class='teacher'>{lesson.FIO}</span>
                                                <span class='audience'>{lesson.Aud}</span>
                                            </div>
                                        </div>
                                    </div>
                                {/each}
                            </div>
                        {:else}
                            <div class='no-lessons'>Нет занятий</div>
                        {/if}
                    </div>
                {/each}
            </div>
        {/each}
    </div>
{:else}
    <p>Расписание не найдено или произошла ошибка</p>
{/if}
		</div>
	{/if}

	<!-- Drawer Overlay -->
	{#if drawerOpen}
		<div class="drawer-overlay" on:click={closeDrawer}>
			<div class="drawer-content" on:click|stopPropagation>
				<div class="drawer-header">
					<h3>Выберите группу</h3>
					<button class="close-drawer" on:click={closeDrawer}>×</button>
				</div>
				
				<div class="drawer-body">
					{#if loading}
						<p>Загрузка данных...</p>
					{:else if error}
						<p class="error">Ошибка: {error}</p>
					{:else}
						<GroupSelector 
							{formEduData}
							{selectedGroup}
							{loadingSchedule}
							{onGroupSelect}
							{expandedForms}
							{expandedCurs}
							{toggleForm}
							{toggleCurs}
						/>
					{/if}
				</div>
			</div>
		</div>
	{/if}
</div>

<style>
	:global(body) {
		margin: 0;
		font-family: Arial, sans-serif;
		overflow-x: hidden;
	}

	.app-container {
		min-height: 100vh;
		position: relative;
	}

	/* Welcome Screen Styles */
	.welcome-screen {
		display: flex;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		color: white;
	}

	.welcome-content {
		text-align: center;
		padding: 2rem;
	}

	.welcome-content h1 {
		font-size: 2.5rem;
		margin-bottom: 2rem;
		font-weight: 300;
	}

	.select-group-btn {
		background: rgba(255, 255, 255, 0.2);
		color: white;
		border: 2px solid white;
		padding: 12px 30px;
		font-size: 1.1rem;
		border-radius: 25px;
		cursor: pointer;
		transition: all 0.3s ease;
		backdrop-filter: blur(10px);
	}

	.select-group-btn:hover {
		background: white;
		color: #667eea;
		transform: translateY(-2px);
		box-shadow: 0 10px 20px rgba(0,0,0,0.2);
	}

	/* Schedule Screen Styles */
	.schedule-screen {
		padding: 20px;
		background: #f5f5f5;
		min-height: 100vh;
	}

	.schedule-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20px;
		padding: 0 10px;
	}

	.schedule-header h2 {
		margin: 0;
		color: #333;
	}

	.change-group-btn {
		background: #4CAF50;
		color: white;
		border: none;
		padding: 8px 16px;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.2s;
	}

	.change-group-btn:hover {
		background: #45a049;
	}

	/* Drawer Styles */
	.drawer-overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.5);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 1000;
		animation: fadeIn 0.3s ease;
	}

	.drawer-content {
		background: white;
		border-radius: 12px;
		width: 90%;
		max-width: 500px;
		max-height: 80vh;
		display: flex;
		flex-direction: column;
		box-shadow: 0 20px 40px rgba(0,0,0,0.3);
		animation: slideUp 0.3s ease;
		overflow: hidden;
	}

	.drawer-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20px;
		border-bottom: 1px solid #eee;
		background: #f8f9fa;
	}

	.drawer-header h3 {
		margin: 0;
		color: #333;
	}

	.close-drawer {
		background: none;
		border: none;
		font-size: 24px;
		cursor: pointer;
		color: #666;
		padding: 0;
		width: 30px;
		height: 30px;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 50%;
		transition: background-color 0.2s;
	}

	.close-drawer:hover {
		background: #e9ecef;
		color: #333;
	}

	.drawer-body {
		flex: 1;
		overflow-y: auto;
		padding: 20px;
	}

	/* Анимации */
	@keyframes fadeIn {
		from { opacity: 0; }
		to { opacity: 1; }
	}

	@keyframes slideUp {
		from { 
			opacity: 0;
			transform: translateY(30px) scale(0.95);
		}
		to { 
			opacity: 1;
			transform: translateY(0) scale(1);
		}
	}

	/* Стили для таблицы расписания */
	.schedule-info {
		max-height: 600px;
		overflow-y: auto;
	}

	.month-section {
		margin-bottom: 30px;
		border: 1px solid #e0e0e0;
		border-radius: 8px;
		overflow: hidden;
	}

	.month-title {
		background: #4CAF50;
		color: white;
		padding: 12px 15px;
		margin: 0;
		font-size: 18px;
	}

	.day-section {
		border-bottom: 1px solid #f0f0f0;
	}

	.day-section:last-child {
		border-bottom: none;
	}

	.day-header {
		background: #f8f9fa;
		padding: 10px 15px;
		display: flex;
		justify-content: space-between;
		font-weight: bold;
		border-bottom: 1px solid #e9ecef;
	}

	.date {
		color: #495057;
	}

	.day-week {
		color: #6c757d;
	}

	.lessons {
		padding: 10px;
	}

	.lesson {
		display: flex;
		margin-bottom: 15px;
		padding: 12px;
		background: white;
		border: 1px solid #dee2e6;
		border-radius: 6px;
		transition: box-shadow 0.2s;
	}

	.lesson:hover {
		box-shadow: 0 2px 4px rgba(0,0,0,0.1);
	}

	.lesson-time {
		flex: 0 0 80px;
		font-weight: bold;
		color: #495057;
		font-size: 14px;
		display: flex;
		align-items: flex-start;
	}

	.lesson-content {
		flex: 1;
	}

	.subject {
		font-weight: 600;
		color: #212529;
		margin-bottom: 5px;
		font-size: 15px;
	}

	.lesson-details {
		display: flex;
		flex-wrap: wrap;
		gap: 10px;
		font-size: 13px;
		color: #6c757d;
	}

	.type {
		background: #e3f2fd;
		color: #1976d2;
		padding: 2px 6px;
		border-radius: 3px;
	}

	.teacher {
		color: #d81b60;
	}

	.audience {
		color: #388e3c;
		font-weight: 500;
	}

	.no-lessons {
		padding: 20px;
		text-align: center;
		color: #6c757d;
		font-style: italic;
	}

	.schedule-info::-webkit-scrollbar {
		width: 6px;
	}

	.schedule-info::-webkit-scrollbar-track {
		background: #f1f1f1;
	}

	.schedule-info::-webkit-scrollbar-thumb {
		background: #c1c1c1;
		border-radius: 3px;
	}

	.schedule-info::-webkit-scrollbar-thumb:hover {
		background: #a8a8a8;
	}

	/* Стили для выделения сегодняшнего дня */
	.day-section.today {
		border-left: 4px solid #4CAF50;
		background-color: #f8fff8;
		margin-left: -4px;
	}

	.day-section.today .day-header {
		background: #e8f5e9;
		color: #2e7d32;
		font-weight: bold;
	}

	.today-badge {
		background: #4CAF50;
		color: white;
		padding: 2px 8px;
		border-radius: 12px;
		font-size: 12px;
		font-weight: bold;
		margin-left: 10px;
	}

	/* Улучшенная прокрутка */
	.schedule-info {
		scroll-behavior: smooth;
	}

	.error {
		color: #d32f2f;
		text-align: center;
		padding: 20px;
	}

	.month-stats {
    background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
    padding: 10px 15px;
    display: flex;
    gap: 25px;
    border-bottom: 1px solid #90caf9;
    font-size: 14px;
	}

.stats-item {
    color: #0d47a1;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 5px;
}

.stats-item:before {
    font-size: 12px;
    background: #2196f3;
    color: white;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
}
	/* Мобильные стили */
	@media (max-width: 500px) {
		/* Welcome Screen для мобильных */
		.welcome-screen {
			padding: 1rem;
		}

		.welcome-content h1 {
			font-size: 1.8rem;
			margin-bottom: 1.5rem;
			line-height: 1.3;
		}

		.select-group-btn {
			padding: 15px 25px;
			font-size: 1rem;
			width: 100%;
			max-width: 280px;
		}

		/* Schedule Screen для мобильных */
		.schedule-screen {
			padding: 10px;
		}

		.schedule-header {
			flex-direction: column;
			gap: 15px;
			align-items: flex-start;
			margin-bottom: 15px;
		}

		.schedule-header h2 {
			font-size: 1.3rem;
			line-height: 1.4;
		}

		.change-group-btn {
			width: 100%;
			padding: 12px;
			font-size: 1rem;
		}

		/* Расписание для мобильных */
		.schedule-info {
			max-height: none;
			overflow-y: visible;
		}

		.month-title {
			padding: 10px 12px;
			font-size: 16px;
		}

		.day-header {
			padding: 8px 12px;
			flex-direction: column;
			align-items: flex-start;
			gap: 5px;
		}

		.day-header .date,
		.day-header .day-week {
			font-size: 14px;
		}

		.today-badge {
			font-size: 11px;
			padding: 3px 8px;
			margin-left: 0;
			align-self: flex-start;
		}

		.lesson {
			flex-direction: column;
			padding: 10px;
			margin-bottom: 10px;
		}

		.lesson-time {
			flex: none;
			width: 100%;
			margin-bottom: 8px;
			padding-bottom: 5px;
			border-bottom: 1px solid #e0e0e0;
			font-size: 13px;
		}

		.lesson-content {
			width: 100%;
		}

		.subject {
			font-size: 14px;
			margin-bottom: 8px;
			line-height: 1.3;
		}

		.lesson-details {
			flex-direction: column;
			gap: 5px;
			font-size: 12px;
		}

		.lesson-details span {
			padding: 3px 6px;
			border-radius: 3px;
		}

		.type {
			background: #e3f2fd;
			color: #1976d2;
			align-self: flex-start;
		}

		.teacher {
			color: #d81b60;
			background: #fce4ec;
			align-self: flex-start;
		}

		.audience {
			color: #388e3c;
			background: #e8f5e8;
			align-self: flex-start;
			font-weight: 500;
		}

		.no-lessons {
			padding: 15px;
			font-size: 14px;
		}

		/* Drawer для мобильных */
		.drawer-overlay {
			padding: 10px;
			align-items: flex-end;
		}

		.drawer-content {
			width: 100%;
			max-width: none;
			max-height: 85vh;
			border-radius: 16px 16px 0 0;
			animation: slideUpMobile 0.3s ease;
		}

		.drawer-header {
			padding: 15px 20px;
		}

		.drawer-header h3 {
			font-size: 1.2rem;
		}

		.drawer-body {
			padding: 15px;
			max-height: 60vh;
		}

		@keyframes slideUpMobile {
			from { 
				opacity: 0;
				transform: translateY(100%);
			}
			to { 
				opacity: 1;
				transform: translateY(0);
			}
		}

		.day-section.today {
			border-left: 3px solid #4CAF50;
			margin-left: -3px;
		}

		.month-section {
			margin-bottom: 20px;
		}

		.day-header {
			position: relative;
		}

		.lessons {
			padding: 8px;
		}
	}
	.month-stats {
    background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
    padding: 10px 15px;
    display: flex;
    gap: 25px;
    border-bottom: 1px solid #90caf9;
    font-size: 14px;
	}

.stats-item {
    color: #0d47a1;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 5px;
}

.stats-item:before {
    font-size: 12px;
    background: #2196f3;
    color: white;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
}

	@media (max-width: 350px) {
		.welcome-content h1 {
			font-size: 1.5rem;
		}

		.schedule-header h2 {
			font-size: 1.1rem;
		}

		.month-title {
			font-size: 14px;
			padding: 8px 10px;
		}
	}

	@media (max-width: 500px) and (orientation: portrait) {
		.welcome-screen {
			min-height: calc(100vh - 60px);
			padding-bottom: 60px;
		}
	}

	@media (max-width: 500px) and (orientation: landscape) {
		.drawer-content {
			max-height: 90vh;
		}
		
		.drawer-body {
			max-height: 70vh;
		}
		
		.welcome-content h1 {
			font-size: 1.5rem;
			margin-bottom: 1rem;
		}
	}
</style>