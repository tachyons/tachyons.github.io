---
layout: single
title: Intellectual property, Open source and mimemagic
tags: ['open source', 'free software', 'intellectual property']
date: 2021-03-29 00:17 +0530
---


The alleged GPL violation in a ruby library called mimemagic which broke builds of rails projects across the world brought discussion of Intellectual property in the software world one more time last week. It is important to have a general idea about these terms for a software developer to avoid getting into big troubles in the future.

This is my attempt to write down my understanding about open source and Intellectual property, I am not a lawyer and the implementation of IP is based on the local laws which is not universal.

Though the modern concept of IP is originated from the exclusive access to rights and grants by the British monarch, it soon became a mechanism to promote the creation of intellectual goods, by giving the inventors some rights to the work they made.

Though IP may sound an alien term, it is something critical for any company, for instance, company name, logo, design of the logo etc are trademarks which is a kind of intellectual property. This enables companies to protect reputation and goodwill of the brand by preventing others from copying same brand symbols to sell their products.

Content produced by the company is protected by copyright, this also includes the software they made, in almost all legislations content produced is inherently owned by the author and protected by copyright, this prevents unauthorized access and redistribution of content by other people.

Another important intellectual property is patent which provides protection for innovative ideas, unlike copyright patent requires registration within a particular timeframe to the government agency. In general Patent protects ideas and copyright protects manifestation of that idea

There are other IPs like trade secrets which we aren't covering here to limit the scope of the article.

Historically hardware was the most expensive piece of a computer and program or the software that runs on them wasn't valued as important. But as more software distribution patterns emerged people started distributing just binaries of the software which prevented users from changing the software as per their needs. Richard R Stallman who was working in MIT lab during this period had a big trouble because of this, they used to adjust the printer firmware as per their need of paper size, But the new printer firmware came with just binary format which prevented from making the modification which caused wastage of papers they already had, Stallman requested the firmware owner to share the source code, but he refused.

That incident hit him big as usage of a computer owned by the user is dictated by the firmware owner. He believed this is a big ethical and social issue. He launched the GNU project in 1983 to create an operating system consisting of only free software. As per his own words the word free of "free" as in "free speech," not as in "free beer". Many including Linux followed this approach and many free software programs became available in the market.

In late 90s some people in industry realized the advantage of having free software in the enterprise world, i.e. when the code is shared more people can learn and adapt from it and there by saving time and cost. They also a coined a term called "Open Source" since people may easily interpret free software as free of cost software.

While free softwares and open source softwares and practically largely the same, they are two movements with different motivations but almost the same path to achieve the same. While free software movement is from the perspective of the user to have full control of his system, open source is from the perspective of software engineer or company to get benefit from shared source code.

Open source Initiative, which formalized the open source with 10 [rules](https://opensource.org/osd) and listed software licenses compatible with these rules as open source licenses.

But why does an open source software need license?

The Answer is simple, as we discussed in the initial section, any content produced is owned by the author and others can't redistribute without others permission. Licenses are a contract from a software developer to  the user to share some of his/her/their rights to the user, In the case of open source it will be freedom to redistribute, make derived work etc. This is different from EULA(End user license agreement) we see in the proprietary softwares which is to limit what the user can do with the software we purchased.

Also keep in mind that open source is not public domain which are the work which is not owned by anyone

There are many kinds of open source licenses ranging from permissive license which doesn't ask the users anything more than attribution to copyleft licenses which mandate you to make your software of also open source if you use one in your software.

I'll be writing another article about commonly used licenses and things to be taken care of while using such licenses in another post, till then you can refer [this awesome website](https://tldrlegal.com/).

Now let's go back to the incident that happened in the last week. freedesktop is a project on interoperability of free software projects, they maintain the list of [mime types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types) and is licensed under [GPL](https://tldrlegal.com/license/gnu-general-public-license-v3-(gpl-3)). GPL is a very restrictive license and if you want to link your software to a GPLed software and distribute, then you will have to license your software with the same restrictions. In other words, you can't directly link a  software linked with a gpled software/code and distribute under proprietary or permissive open source license like MIT. The mimemagic gem which is MIT licensed used this GPLed code from freedesktop which is violation of GPL license. Initial fix to convert mimemagic license to GPL. But due to the nature of copyleft licenses, software which includes mimemagic will also have to comply GPL rules. So they later removed the GPLed file and made the library to use mime data from the user's computer.

This may raise some questions about using GPL programs you use with your closed source(projects which are not open source) projects. First, you don't have to care much about GPL in backend of software as service model like most websites does, as you are just distributing the output of the software, not the software itself to the end user, this is also called as [saas loophole](https://resources.whitesourcesoftware.com/blog-whitesource/the-saas-loophole-in-gpl-open-source-licenses). Even if you aren't, it is unlikely that mysql being part of your source code, if you are just mysql as database, you are using its SQL interface and not as part of your programs.

But there are many other things a developer need to be to consider,

- You aren't supposed to copy and paste snippets from stackoverflow (read more [here](https://meta.stackexchange.com/questions/12527/do-i-have-to-worry-about-copyright-issues-for-code-posted-on-stack-overflow)).

- You aren't supposed to copy code snippets from GitHub even if the code is open source, some may require making your code also open source, some require attribution.

- If you found some code on GitHub or any publically accessible places, you shouldn't assume that code is open source unless license is explicitly given. For instance projects without explicit license in GitHub, all you can do is read the code and fork, you aren't supposed to change the code

And many more, the purpose of this article is just to give a general overview about license, and this is not a legal opinion
