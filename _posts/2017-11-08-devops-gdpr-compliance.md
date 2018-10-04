---
ID: 5208
post_title: 'DevOps GDPR Compliance: The “Spark Notes” edition'
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/devops-gdpr-compliance/
published: true
post_date: 2017-11-08 14:09:45
---
<span style="font-weight: 400;">The upcoming enforcement of the European General Data Protection Regulation (GDPR) means that more likely than not your organization probably needs to make some changes to how it handles personal data, and how your team handles breaches and incident response.  We’re a company of developers, and know how it feels to have to sift through 100s of pages of legal jargon to figure out the changes you really need to make to achieve DevOps GDPR compliance for your organization.</span>

<span style="font-weight: 400;">In this article we’re going to focus on what’s important and cover an overview of the regulation, the key terminology you’ll see, what’s important to your team, and other resources you can use to learn more. In future posts we’ll be going through Sysdig Secure and how you can use it detect intrusions, block attacks, and run deep forensics in GDPR specific scenarios.</span>

**GDPR explained for DevOps Engineers**

<span style="font-weight: 400;">The European General Data Protection Regulation (GDPR) is a regulation that’s meant to unify and strengthen data protection for all members of the EU. Or via </span>[<span style="font-weight: 400;">http://www.eugdpr.org/</span>][1]<span style="font-weight: 400;"> “it’s the most important change in data privacy regulation in 20 years”</span>

**Timeline**

<li style="font-weight: 400;">
  <b>January 25th, 2012</b><span style="font-weight: 400;"> - initial proposal for updated data protection regulations</span>
</li>
<li style="font-weight: 400;">
  <b>December 15th, 2015</b><span style="font-weight: 400;"> - the parliament and council come to agreement on the Act</span>
</li>
<li style="font-weight: 400;">
  <b>April 27th, 2016 </b><span style="font-weight: 400;">regulation entered into force 20 days after it’s published in the EU Official Journal</span>
</li>
<li style="font-weight: 400;">
  <b>May 25th, 2018 </b><span style="font-weight: 400;">- Following a 2 year grace period the GDPR becomes full enforceable</span>
</li>

**Who does this effect?**

<span style="font-weight: 400;">GDPR requirements apply to any organization doing business in the EU or that processes personal data originating in the EU, be it the data of residents or visitors. (Yes, that includes companies headquartered outside the EU that collect data originating in the EU from EU citizens)</span>

**GDPR Key Terms**

**Data Processor - **<span style="font-weight: 400;">This is a “person, public authority agency or any other body which processes personal data on behalf of the controller.”</span>

*<span style="font-weight: 400;">Example:  A billing company processing customer payments on behalf of the Cellular Service  company is the Processor.</span>*

**Data Controller - **<span style="font-weight: 400;">This is a “person, public authority, agency or any other body which, alone or jointly with others, determines the purposes and means of the processing of personal data.“</span>

*<span style="font-weight: 400;">Example: A Cellular Service Provider collecting personal information from its customers is the Controller.</span>*

**Regulation vs Directive -** <span style="font-weight: 400;">A regulation is a legal act of the European Union that becomes immediately enforceable as law in all member states simultaneously. Regulations can be distinguished from directives which, at least in principle, need to be transposed into national law.</span>

**GDPR DevOps FAQ**

**What is personal data? - **<span style="font-weight: 400;">anything than be used to directly or indirectly identify a person</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Names</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Emails</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Bank details</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Medical information</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">IP addresses</span>
</li>

**Do I need a Data Protection Officer (DPO)?**

<span style="font-weight: 400;">Yes, if you:</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Are a public authority</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Engage in large scale systematic monitoring</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Do large amounts of data processing of sensitive personal data</span>
</li>

**What are the penalties for not complying with GDPR regulations?**

<li style="font-weight: 400;">
  <span style="font-weight: 400;">If it is determined that non-compliance was related to technical implications such as impact assessments or breach notifications, then the fine may be up to an amount that is the </span><b>greater</b><span style="font-weight: 400;"> of €10 million or 2% of global annual revenue from the prior year.</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">If your organization is found in non-compliance with key provisions of the GDPR, authorities can enforce a fine in an amount that is up to the </span><b>greater</b><span style="font-weight: 400;"> of €20 million or 4% of global annual revenue in the prior year. </span>
</li>

**What you need to know about handling breaches?**

<li style="font-weight: 400;">
  <span style="font-weight: 400;">A data controller must notify authorities within 72 hours of when they discover the breach</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The controller must know the number of people affected by the breach, and the contents of the data that was accessed</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The controller needs to understand and communicate the implications that this breach may have on its data subjects</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">The controller needs to describe measures implemented or planned to mitigate the spread of the breach as well measures to prevent this in the future. </span>
</li>

**Resources for GDPR Compliance Initiatives**

<li style="font-weight: 400;">
  <a href="https://gdpr-info.eu/"><span style="font-weight: 400;">https://gdpr-info.eu/</span></a><span style="font-weight: 400;"> - A comprehensive website that converts the official PDF of the GDPR into an easy to parse website</span>
</li>
<li style="font-weight: 400;">
  <a href="http://www.globalprivacyblog.com/files/2017/05/GDPR-Compliance-Checklist-003.pdf"><span style="font-weight: 400;">GDPR compliance checklist </span></a>
</li>
<li style="font-weight: 400;">
  <a href="https://www.itgovernance.co.uk/blog/list-of-free-gdpr-resources/"><span style="font-weight: 400;">List of free GDPR resources</span></a>
</li>

**How Sysdig Secure helps with your GDPR compliance initiatives **

<span style="font-weight: 400;">We’ll be covering GDPR specific use cases in following posts, but there are two main areas that </span>[<span style="font-weight: 400;">Sysdig Secure</span>][2]<span style="font-weight: 400;"> will address out of the box to make sure you’re confident on May 25th, 2018. </span>

<li style="font-weight: 400;">
  <b>Breach Prevention</b><span style="font-weight: 400;"> - Sysdig Secure has flexible policies that allow you to define what directories hold sensitive data, and detect users and programs reading from those directories. </span>
</li>
<li style="font-weight: 400;">
  <b>Breach Response & Forensics </b><span style="font-weight: 400;">- Sysdig Secure has full stack forensics capabilities that will track every single user command, and capture all activity pre and post any policy violation. What does this mean? If a breach occurs, we’ll know every connection opened or file accessed, and even be able to tell you the contents that were read, written, or passed of the wire. </span>
</li>

<span style="font-weight: 400;">Contact us <a href="https://go.sysdigrp2rs.wpengine.com/docker-security-demo">here</a> if you’d like to learn more about how we can help with your GDPR compliance initiatives. </span>

 [1]: http://www.eugdpr.org/
 [2]: https://sysdigrp2rs.wpengine.com/product/secure