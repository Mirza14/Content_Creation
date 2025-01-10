# Investigation Scenario

- An Intruder has disconnected the main server from the Warevilla network in the Data centre. As an Analyst, investigate the attack on CCTV Servers to identify the adversary, who got unauthorized access to the server and deleted the CCTV streams.

## Learning Objectives

- How to extract custom fields in Splunk (Log parsing).
- Learn to create a parser for the custom logs.
- Use Splunk Search Processing Language (SPL) to investigate.

### Investigation time

- Figure out the datasets that are ingested into Splunk. Look at the *sourcetype* field for all the indexes using a wildcard search.
![Check the image for your reference](/Blogs/Images/image1.png)

- There are two datasets available:

1. *web_logs:* This file contains all the events related to web connections to and from the CCTV web server.
2. *cctv_logs* This file contains information about the CCTV application access logs.

**Examining CCTV Logs*

- Investigate by examining CCTV logs. To do so, try this simple search `index=* sourcetype=cctv_logs`
![Check the image for your reference](/Blogs/Images/image2.png)

*Understand the Problem:*

- After examining the logs from the above picture, these are the main issues:

1. Logs are not parsed properly by Splunk.
2. Splunk does not consider the actual timeline of the event, instead, it uses only the ingestion time.

*Fixing the Problem:*

- Logs look like unstructured data, therefore, we must extract relevant fields from them and adjust the timestamp, events, etc.,
- Logs were created from a custom log source that Splunk doesn't understand, so Splunk could not parse the fields properly.

#### Field Extraction Process

- Click on *Extract New Fields* from the left pane to extract new field.
![Check the image for your reference](/Blogs/Images/image3.png)

- On the next page, event logs will be presented that must be parsed propertly. Select the sample event and click on the *Next* button at the top of the page.
![Check the image for your reference](/Blogs/Images/image4.png)

- In selecting the method for extracting the fields: There are two methods using Regular Expression and Delimeters.
![Check the image for your reference](/Blogs/Images/image5.png)

- In selecting the fields, highlight the value in the event log and create the field extraction by giving them the relevant field name.
![Check the image for your reference](/Blogs/Images/image6.png)

- We will assign an appropriate name to each of the extracted fields based on the table below:

| Timestamp | Event | user_id | user_name | Session_id |
| --------- | ----- | ------- | --------- | ----------- |
| Event     | Logout |  5      | bytes    | kla95sklml7nd14dbosc8q6vop |

- It is visually clear that from the preview section, by selecting the fields, Splunk creates a regular expression to extract the field from all the events. All the extracted fields will be displayed in the Preview tab, as shown below:
![Check the image for your reference](/Blogs/Images/image7.png)

- You may note that some logs may have a different log format, and they may not be parsed using the parser we created above. Therefore, we may have to re-extract the fields from those events.

- We can click on each extracted field to check the extracted values. When we're satisfied with the extracted values, we can click on the *Next* button at the top of the page.

- In the validate step, will see a green tick mark next to the sample logs to indicate the correct extraction of the fields, or a red cross sign to signal an incorrect pattern, as shown below:
![Check the image for your reference](/Blogs/Images/image8.png)

- Actual RegEx behind this parser, `^(?P<Timestamp>\d+\-\d+\-\d+\s+\d+:\d+:\d+)\s+(?P<Event>\w+)\s+(?P<user_id>\d+)\s+(?P<user_name>\w+)\s+(?P<Session_id>.+)`

- The last step is to validate the extracted fields are right and save and perform the logs analyse.
![Check the image for your reference](/Blogs/Images/image9.png)

- This tab shows us the regular expression created, the fields extracted, and the sample event that contains the fields we wanted to extract. Let's save this session by clicking on the green Finish button at the top of the page and move on to the search tab to search the logs. To do so, we can click on the Explore the fields I just created in Search link on the next page.
![Check the image for your reference](/Blogs/Images/image10.png)
