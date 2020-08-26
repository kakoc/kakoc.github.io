+++
title = "Personalized workspace"
description = "About my journey into personalization and final thoughts"
+++

## Foreword

Operating systems, text editors, keyboards, keyboard keymaps, workplaces - I've tried to work with different types of them and want to share my thoughts and conclusions about such "personalizations".  
Initially I wanted to describe the every aspect individually. But after a couple of sentences I had figured out that those things are mostly related to each other, intersect with each other. So this is why I'm planning to just describe the whole story, without any separations.

## Journey

**Windows**

As probably most of us I started with Windows. At that time it was really annoying to fullstack develop web applications with Windows(docker, console, tools). All looked like hacks with hacks. So I switched to OS X. For the sanity purpose as I'm hearing now from my colleagues - Windows is a good OS for developers too.  

**Dvorak Programmer**

Being a Windows user and while learning programming, maybe when I was procrastinating, but I begun to thought about boundaries. And the first boundary which came up to my mind was under my fingers - keyboard layout. Why is such keyboard layout used? And I fastly googled that it's really mostly due to historical reasons, but not due to ergonomic, performance reasons. After that I fastly googled that there are alternatives such as Dvorak, Dvorak Programmer, Colemak. My choise was Dvorak Programmer(I don't remember why particularly a such layout, maybe due to "Programmer" word presense, maybe due to "easy" installation). I instantly installed it and begin to explore. From the first point of view all things made sense. Instead of numbers different programming symbols({},[],(),&,!,$,#), in order to type (';,) also no need to press Shift.  
~How learnt~: participated in keyboard racing, while strolling tried in mind type different sentences. Basically it wasn't so hard transition compared to... Vim.  

**OS X**

This operating system was a very neat for the web development out of the box. Everything was ok...  
At that time while watching some tutorial I had found that a guy very quickly did different kinds of movements, text manipulations. I started to looking for what kind of editor he used. And in the description he pointed out that it was Vim. I fastly googled about Vim and found nice video tutorials about it. But unfortunatelly it was so hard to begin with. A lot of configs, plugins, tools, which were needed for comfortable work. I had a couple of attempts to start programming with terminal vim and all those attempts I failed. I really liked the idea behind text editing, navigation, but I didn't like to do a lot of configurations. I discovered that all popular editors(such as intellij, atom, vscode) can support that vim style editing which I particularly wanted. So at that time it was a perfect solution for me. I installed Vim plugin into my WebStorm IDE and begun to live with Dvorak Programmer, Vim and WebStorm. Over time in one day at work(it was my first working day) I've faced with the problem that my Web Storm license was for students but I was beginning to develop corporate software and this I needed to buy Web Storm or switch to the other IDE/editor. I decided to switch to VSCode. But with VSCode I faced with the problem that with Vim plugin text editing/movements are really weird - high latency, from time to time unexpected movements. Vim plugin could be ok when your project is small, but editing in big files is really weird. And I'm not talking about files with 10k lines, 150 lines are enough to get a weird editing feedback. And that was basically the time when I tried Emacs at my work. But don't think that it was a quick and easy transition. Range between switching from WebStorm to VSCode and between switching from VSCode to Emacs is equal around 1 year. When I switched to VSCode at that time I begun to play with Clojure. Emacs the most popular environment for lisp development and Clojure is not an exception. So I started to learn Emacs in my free time.  

**Emacs**

When I firstly installed Emacs it was even ugly than with Vim. Because even without those plugins and other configurations you can do a lot of with it out of the box. But with Emacs it's not possible, at least for me. Because out of the box you have so ugly layout for editing/movements... I thought that those keymaps were established via a random function. And also new googling and also a new solution. It was SpaceEmacs. The basic idea behind it is that you achieve Vimmed Emacs - you have Emacs with Vim style editing + Emacs' features. The downside of Spacemacs is that you receive totally reworked Emacs, so that you need to read a lot of materials about Emacs and additionally about Spacemacs. And I'm not surprised that I've got a lot of problems when I needed to change settings. All that stuff was some sort of a magic. Those layers akin to plugins in Vim in the sense of 0 configuration. But not only magic, desire to control things forced me to switch to classical Emacs and begin to customize it for myself. Spacemacs didn't a mistake, it showed me what things with Emacs I can do and perspectives were amazing! Customizing Emacs wasn't extremely painful, rather just a long way. Fortunatelly after playing wiht Spacemacs I had a picture in my mind of features which I needed. And fortunatelly for all those features there were screencasts. That way was basically incremental: I edited stuff, found some painfull peaces and after that tried to find a solution(packages, snippets, etc). So yes, it took around 1 year in order to be ready to use Emacs on day to day basis. And now years later I can say that there ara 2 thigs I hate in Emacs: EmacsLisp and the absense of real async.  

**To Linux**

At that time I worked with Emacs + OS X. And at that time I heavily used docker locally + in Emacs' there were a lot of LSP servers since multiple projects modified at the same time. Those things and I even not talking about opened tabs in browsers ate all available memore of my iMac(16 gb) and I always needed to wait + at that time the was an interesting bug with Angular because of which recompilation time was extremely longer than on Windows, Linux + it was really annoying to I couldn't work without mouse in order to be able to work with windows effectively. I tried windows managers on OS X and those are really fake compared to managers which Linux has. 

Also after some time I had found that it was extremely uncomfortable to me to place my hand on the mouse every time I needed it. Probably it was a defensive mechanism of my body due to sharp corners of the Apple mouse. Since I'm touching Apple mouse here sorry but I can't ommit the other aspect of Apple mouse: ~charging~: in order to charge Apple keyboard you just need to insert the lightning cable into the socket which is placed on the keyboard front end. So that the keyboard can be charged while you don't need to stop your work. Unfortunatelly It's not related to Apple mouse. In order to charge Apple mouse you need to flip the mouse on 180degrees so that the backend of the mouse is looking on the ceiling. Because the socket is placed on the mouse' backend! The charm to such situation adds the notification system which notifies you when around 2% of a battery is left.  
It was really a funny situation: when I struggled with such problems Youtube proposed me to watch a video about the genius of Jony Ive =).

Those facts triggered me to switch to Linux.  

**Linux**

Basically the transition to Linux wasn't so hard. I had some experience with different Linux distributives. Not all of them were supported at my job so I picked up Mini Ubuntu(Ubuntu without GUI) + i3 instead of Arch. The hardest thing was to configure keymaps since I used Dvorak Programmer. But I needed to do that only once. Now I have bash scripts and it's not a problem.  
And with setup I begun to feel happy. All needed actions can be performed from the keyboard in simple ways. The only thing which disturbed me was a necessity for some trivial actions in browser to use mouse. But at that time I was a Vim'er so that I installed Vimium plugin for my browser and all problems are gone.  

**Splitted keyboard**

After some time I started to suffer from pressing keys on my keyboard. It was a simple keyboard from Microsoft which I got with the workstation at my work. I had heard a lot about mechanical keyboard and that are awesome. But I even heard about splitted keyboards and that they give you a right ergonomic. I decided to give it a try and bought ErgoDox EZ. And it was the hardest thing to adopt with: I needed the rebind ~50% of keys + learn how use it in the reality. Again keyboard racing... Too much of them. Rebindings + racings = a lot of time and energy. Finally I started to use it on my day to day basis... And you know what? I didn't find any differences, except keys pressing were neat. Probably the reason why it's comportable for me to use original keyboard is that I'm using Dvorak Programmer and with its keymap most of the time your hands are placed in a right, comfortable position. Additionally I needed to work from other country. I switched my workstation to laptop as well as ErgoDox EZ to laptop's keyboard and it still suits me.

**Work standing**

While temporary working from abroad in the accomodation but not from the my main home/office I didn't have a comformtable table, chair. But there was a wide and high windowsill. And I tried to work standing with it. And you know it was an interesting experience. With work standing your mind is always focused, it's harder to begin to procrastinate, maybe because your muscles are always toned compared to the relaxed position when you are sitting in the comfortable chair. So that new innovation I brought from Ukrain to my home office and started to work standing on day to day basis. Little nitpicks: need to have a soft rug under you legs + need some time for muscles for adoption: first time it was impossible to work all working day with work standing.

## Conclusion

Based on my way I want to give a feedback about things which are worth themselves and about things which aren't. A little notice: it can be not so objectively since when you have some knowledge it can be hard to objectively say whether those knowledge are useful or not just because they have already changed you.

So:  

 - Windows - yes(nowadays)
 - Linux - yes
 - OS X - no. Every year worse and worse
 - Dvorak Programmer - no(time for adoption, always a lot of configurations, impossible to work on other people's computers)
 - Vim - yes
 - Emacs - yes
 - Splitted Keyboard - no
 - Work standing - yes
