![Logo](https://github.com/radha2106/Viz1-Callcenter/blob/main/Logo.png)

# Project Sharelove

This is a proyect about KPI Metrics in a callcenter for a customer service by differents departments; emails, chats and phone. This include the precentage of good/bad rates given by how well done is the service provided. Data info was/is created in excel (since I cant find any data related with callcenter); then all the info is imported to SQL (excel files are keep as backups), all the calculations and aggroupations are done in SQL, after that just connect to PowerBi. Background is made in Figma.

## Documentation and Steps

### Excel
1. Bring a list of dates ranging from January to December (dont use first week and last 2 week of year and dont use weekends).
2. Bring a list of agents names.
3. Repeat agents names by amount of dates remaining, do the same for the dates.
4. Add an index to rows and get the months numbers.
5. Give 1-3 random days off to agents by months.
6. Use a column to filter if the agent is absent or not.
7. Add random number to determine the amount of call/chat/emails will the agent receive.
8. Using that random number repeat the agent name by the amount received.

Better details here: 
- **Excel Steps:** [Steps for excel](https://github.com/radha2106/Viz1/blob/main/Excel%20Steps.txt)
- **Initial Steps:** [Steps for call/chat/emails](https://github.com/radha2106/Viz1/blob/main/steps%20for%20call-chat-emails.txt)

### Steps for SQL
1. Create a calendar table
2. Create an absences table
3. Create an agent table
4. Create a mode table (where different type of response are)
5. Create a QA table (where QA score are stored)
6. Create different tables depending on how many department are (calls, chats, emails)
7. Create views
8. Create triggers and functions to help better checking inserting new rows/info

Better details here: 
- **SQL Snaps:** [SQL Table Snap](https://github.com/radha2106/Viz1/tree/main/tables_sql_folder)
- **Full SQL Tables Codes:** [SQL Full Code](https://github.com/radha2106/Viz1/blob/main/table_trigger_view.psql)

### Steps in PBI
1. Connect PBI query to Postgres query and import the tables
2. Make relationship between tables
3. Create necessary measures in an additional folder
4. Use a palette color of 2-3 colors (#013A63;#FDFCFC;#F1F1F1)

- **Measure are here**: [DAX Codes](https://github.com/radha2106/Viz1/blob/main/Dax%20Code).
- **Data Modeling is showed like this**: [Data Modeling](https://github.com/radha2106/Viz1/blob/main/DataModel%20PBI.png).
- **Full Dashboard is here**: [Viz1](https://app.powerbi.com/view?r=eyJrIjoiMWU2NDM5ZWItNThlYS00ZTllLThhZWUtZDNiNTE3MDJkNTY1IiwidCI6IjQ4MjkzMjgyLTgzMmQtNGQwYi05ZTBmLTVmMmFmYTg5YTFlNCIsImMiOjJ9)

## Colors Used

| Color             | Hex                                                                |
| ----------------- | ------------------------------------------------------------------ |
| Base Color | ![#013A63](https://via.placeholder.com/10/013A63?text=+) #013A63 |
| Background Color | ![#FDFCFC](https://via.placeholder.com/10/FDFCFC?text=+) #FDFCFC |
| Optional Color | ![#F1F1F1](https://via.placeholder.com/10/F1F1F1?text=+) #F1F1F1 |
