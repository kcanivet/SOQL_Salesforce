# Subdividing a Campaign in Salesforce
## Using SOQL Queries and Google Sheets

When I began comparing events (classified as "Campaigns") in Salesforce, some annual events like the Scotiabank Calgary Marathon had been grouped into 1 event.  This posed a bit of a challenge because the event lasted for 8 years and raised significantly more money than any other event.  It was difficult to compare how successful an event was or events in a year or compare participants in one event vs another.  So a division was needed.  I couldn't find a standard report format that would include all the peices I needed, so I opened the developer consol and dove into the world of SOQL.  
> Wait, isn't it SQL?
> Not quite... SOQL (Salesforce Object Query Language) isn't quite the same thing as SQL (Structured Query Language) but it is close.
