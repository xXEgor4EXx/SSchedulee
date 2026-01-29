export async function GetSchedule(){
	try{
		 const response = await fetch('https://api.ursei.su/public/schedule/rest/GetGsSchedIniData');
     if (!response.ok) {
        throw new Error(`Ошибка HTTP. Статус: ${response.status}`);
     }
     return await response.json();  
	}
	catch(err){
		throw err;
	}	
}

// Функция для получения расписания конкретной группы
export async function GetGroupSchedule(GS_ID) {
	try {
		const response = await fetch(`https://api.ursei.su/public/schedule/rest/GetGsSched?grpid=${GS_ID}`);
		if (!response.ok) {
			throw new Error(`Ошибка HTTP. Статус: ${response.status}`);
		}
		return await response.json();
	}
	catch(err) {
		throw err;
	}
}