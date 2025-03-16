## Get yourself a note about the size of your OneNote book. 

![Checking your OneNote book size](/images/01.png)
The formula for the file size conversion:

*div(div(outputs('Get_file_metadata_using_path')?['body/Size'],1024),1024)*

In this simple flow, a notification via Teams will be sent in regular intervals. If the OneNote book queried grows beyond 2 GB in size you should consider splitting it up into multiple OneNote books, for performance and efficiency reasons. (Depends on your client hardware, however. Assuming you are using OneNote maybe via the fat client for your OS, be it Windows or MacOS.)

Want to learn more? Check out my many other articles:

[Advanced Automation using Azure & serverless services](https://medium.com/serverless-and-lowocode-pioneers/using-logic-apps-to-orchestrate-a-complex-video-processing-process-flow-a0ef20237511)

And even more:

**LinkedIn** > [LinkedIn Articles about Business & Tech](https://www.linkedin.com/today/author/mbrueckner)

**Medium** > [Articles about detailed topics around Cloud Computing and more](https://medium.com/@mohammedbrueckner)

**DevTo** > [Even more detailed articles on topics like using AI Services](https://dev.to/mrbrue)

Cheers!
[Mo](https://github.com/MoBRUEC)
