# Obsidian-useful-scripts
Offer useful scripts and css snippets for obsidian. 

## Insert Modified Notes to Daily Notes
### Concepts
This template will detect modify behaviors and help you to record which notes that you modified today, automatically inserting these notes to any place of your daily notes. 
![image](https://github.com/user-attachments/assets/58e1ac58-acf6-498e-8832-dfe3fa0677fe)

The template is modular that: 
1. You don't have to modify your current daily notes templates at all.(But you can also customize a place for recording)
2. You can either insert the note to an existing place like the end of daily note, or modify your daily note template to add a place for inserting.
3. You can customize the query to collect the code. The query is the same as `Dataview Query`. 

### How to use this template?
1. You need to install the Templater plugin and Dataview plugin
2. In setting of Templater plugin, ensure that the folder template is enabled to auto-enforce daily note template to new created daily notes![image](https://github.com/user-attachments/assets/5edefd02-e065-46c6-b170-6f3c81eeb055)

3. You need to specify a folder to store your templates in the setting of Templater plugin
4. Create a new `.md` file in your template folder
5. Copy the template below and paste it into your `.md` file that just created
6. Modify the `Constants`:`RECORD_NOTE_FOLDER`,`QUERY_STRING`,`START_POSITION`,`END_POSITION`
7. In the setting of Templater plugin, add this template as "start-up template"![image](https://github.com/user-attachments/assets/4448ea44-9cc3-4ae8-b86e-f21381e67868)


#### The Record Note Folder
By default, the record folder is "Logs/Daily Notes". You should adjust it to your daily note folder. 

#### The Query String
You can seamlessly move your dataview query here but deleting the line-breaks. 
By default, will use the data in this dataview query:
```dataview
table WITHOUT ID
file.link as "Modified Notes", file.mtime as "Edit Time"
from !"MyTestFolder"
where file.mday = date(today)
sort file.mtime asc
limit 32
```

#### Start Position and End Position
This string defines where to insert the data. Not only the admonition plugin is supported, but for all other plugins. 
By default, this will insert a table of data to an admonition-callout:
```ad-note
title: Modified Notes on this day
collapse: close
```
Effect:
![image](https://github.com/user-attachments/assets/58e1ac58-acf6-498e-8832-dfe3fa0677fe)

It's also recommended to place it under a title(Require this title exist in your daily note template):
```
START_POSITION = "#### Dataview Query";
END_POSITION = "#### Dataview JS";
```
Effect:
![image](https://github.com/user-attachments/assets/57aa7556-265d-4d5d-b739-4ea9863b1dea)

If you want to insert to the end of your daily note, just leave `END_POSITION` blank. 
