---
layout: post
title: Using Sieve to Filter Emails with Zip Attachments
author: jens_knipper
date: '2024-07-08 01:00:00'
description: I was recently the target of a phishing attack using fake invoices. They were easy to spot, but it was still annoying that they were landing in my inbox. So, I wanted to create a solution where these emails would not bother me anymore.
categories: Roundcube, Sieve, Filter
---
Luckily the emails were easy to spot and also quite similar, which made it easy to create a custom filter for them.  
They did not mention my name, the subject always contained the word "Invoice" and a zip file was attached.

Writing a filter for the first two criteria was, but I wanted it to be thorough and also  filter out emails with zip file attachments.

My email provider uses [Sieve](http://sieve.info/) to let the user define filters.
However, it was quite difficult to learn how to define more specific rules that are not as simple as 'check the subject if it contains the word invoice'.

After doing some research and through trial and error, I came up with the following rule:
```
require ["fileinto","mime"];

# rule:[Spam]
if header :mime :anychild :contenttype :is "content-type" "application/zip"
{
	fileinto "Spam";
}
```

The `:anychild` parameter is important here, because emails with attachments have the content type `application/multipart`. 
This means we have to look at the single parts of the email to determine if it contains a zip file.
This is what the `:anychild` parameter does.

To translate it into everyday language, what this filter does is: look at all the parts of an email. If any part is a zip file, move the email to the spam folder.  
This can now be used to add further filter criteria and make the filter more specific.