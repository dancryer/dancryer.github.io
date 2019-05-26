---
layout: post
title: "Preview Word, Excel, PowerPoint and PDF files in the browser"
permalink: /2018/04/preview-word-excel-pdf-browser
---

On a recent project, our system needed to be able to display a preview of various types of documents. We needed to be able to preview Word files, Excel files, PowerPoint presentations, PDF files, OpenOffice documents and so on.

Naturally, the first port of call was a Google search for *"how to display Word files in javascript." *We already knew it was possible to preview PDF files using [pdf.js](https://mozilla.github.io/pdf.js/), but feared that Microsoft formats might be a little trickier! I was envisioning having to use PHP Office to convert to PDF and then pdf.js to display them.

Luckily, we were saved that fate by the existence of the Office 365 and Google Docs preview functionality. With them, previews become as simple as:
### **Google Docs Preview:**
```
<iframe style="width: 100%; height: 450px;" src="http://docs.google.com/gview?url=**{ url encoded document url }**&amp;embedded=true" frameborder="0"></iframe>;
```


<img class="aligncenter wp-image-249 size-large" style="max-width: 90%; display: block; margin: 60px auto;" src="https://www.dancryer.com/wp-content/uploads/2018/04/google-preview-1024x465.png" alt="Example of a Google Docs PDF document preview." />
### **Office 365 Preview:**
```
<iframe style="width: 100%; height: 450px;" src="https://view.officeapps.live.com/op/embed.aspx?src=**{ url encoded document url }**&amp;embedded=true" frameborder="0"></iframe>
```


<img class="aligncenter wp-image-248 size-large" style="max-width: 90%; display: block; margin: 60px auto;" src="https://www.dancryer.com/wp-content/uploads/2018/04/microsoft-preview-1024x414.png" alt="Example of a Microsoft Excel document preview." />

Naturally, you won't want to use this if the documents you are previewing are particularly sensitive, as Google or Microsoft respectively will need to download a copy of your file in order to preview it. However, for a quick and efficient way of previewing documents, it saved us an awful lot of time!
