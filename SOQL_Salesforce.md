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



