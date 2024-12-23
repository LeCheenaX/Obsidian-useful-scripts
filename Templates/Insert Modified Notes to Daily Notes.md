<%*
// Based on Insert Static DV Table, dataview plugin and templater plugin. 

// Get today's date in ISO format
let today = moment().format("YYYY-MM-DD");

// Specify the note where you want to record the data
let DailyNote = moment(today).format("YYYY-MM-DD");
let recordNoteFolder = "日志/Daily Notes";
const recordNote = DailyNote; // Change this to your desired note name
const note = app.vault.getAbstractFileByPath(`${recordNoteFolder}/${recordNote}.md`);

// Define the start and end positions in the note to place the inserted data. 
// You can also use either "## Start Title" or "thing1\nthing2" if line break exist. 
let startPosition = "title: 当天编辑的文件\ncollapse: close"; 
let endPosition = "````"; // You can also use "## End Title"
// Get the content that you want to add to recordNote

const query = `table WITHOUT ID file.link as "当日编辑", file.mtime as "编辑时间" from !"MyTestFolder" where file.mday = date(today) sort file.mtime asc limit 32`;

// Set Program Delay for templater to parse created files
function delay(ms) {
return new Promise(resolve => setTimeout(resolve, ms));
}

new Notice("Autoupdate scripts are running! ", 3000);
console.log("Autoupdate scripts are running! ");

async function updateDailyNotes() {
	// Specify if note exists
	if (!tp.file.find_tfile(recordNote)) {
		// If the note doesn't exist, create it in the specified folder using Templater
		await tp.file.create_new("", recordNote, false, recordNoteFolder) 
		new Notice(`Created new note ${recordNote} in folder ${recordNoteFolder}.`, 5000);
		console.log(`Created new note ${recordNote} in folder ${recordNoteFolder}.`);
		await delay(500);
	}
	
	// Data Processing
	startPosition = startPosition.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
	endPosition = endPosition.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
	const regex = new RegExp(`${startPosition}[\\s\\S]*?(?=${endPosition})`);
	
	const dv = app.plugins.plugins["dataview"].api;
	const dvqueryOutput = await dv.queryMarkdown(query);  
	const queryOutput = dvqueryOutput.value
	let recordData;  
	const lines = queryOutput.split('\n');
	if (lines.length > 3) {
	    recordData = queryOutput.trimEnd();
	} 
	else {
	    recordData = "No note is modified today! ";
	}
	
	// Append the data to the specified note
	const content = await app.vault.read(note);
	let newContent;
	if (regex.test(content)) {
		newContent = content.replace(regex, `${startPosition}\n${recordData}\n`);
		await app.vault.modify(note, newContent);
		new Notice("Daily note auto updated! ", 2000);
		console.log("Daily note auto updated! ");
	} 
	else {
		new Notice("⚠️ ERROR updating note: " + recordNote +"! Check console log.", 5000);
		console.log(`⚠️ ERROR: The given pattern "${startPosition} ... ${endPosition}" is not found in ${recordNote}! `);
	}
};

// Debounce function to limit the rate at which a function can fire
function debounce(func, wait) {
    let timeout;
    return function(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

// Set up event listener to run the update function on every file save with debounce
app.vault.on('modify', debounce(async (file) => {
	console.log(`Detected File Change: ${file.name}`);
	if (file.name === `${recordNote}.md`)
		await delay(200);
	else
		await updateDailyNotes();
}, 60000)); // 60 seconds debounce


%>