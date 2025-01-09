<%*
// Configuration Constants
const RECORD_NOTE_FOLDER = "Logs/Daily Notes";
const QUERY_STRING = `table WITHOUT ID file.link as "Modified Notes", file.mtime as "Edit Time" from !"MyTestFolder" where file.mday = date(today) sort file.mtime asc limit 32`;
const START_POSITION = "title: Modified Notes on this day\ncollapse: close"; 
const END_POSITION = "````";
const DAILY_NOTE_FORMAT = "YYYY-MM-DD";


// Get today's date in ISO format
let today = moment().format("YYYY-MM-DD");
let DailyNote = moment(today).format(DAILY_NOTE_FORMAT);
let recordNote = DailyNote; 
const dv = app.plugins.plugins["dataview"].api;

// Delay function
function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

new Notice("Autoupdate scripts are running! ", 3000);
console.log("[Modified File Recorder] Autoupdate scripts are running! ");

// Function to create a new note
async function createNewNote() {
    await tp.file.create_new("", recordNote, false, RECORD_NOTE_FOLDER);
    new Notice(`Created new note ${recordNote} in folder ${RECORD_NOTE_FOLDER}.`, 5000);
    console.log(`[Modified File Recorder] Created new note ${recordNote} in folder ${RECORD_NOTE_FOLDER}.`);
    await delay(2000);
}

// Function to fetch query output
async function fetchQueryOutput() {
    try {
        return await dv.queryMarkdown(QUERY_STRING);
    } catch (error) {
        new Notice("⚠️ ERROR querying data: " + error.message, 5000);
        console.log(`[Modified File Recorder] ⚠️ ERROR: ${error}`);
        throw error; // Rethrow to handle in the calling function
    }
}

// Function to process query output
function processQueryOutput(queryOutput) {
    const lines = queryOutput.split('\n');
    return lines.length > 3 ? queryOutput.trimEnd() : "No note is modified today! ";
}

// Function to read note content
async function readDailyNoteContent(note) {
    return await app.vault.read(note);
}

// Function to update the note
async function updateNoteContent(content, recordData, note) {
    const regex = new RegExp(`${START_POSITION}[\\s\\S]*?(?=${END_POSITION})`);
    if (regex.test(content)) {
        const newContent = content.replace(regex, `${START_POSITION}\n${recordData}\n`);
        await app.vault.modify(note, newContent);
        new Notice("Daily note auto updated! ", 2000);
        console.log("[Modified File Recorder] Daily note auto updated! ");
    } else {
        new Notice("⚠️ ERROR updating note: " + recordNote + "! Check console log.", 5000);
        console.log(`[Modified File Recorder] ⚠️ ERROR: The given pattern "${START_POSITION} ... ${END_POSITION}" is not found in ${recordNote}! `);
    }
}

// Main function to update daily notes
async function updateDailyNotes() {
    try {
        if (!tp.file.find_tfile(recordNote)) {
            await createNewNote();
        }
        
		let note = app.vault.getAbstractFileByPath(`${RECORD_NOTE_FOLDER}/${recordNote}.md`);
        const dvqueryOutput = await fetchQueryOutput();
        const recordData = processQueryOutput(dvqueryOutput.value);
        const content = await readDailyNoteContent(note);
        await updateNoteContent(content, recordData, note);
    } catch (error) {
        new Notice("⚠️ An unexpected error occurred: " + error.message, 5000);
        console.log(`[Modified File Recorder] ⚠️ An unexpected error occurred: ${error}`);
    }
}

// Debounce function to limit the rate at which a function can fire
function debounce(func, wait) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), wait);
    };
}

// Set up event listener to run the update function on every file save with debounce
app.vault.on('modify', debounce(async (file) => {
    console.log(`[Modified File Recorder] Detected File Change: ${file.name}`);
    if (file.name === `${recordNote}.md`) {
        await delay(200);
    } else {
        console.log(`[Modified File Recorder] Try updating ${recordNote}.md`);
        await updateDailyNotes();
    }
}, 60000)); // 60 seconds debounce

%>
