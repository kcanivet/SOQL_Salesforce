# Subdividing a Campaign in Salesforce
## Using SOQL Queries and Google Sheets

When I began comparing events (classified as "Campaigns") in Salesforce, some annual events like the Scotiabank Calgary Marathon had been grouped into 1 event.  This posed a bit of a challenge because the event lasted for 8 years and raised significantly more money than any other event.  It was difficult to compare how successful an event was or events in a year or compare participants in one event vs another.  So a division was needed.  I couldn't find a standard report format that would include all the peices I needed, so I opened the developer consol and dove into the world of SOQL.  
> Wait, isn't it SQL?
> Not quite... SOQL (Salesforce Object Query Language) isn't quite the same thing as SQL (Structured Query Language) but it is close.

### Getting Started
There are a few tools you can use to help build your queries in Salesforce. 
1) Object Manager (Salesforce) - To get the API name for common item names in Salesforce.  For instance "Donation" is actually "Opportunity", but most items are closer.  Two word items often have spaces removed or underscores inserted to translate how it appears in Salesforce to how it is stored in the database.
2) Schema Builder (Salesforce) - To see the table connections and table elements.  Accessed through the Object Manager.
3) Developer Console (Salesforce) - To write the queries and preview the results
4) Spreadsheet Program (I used Google Sheets with Salesforce plug-in) - To perform the actual event/campaign switch and data manipulation

The purpose of the majority of these tools is to find the items to add to your queries, what is possible to add based on table connections, and getting the right syntax.  The structure of the queries does follow SQL fairly closely.

### The evolution of my query

Yeah, we got something!  Exploring the Campaign Table
```sql
SELECT Name, NumberOfContacts From Campaign
```
![](https://github.com/kcanivet/SOQL_Salesforce/blob/main/MitoCanada_Query_Campaign1.jpg)

Off to a good start, now let's focus on the Scotiabank Marathon and add in some dates
```sql
SELECT Name, NumberOfContacts, StartDate, EndDate From Campaign WHERE Name LIKE '%Scotiabank%'
```
Some of the information we need is in the Campaign table, but the donations are a key part to what we want to investigate.  Displaying the some of the record ID fields and our first record from a linked table `Campaign.Name` from the Campaign table.
```sql
SELECT AccountId, Amount, CampaignId, Campaign.Name From Opportunity
```
Adding in the Scotiabank filter using LIKE and sorting the list by event close date
```sql
SELECT AccountId, Amount, CloseDate, CampaignId, Campaign.Name From Opportunity WHERE Campaign.Name LIKE '%Scotiabank%' ORDER BY CloseDate
```
Focusing on one year at a time since we want to move donations and participants from one parent event to the individual child events by year
```sql
SELECT AccountId, Amount, CloseDate, CampaignId, Campaign.Name From Opportunity WHERE Campaign.Name LIKE '%Scotiabank%'AND CALENDAR_YEAR(CloseDate)=2012 ORDER BY CloseDate
```
After some digging, experimenting, reorganizing columns and a good dose of "Googling" we arrived at a query that contained all the info we needed to proceed to the next step. _I found out later that there was a bit a flaw in this query, but keep reading to the **Opps!** Section_
```sql
SELECT Id, Account.Name, AccountId, Amount, CloseDate, Campaign.Name, CampaignId From Opportunity WHERE Campaign.Name LIKE '%Scotiabank%'AND CALENDAR_YEAR(CloseDate)=2012 ORDER BY CloseDate 
```
![](https://github.com/kcanivet/SOQL_Salesforce/blob/main/MitoCanada_Query_Scotiabank_almost.jpg)

### What to do with the list
Once I got the query results I was looking for, the next step was to use the list to make the data transformations.  I searched for a way to export a list directly from the Developer console but didn't find one.  The ultimate destination for the donation / participant list is a spreadsheet program to make the event changes, so my search led me to Google sheets.  The salesforce plug-in needs to connect to a report or a table to extract and push data and it is on the plug-in Import that I found my answer.

![](https://github.com/kcanivet/SOQL_Salesforce/blob/main/MitoCanada_Query_Sheets_SOQL_ed.jpg)

From the SOQL query button in the plug-in, I copy and pasted my query from the Salesforce Developer console and within a few seconds the data was there!
Now that you have the data, moving a donation from the original campaign to its new campaign is acomplished with a simple Find & Replace.  Each event in Salesforce has an ID, so you can build a report with Campaign and ID or run a query to build the list and use them in the Find & Replace function of your spreadsheet.  Once the changes are complete you switch the Salesforce plug in to Update mode.   
1) Select the rows you wish to modify and in my case I want to update.  
2) The plug-in needs an ID Column and Results Column to proceed.  In my case there are 3 ID fields to choose from, but I want to change the Opportunity ID (aka Donation ID, or just the column labeled ID).  The Results Column is a place for the plug-in the write if the data transfer was successful.  Choose a column where you don't overwrite something important.
3) ![](https://github.com/kcanivet/SOQL_Salesforce/blob/main/MitoCanada_Marathon_Sheets_update5.jpg) Choose the column(s) to change in Salesforce.  I unselected everything but the CampaignID to minimize the chances of overwriting something.
4) Once your results column populates, Salesforce will be up to date!  You may need to refresh your report or dashboard to see the changes in Salesforce.  Take a minute to inspect the changes and document the results.
5) Now I just repeated the process for every year in the Parent event, modifying the query to include a different year each time.

### Opps!
The importance of step 4 above should not be overlooked.  I discovered a flaw in my query, which led to a series of mistakes (ultimately corrected) but the heat wave wasn't the only thing that led to a bit of sweating!
The first 5 years went smoothly, but in 2017 there was a Scotiabank Calgary Marathon and a Scotiabank Toronto Marathon.  As a reminder, my query contained `LIKE '%Scotiabank%'` so both campaigns showed up.  They did have different CampaignID's which should have saved me, but after repeating the same steps several times and getting into a rythm, I mistakenly thought I didn't select enough rows, so I quickly fixed that!... _Nooooooooooooooooooooooooooooooooo!_
Now how to fix it...
I consulted my before and after screenshots and indeed the number of participants in the Toronto Marathon did drop.  My initial reaction was that it should be easy.  I'll just find all the people from Toronto in the Calgary marathon and move them back... but the numbers didn't add up.  It turns out that some, but not many, people traveled from one city to the other to take part in the marathons.  Reviewing my workflow and thinking about what changed and what didn't, I was finding a bunch of dead ends.  It finally occured to me that there should be a time difference between changing the proper campaigns and the wrong ones.  After another round of desparate googling and testing I found the solution:
```sql
SELECT Id, Account.Name, AccountId, Amount, CloseDate, Campaign.Name, CampaignId, LastModifiedDate From Opportunity WHERE Campaign.Name LIKE '%Scotiabank Calgary%'AND CALENDAR_YEAR(CloseDate)=2017 ORDER BY CloseDate 
```
The `LastModifiedDate` had a time stamp detailed enough to find a subset that I changed before moving on to the next task.  Adding that timestamp to the query netted the correct number that were displaced from the Toronto Marathon and after repair, the donations balanced out as well.
```sql
SELECT Id, Account.Name, AccountId, Amount, CloseDate, Campaign.Name, CampaignId, LastModifiedDate From Opportunity WHERE Campaign.Name LIKE '%Scotiabank Calgary%'AND CALENDAR_YEAR(CloseDate)=2017 AND LastModifiedDate = 2021-06-30T00:27:48.000+0000 ORDER BY CloseDate
```

### Other Lessons Learned
**Back Up your data!** It's something that is always valueable, but this highlighted one one advantage and disadvantage in using Google Sheets in combination with the plug in... autosave as makes it easy to keep swaping data in and out of the same sheet but there is no going back.  To combat the autosave, you need to think new file first, not after.

