---
title: "About"
permalink: /about/
toc: true
---

# Something about me

Hello and welcome!

Many thanks for taking the time to visit my GitHub page, my name is Daniele Catanesi I'm an IT Industry veteran with more than 20 years’ experience in various fields of the IT world raging from IT Infrastructure, Network and Training as I used to work as Senior Infrastructure Trainer for a large training institution in Italy.

I sued to work as a *Messaging Specialist* but nowadays I spend my time designing *automation frameworks* and writing code to automate various business processes.

I guess somebody would use the word *DevOps* but I still prefer to consider me an Infrastructure Engineer.

**If you're interested in the full story feel free to read below.**

## The beginning

I started my career as a Windows Systems Administrator and later specialized in messaging systems where I oversaw messaging systems based on *Microsoft Exchange* for multiple enterprise customers.

In my role as Messaging Engineer I was in contact with a multitude of technologies spanning from Storage to Networking and  **nix* based systems that I was mainly using to maintain custom anti-spam rulesets and enforce messaging hygiene.

I enjoyed so much working with *nix that it eventually turned it from a passion to my full-time job.

## The dark side

I switched sides when I joined the oldest IT Company in the world, hint it has to do with the *blue* color, as Linux Systems Engineer dealing with Mail Systems, High Availability and Automation (mainly via Bash, Python and Puppet which at the time was an emerging technology).

During my years serving as Linux Engineer I learned a lot about writing reusable code and, as a colleague called it, the *tool mindset*

> Try not to write code that will serve a single purpose but write code that can be re-used multiple times. Your goal is writing a tool not a bunch of code that can be thrown away after you completed the task at hand.

## The end of cscript.exe

When Microsoft started developing *Monad* that picked up my interest as, like many other engineers, I always thought the lack of a proper scripting language was one of the weakest points for Microsoft Enterprise ecosystem and, as a bonus, it had many similarities with technologies I was already familiar with (mainly Bash at the time) so I started playing around with it.

I did not know but my skills would have come handy sooner than I thought.

## Back to the roots

Shortly after Microsoft released Exchange 2007, the first Exchange version that was heavily relying on PowerShell rather than on GUI tools, I had my chance to put my newly acquired skills at good use. As it often happens in life things abruptly changed, the project I was working on came to an unexpected end and my role was made *redundant* as the bulk of my work was bound to that specific project.

One day a colleague from another team told me they were facing a shortage in resources with skills in the Messaging and *Exchange* area and were desperate to find somebody to complete a project, while I was not planning to go back to the Microsoft world I thought this could give me some time to work on something rather than sitting on a bench without much to do. That's how all is started... **again!**

I told my colleague I had Exchange and Messaging skills as that used to be my old role and the same afternoon I spoke to his manager and the project Architect who agreed to immediately take me onboard.

Being completely honest I will tell after a lot of time spent into an SSH session going back to a world made of GUIs was familiar and *weird* at the same time but allowed me to further enhance my PowerShell skills and see how the Microsoft world has evolved during my *absence*.

Back ad the time I had to write a lot of *custom* code to carry on simple tasks like sending an email, no *Send-MailMessage* cmdlet existed at the time, but that just made motivated me to learn more and write tools/functions that I felt could be beneficial for other engineers like me. Nowadays most of the functions I wrote have been implemented as native cmdlet with better performances and functionality but still there are bits that are still used today like my *Microsoft Exchange Nagios Monitoring* plugin.

Like all good things even this project came to an end but I received an unexpected offer to remain in the team as an Advisory Engineer helping with Automation and Messaging projects, as the section title implies I was back to my roots.

## PowerShell and GUIs

In my new role as *Advisory Engineer* I found myself solving a lot of the same challenges I was dealing as a Linux Engineer, processes that were being executed manually, a lot of scheduled tasks that were configured with dependencies I defined as *black magic* and a lot, and I mean **a LOT**, of batch files.

I spent most of my first year converting all these batch files to proper PowerShell scripts while discovering the power of workflows, not a new concept in the Linux world, and started to gradually introduce **Orchestrator Runbooks** that helped removing lot of the black magic I was talking of earlier.

Once all the legacy tasks have been converted I started writing support scripts for my colleagues but there was an issue that seemed all too common, especially among Junior staff, which was adapting to the new paradigm of having a CLI tool carrying on all the heavy lifting work, this is when I started poking around with PowerShell and GUIs.

I started converting some of the CLI only tools I wrote to their GUI counterpart while most these tools were internal only I released some of them like my **[Exchange message tracking GUI](https://gallery.technet.microsoft.com/Exchange-message-tracking-73a2604c)** which is a replacement for (now defunct) Exchange Log Tracking utility.

## The present

Fast forward to date I’m lucky enough to work as a Senior Engineer, guess my role goes by the fancy name *DevOps* nowadays, developing complex automation frameworks for a large enterprise.

In few words I spend my time writing PowerShell code and not shying away from C# or Python which I deeply love for its simplicity and elegance, don’t tell around but sometimes I find it faster to implement something in Python rather than in PowerShell/C#.

I mainly deal with Active Directory and Identity Management so more often than not you will find example and snippets related, in a way or another, to Identity Management in general.

## Why a blog

I used to maintain a full blown WordPress blog where I rambled about various technical arguments but with the passing of years time grew scarce and I closed it down, issue is I do love to write (ok ramble is more appropriate) and to give back to the community so I started this GitHub space where I share code snippets, full scripts or functions hoping to help somebody out there.
