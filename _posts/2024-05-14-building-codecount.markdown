---
layout: post
comments: true
title:  "Building CodeCount"
excerpt: "A freelancer's friend for managing short-term tasks"
date:   2024-05-14 23:29:19 -0700
updated:  2024-05-15 12:09:59 -0700
categories: softwaredevelopment
---

<figure style="text-align:center">
  <img style="margin: 0 auto;" src="/assets/images/codecount-logo.webp" alt="count every project, every penny, every possibility" height=512>
  <figcation>Count Every Project, Every Penny, Every Possibility [<a href="https://chat.openai.com/share/266e3e3f-ae9b-4a5d-9813-728b7301b968" target="_blank">image is AI generated</a>]</figcation>
</figure>

CodeCount is a tracker tool for managing short-term projects mainly for freelancers. It provides freelancers with a way to manage projects' timelines, scope and resources/budget. A full visualisation of all aspects of the project gives an edge in development. A user can view the progression of the actual work done compared to the estimated work done over the duration of the project, the expenses made and the balance as well as the completion percentage. It was envisioned and developed by [**Allison Ogechukwu**](https://www.linkedin.com/in/allison-ogechukwu-2383bb1a4/){:target="_blank"} in partial fulfilment of the requirement of [**ALX SWE programm**](https://tech.alxafrica.com/software-engineering-programme-lagos){:target="_blank"}. Within 2 weeks of planned and dedicated development, the main focus of the web-based application which entails creating, editing and deleting projects to produce burndown chart visualisation as the project evolves was developed. To create the functionality in the quickest way possible, I used technologies and frameworks that take a lot of the heavy lifting away and give room to focus on the application logic.


## What's our (my) story? 🤔

Hey, I wanted something cool and useful to me and others. I developed CodeCount because I have always wanted to stay on top of projects as a self-taught software engineer. Staying organised always has been an innate quality I have had since I formed the ability to conceive thoughts. So when I started coding and developing software, I wanted to transfer that aspect of myself to my workspace and longed for ways software projects were handled. Then I stumbled upon an American TV drama series "Silicon Valley" in 2019 while I was an undergrad. Yes, you guessed it right, there was a character, "Jared", who was the project manager or Scrum master (if you use Agile development) of a small team of developers which later grew to be a multibillion-dollar company. Jared stayed on top of projects the team worked on ensuring project release dates were met and I took note of the tools he used which included the burndown chart. In one of the episodes, the first and only time it was shown was on a huge display screen for the entire dev team to watch. Beyond thinking that the burndown chart display was a metric used to gauge the development progress of the team, I just loved it (in my mind, it was so cool). So when the opportunity to develop a personal project came knocking on my door, I took it up and made a tool, I have always wanted to have in my toolkit.

## Architectural design 🏗️

<figure style="text-align:center">
  <img src="/assets/images/codecount-architecture.png" alt="devops with gnuradio and docker">
  <figcation>CodeCount Architecture</figcation>
</figure>

The architecture as shown is the high-level overview of the underlying structure of the CodeCount project. Mainly the application uses the [`React`](https://react.dev/){:target="_blank"} framework for a single page application interfaced with a backend-as-a-service (BaaS) platform similar to [`Firebase`](https://firebase.google.com/){:target="_blank"} called [`supabase`](https://supabase.com/){:target="_blank"} that offers API and real-time services. I chose to use [`React`](https://react.dev/){:target="_blank"} due to the huge and growing ecosystem of libraries for quickly prototyping features and functionalities. So libraries like [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"}, [`React-Hook-Forms`](https://www.react-hook-form.com/){:target="_blank"}, [`React-Hot-Toast`](https://react-hot-toast.com/){:target="_blank"}, and [`ReCharts`](https://recharts.org/en-US/){:target="_blank"} were used to develop the major features of the application. Notwithstanding, I also used [`React-Bootstrap`](https://react-bootstrap.netlify.app/){:target="_blank"} at the front end to quickly mock up a user-friendly and responsive UI and handle all date parsing and formatting operations with [`date-fns`](https://date-fns.org/){:target="_blank"} library.

Now let's move on to the interesting aspect of the project, the fully functioning features. The MVP (minimum viable solution) of CodeCount was approved to include a burndown chart visualisation, financial tracking capabilities and customisable alerts. Clearly, a good enough release should entice its users with priority features however project expectations and actual outcomes don't always correlate so the fully functional features of the application developed within 2 weeks are as follows;
<ol type="a">
<li><strong>Feature</strong>: Authentication with email and password</li>
<li><strong>Feature</strong>: Protected access to application for all users</li>
<li><strong>Feature</strong>: Project CRUD operations: users can create, update/edit and delete projects</li>
<li><strong>Feature</strong>: User Stories CRUD operations: users can create, update/edit and delete user stories associated with a project.</li>
<li><strong>Feature</strong>: Burndown Chart visualisation: The user can visualise a burndown chart for a given project.</li>
<li><strong>Not a Feature</strong>: Responsive and mobile compatible view.</li>
</ol>

## Challenges 💪

Every project has challenges, some more challenging than others. The same experience goes for me in the CodeCount project. Although React has a rich ecosystem of tools to use, they sometimes are not beginner-friendly so I had a little struggle with one of the [`React`](https://react.dev/){:target="_blank"} libraries I used, [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"}. React-Query is used to manage the API state in the [`React`](https://react.dev/){:target="_blank"} app and usually querying and mutating the API state is handled with [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"}. With a tutorial or two, I had the hang of querying and mutating the API through the library but when bugs surfaced, all hell went loose. It took me a couple of hours to figure out that I had misused [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"} and made different calls to the API for different components to the same endpoint rather than fetching the cache. In the background, [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"} updates itself based on some initial configurations such as stale time etc. After a couple of hours of learning about my errors with the help of online resources and ChatGPT, I figured out the cause of the error (application not updating after mutation operation as expected) and learnt the right way to interact with the [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"} client in my custom hook scripts. This didn't go as easy as I have stated it so let me add a little more context. The bug originated with the latest version of the [`React-Query`](https://tanstack.com/query/latest/docs/framework/react/overview){:target="_blank"} library `@tanstack/react-queryv5` and I had no luck processing finding the cause of the bug so I switched to version 4 based on code snippets and tutorials online which then showed code errors making the debugging process easier and straightforward.

## Take-aways 🚶‍♀️

I feel the CodeCount project reaffirmed my thoughts on my problem-solving skills/capability. I loved how productive I was towards fulfilling the project's deliverable particularly, in the timeframe it was done. More impressive is how I managed to accomplish this while fully working on my machine learning research (actively training models). However, so I do not digress, I have learnt how to use and integrate [`supabase`](https://supabase.com/) with react apps or any language by using their documentation. Also, I honed my skills with react internal state management with hooks and external/remote state management (API) with react-query library and look forward to more complicated and exciting designs with them. Still on the technical takeaways, I improved my GIT commit messages and version control around deliverables and features. Aside from what I am grateful for (which went well), some processes in retrospect could have been improved. One such process is time management. I could have started working on the features as I had planned from the beginning of the project sprint which would have given me enough time to strengthen the quality of the deliverables and covered a lot of edge cases. Though I do tend to revise certain technologies due to a long space of time since the last use, I think staying in check with technology is a good practice to avoid learning/revising time.

Thankfully, the CodeCount project has informed me greatly of my potential as a software engineer and what specialisation I would like to pursue (being backend). This also informs me of my future endeavours as a software engineer. Seeing the inclination to handle more data-based logic also aligns with my other aspirations such as research scientist and machine learning engineer roles.

>_P.S. Knowing how to use Vim/Emacs or any other terminal-based editor is handy when dealing with remote access (since most remote servers are Linux-based) but I like to avoid the stress of figuring out how to make multiple changes at the same time or commenting code shortcuts and just use VSCode so although ALX advocates for Vim/Emacs, I am a strong VSCode fan (Sorry 😖)_.

In conclusion, taking part in the CodeCount project exposed me to modern technologies, gave me an avenue to try out my problem-solving skills and display my creativity. I feel a step closer to my software engineering goal.

**Project link** ➡️ [https://github.com/AllisonOge/codecount](https://github.com/AllisonOge/codecount){:target="_blank"}

**Landing page** ➡️ [https://allisonoge.github.io/codecount/](https://allisonoge.github.io/codecount/){:target="_blank"}

**More about me** ➡️ [https://www.linkedin.com/in/allison-ogechukwu-2383bb1a4/](https://www.linkedin.com/in/allison-ogechukwu-2383bb1a4/){:target="_blank"}

Feel free to leave any comments 🙏
