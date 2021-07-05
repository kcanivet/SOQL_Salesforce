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
