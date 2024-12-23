<%*
// Insert a static markdown table originated from dataview table to the cursor. Support both Dataview Query and Dataview JS. 
//Refer: https://github.com/blacksmithgu/obsidian-dataview/issues/42
//Unknown: https://github.com/dsebastien/obsidian-dataview-serializer

const dv = app.plugins.plugins["dataview"].api;

/* DataView JS */

// Table Titles
const title1 = "当日文件";
const title2 = "编辑时间";

// Format Optimization
const escapePipe = s => new String(s).replace(/\|/, '\\|'); // required for links in Markdown table

// Get Time
const today = new Date();
today.setHours(0, 0, 0, 0);

// Return Table
// Note that dv.table() cannot be used as it creates HTML but we want Markdown.
const pages = dv.pages('!"MyTestFolder"')
	.filter(p => new Date(p.file.mtime) >= today)
	.sort(p => p.file.mtime, 'desc')
	.limit(32)
	.map(p => {
	let mfiles = escapePipe(dv.fileLink(p.file.path))
	let mtime = escapePipe(moment(p.file.mtime.toString()).format("YYYY-MM-DD | HH:mm"))
	return `|${mfiles}|${mtime}|`
	})
	.join("\n");

tR += `| ${title1} | ${title2} |\n| ------- | ----- |\n${pages}`;


/* DataView Query */
/*
const query = `
table WITHOUT ID 
file.link as "当日编辑", file.mtime as "编辑时间" 
from !"MyTestFolder" 
where file.mday = date(today) 
sort file.mtime asc 
limit 32
`

const dvqueryOutput = await dv.queryMarkdown(query);
const queryOutput = dvqueryOutput.value

tR += queryOutput.trimEnd();
*/
%>

