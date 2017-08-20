---
layout: post
title: Generating Dynamic PDF Using iText
tags: [iText, pdf-generation]
---

http://www.lowagie.com/iText/\[iText\] is a library that allows you to generate PDF files on the fly.

In the following blog post I would show how to 1. Create PDF using PDF Template and PDF Form 2. Appending documents

## Create PDF using PDF Template for PDF AcroForm:

template.pdf is a PDF template with PDF Form fields firstName and lastName.

```java


// Read the template InputStream inputStream = new ClassPathResource("template.pdf", DemoPdfDynamicResource.class).getInputStream(); PdfReader reader = new PdfReader(inputStream);

// Writes the modified template to http response PdfStamper stamper = new PdfStamper(reader, output);

// Retrieve the PDF form fields defined in the template AcroFields form = stamper.getAcroFields();

// Set values for the fields form.setField("firstNameField", "Aparna"); form.setField("lastNameField", "Chaudhary");

// Setting this to true to make the document read-only stamper.setFormFlattening(true);

//Close the stamper instance stamper.close(); 
```

## Appending Documents

Here we will append all the pages from additionalInfo.pdf file to the template pdf.

```java

InputStream inputStream = new ClassPathResource("additionalInfo.pdf", DemoPdfDynamicResource.class).getInputStream(); if (inputStream != null) { PdfReader reader = new PdfReader(inputStream);

// get page count for additionalInfo.pdf int totalPageCount = reader.getNumberOfPages();

// get page count for template.pdf int templatePageCount = stamper.getReader().getNumberOfPages();

int i = 0; while (i &lt; totalPageCount) { i++;

// Insert a page in the template stamper.insertPage(templatePageCount + i, PageSize.A4);

// write the content of page from additionalInfo.pdf to the inserted page stamper.getUnderContent(templatePageCount + i).addTemplate(stamper.getImportedPage(reader, i), 1, 0, 0, 1, 0, 0); } } 
```

The advantage of using PDF templates over HTML templates is that you don’t have to invest lot of time in formatting. Also dealing with images is quite simple with PDF templates. There are different tools available to create PDF templates and forms. If you are using Mac Platform then PDFPen is an option.
